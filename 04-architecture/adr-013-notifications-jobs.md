---
id: ADR-013
status: approved
approved_by: Camilo
approved_at: 2026-11-12
---

# ADR-013 — Notifications and background jobs

Implements: REQ-F-NOTIF-01…05, REQ-F-ACC-05, REQ-F-AUTH-01/02/08, REQ-F-ADM-03, REQ-F-CARD-06, REQ-N-ARC-06,
DEC-017, DEC-019, DEC-022, DEC-026.
Builds on: C4-001 (worker/scheduler process; "Redis as broker → ADR"), ADR-001 §4–5 (events, unit of work),
ADR-006 §1 (publish job bumps `seq`), ADR-007 §3 (validation jobs), ADR-008 §5 (partition job), ADR-012 §4 (push
language).

## Context

Many earlier ADRs deferred work to "a job": emails, pushes, card publication, reminders, media validation,
purges, rollups, partition maintenance. This ADR picks the job runner, the scheduler, and the push channels.

The decisive question is the one ADR-001 §4 left open: **what if the process dies between committing a change and
enqueueing its job?** With an external broker, the job is lost (the Django docs themselves recommend enqueueing in
`transaction.on_commit()`, which still leaves that gap).

## Decision drivers

1. A committed change never loses its job (no lost invite emails, no unpublished cards).
2. Jobs are idempotent: a retry or a duplicate never double-sends.
3. Fewest moving parts and dependencies (REQ-N-ARC-06).
4. Framework-standard API, so the backend can be swapped without touching use cases.

## Options considered (job runner)

| Option | Verdict |
|--------|---------|
| Celery + Redis broker + beat | ✅ Most mature; retries and periodic built in. ❌ Many dependencies; enqueue is not transactional (driver 1). |
| RQ / Dramatiq + Redis | ✅ Simpler than Celery. ❌ Same non-transactional enqueue; periodic needs another library. |
| `procrastinate` (PostgreSQL) | ✅ Transactional, periodic and retries built in. ❌ Its own API instead of Django's standard one. |
| **Django Tasks API (Django 6.0, `django.tasks`) + `django-tasks-db` backend** ⭐ | ✅ Standard Django API; the database backend writes the job row **inside the current transaction**, so job and change commit together (driver 1); no broker. ❌ Young backend; retries and periodic scheduling are ours (small). |

Verified in the official docs: Django 6.0 ships only development backends (`ImmediateBackend`, `DummyBackend`)
and points to third-party backends for production; `django_tasks_db.DatabaseBackend.enqueue()` creates the row
synchronously in the active transaction and runs via `manage.py db_worker`. Compatibility with the exact pinned
Django version is confirmed in ADR-016.

## Decision

### 1. Runner

- Use cases enqueue through a `JobQueue` port (`shared/application`); the adapter calls Django Tasks.
- **Reliable side effects are enqueued inside the use-case transaction**, not after commit. In-process domain
  events (ADR-001 §3) remain for cheap reactions; anything that must happen is a job.
- Backend: `django-tasks-db` on PostgreSQL. **Redis is not a broker** (closes the open point in C4-001).
- Two queues, two worker processes: `default` (email, push, publish, purges) and `media` (Pillow/ffmpeg,
  ADR-007), so a long video never delays a push.
- Local and tests: `ImmediateBackend` for unit/integration tests, the database backend in Docker Compose.

### 2. Idempotency and retries

- Every job has an **idempotency key** stored in a `job_execution` table with a unique constraint, e.g.
  `push:card_published:{card_id}:{user_id}`, `email:invite:{invite_id}:{attempt}`. A second run with the same key
  is a no-op.
- Retries: a wrapper re-enqueues failed attempts with exponential backoff (1 min → 4 h, max 6 attempts), then
  marks the job `failed` and writes an audit event (ADR-008). Deferred execution support of the backend is
  verified in plan F before relying on it.

### 3. Scheduler (periodic work)

A small `scheduler` management command runs as its own process and ticks every 60 s. Each periodic job is
enqueued with a **slot key** (`rollup:daily:2026-11-12`) stored under a unique constraint, so two schedulers, a
restart or a late tick never enqueue twice. No dependency.

| Job | Schedule | Requirement |
|-----|----------|-------------|
| Publish due cards (`publish_at <= now`) → mark published, record sync change, enqueue push | every minute | REQ-F-CARD-06, ADR-006 |
| Expire invites older than 72 h | hourly | DEC-017 |
| Anniversary / partner birthday reminders | hourly tick; sends at **09:00 couple time** | REQ-F-NOTIF-02 (yearly, DEC-036) |
| Purge archived couples after 90 days (rows, media prefix, DEK) | daily | DEC-019, ADR-009 §5 |
| Purge tombstones older than 90 days | daily | ADR-006 §5 |
| Audit partitions: create ahead, drop expired | daily | ADR-008 §5 |
| Metrics rollups | daily / weekly / monthly | REQ-F-ADM-06 |
| Release stale quota reservations | hourly | ADR-007 §4 |
| Delete `sync_client_op` rows older than 30 days | daily | ADR-006 §1 |
| Check Expo push receipts | every 15 min | §4 |

### 4. Push channels

| Channel | Registration | Delivery | Dependency |
|---------|--------------|----------|------------|
| Mobile | App registers its **Expo push token** after login (Expo Notifications, REQ-F-NOTIF-03) | Expo Push HTTP API; receipts checked later; `DeviceNotRegistered` deletes the token | HTTP client `requests` (declared explicitly) |
| Web | Service worker + **Web Push** subscription, permission asked only after a user gesture (REQ-F-NOTIF-03) | Encrypted Web Push with VAPID keys; `404/410` deletes the subscription | `pywebpush` |

Devices live in the `notifications` module: `(user_id, channel, token/subscription, last_seen_at)`.

### 5. Notification rules

| Type | Recipient | Trigger | Payload |
|------|-----------|---------|---------|
| `card_published` | The partner of the card's author | Publish job | Generic text in recipient's language (ADR-012) + `{type, card_id}` for the deep link. **Never the body** (REQ-F-NOTIF-05) |
| `anniversary` | Both partners | Scheduler | Generic text + days/years count |
| `partner_birthday` | The other partner | Scheduler | Generic text + partner's name |

- Per-user, per-type toggles (REQ-F-NOTIF-04, REQ-F-ACC-05); **default on** for all three types.
- Preferences are checked when the job **runs**, not when it is enqueued (a toggle switched off in between wins).
- Emails (invite, recovery) are jobs too, through Django's email backend behind an `EmailSender` port; local
  delivery goes to the mail catcher (REQ-N-OPS-01).

### 6. Open question raised and resolved

**OQ-003** — REQ-F-NOTIF-02 gave no cadence for the anniversary reminder. Resolved by **DEC-036**: yearly in v1;
a monthly reminder ("mesiversario") would be a new REQ ID.

## Consequences

- ✅ Change and job commit together: no lost emails, pushes or publications.
- ✅ No broker to run; Redis stays a cache (ADR-011).
- ✅ Use cases depend on a port; swapping to an RQ backend later touches only the adapter and settings.
- ✅ Dependencies: `django-tasks-db`, `pywebpush`, `requests`. Versions pinned in ADR-016.
- ❌ We own retry and scheduling code (small, tested in plan F).
- ❌ A database-backed queue polls PostgreSQL; fine at this scale, measured in the load test (REQ-N-PERF-06).
- ⚠️ Exactly one scheduler is expected, but the slot keys make two harmless.

## Next step

Approved. Then ADR-014 (testing strategy).
