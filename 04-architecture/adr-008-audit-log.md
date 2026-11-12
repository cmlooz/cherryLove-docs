---
id: ADR-008
status: approved
approved_by: Camilo
approved_at: 2026-11-11
---

# ADR-008 — Audit log and partitioning

Implements: REQ-F-LOG-01…04, REQ-F-AUTH-07, REQ-F-ADM-06, REQ-F-ADM-07, REQ-N-QA-01, REQ-N-SEC-08, DEC-015.
Builds on: ADR-001 §4 (audit is transactional), ADR-002 (`audit` module, request_id middleware), ADR-005 (Actor).

## Context

Every action is logged — including **views** — with actor, couple, action, entity, time, IP/device,
`request_id` and a field diff **without sensitive content**; the log is append-only and partitioned by month
with configurable retention. It also feeds the staff dashboard (REQ-F-ADM-06) and log viewer (REQ-F-ADM-07).

Two things make this harder than it looks: views make it the highest-volume table, and "without sensitive
content" must hold even when a developer adds a new field next year.

## Decision drivers

1. No action escapes the log; a failed or rejected action is logged too.
2. No content (card/note bodies, descriptions, song notes) ever reaches the log.
3. Append-only enforced by the database, not by good intentions.
4. Retention and queries stay cheap at volume (REQ-N-PERF-06 seeds 1 M rows).
5. No new dependency.

## Options considered

| Topic | Options | Choice |
|-------|---------|--------|
| Where | Same PostgreSQL ⭐ · separate DB · external log service | Same DB: transactional with the change (ADR-001 §4); no new infra |
| Partitioning | Native declarative range partitions + own job ⭐ · `pg_partman` extension · `django-postgres-extra` | Native + ~1 job: no extension (not every host offers it), no dependency |
| Diff filter | Denylist sensitive fields · **allowlist** loggable fields ⭐ | Allowlist fails closed: a new field is hidden until someone decides it is safe |

## Decision

### 1. Schema (`audit_event`, module `audit`)

| Column | Type | Notes |
|--------|------|-------|
| `id` | uuid (v7) | |
| `occurred_at` | timestamptz | partition key; PK = `(id, occurred_at)` (PostgreSQL requires the key in the PK) |
| `request_id` | text | REQ-N-QA-01 |
| `actor_type` | enum | `partner · staff · system · anonymous` |
| `actor_id`, `couple_id` | uuid, nullable | null for anonymous / staff-global actions |
| `action` | text | `<entity>.<verb>`: `memory.created`, `note.viewed`, `auth.login_failed`, `sync.pushed`, `admin.couple_archived` |
| `outcome` | enum | `success · rejected · failed` |
| `entity_type`, `entity_id` | text, uuid | |
| `ip` | inet | |
| `device` | jsonb | client type (`web`/`mobile`), app version, normalized user agent |
| `changes` | jsonb | see §2 |

### 2. Diff without content

Each module declares, per entity, which fields are **loggable with values**. Every other changed field is
recorded as name only:

```json
{ "publish_at": { "from": "2026-12-01T00:00:00Z", "to": "2026-12-24T00:00:00Z" },
  "body": { "changed": true } }
```

A unit test fails if an entity has a field that is in neither list (loggable / content). Failed logins store an
HMAC of the email, never the email or password, so attempts can be correlated without exposing who typed what.

### 3. When and how it is written

| Action kind | Written | Why |
|-------------|---------|-----|
| Successful mutation | Inside the use-case transaction via the `AuditLog` port | ADR-001 §4: change and audit commit together |
| Rejected / failed mutation | Own autocommit insert after rollback | The rollback must not erase the attempt (driver 1) |
| View, login, logout, auth failure | Own autocommit insert in the request | No business transaction exists |
| Sync push | One event per op (via its use case) + one `sync.pushed` summary | REQ-F-LOG-01 |

Synchronous inserts in v1 (~1 ms each). If the load test (REQ-N-PERF-06) shows views hurting REQ-N-PERF-01, the
documented seam is buffering view events in Redis and batch-inserting from the worker.

### 4. Append-only in the database

- The application DB role has `INSERT, SELECT` only on `audit_event` (`UPDATE`/`DELETE` revoked).
- A trigger raises on `UPDATE`/`DELETE` as a second barrier.
- Partition maintenance uses a separate `maintenance` role, only from the worker's maintenance job and migrations.

### 5. Partitions and retention

- Parent table `PARTITION BY RANGE (occurred_at)`, one partition per calendar month (UTC), created by a raw-SQL
  migration (Django ORM does not create partitions).
- A daily job keeps **3 future months** created and **detaches + drops** partitions older than the retention
  setting (backoffice, default **12 months**). Dropping a partition is instant; deleting rows would not be.
- Indexes per partition: `(couple_id, occurred_at)`, `(actor_id, occurred_at)`, `(action, occurred_at)`.
- Retention also covers purged couples: audit rows hold no content, so they stay until their partition expires.
  The privacy policy must say so (REQ-N-LEG-02).

### 6. Staff dashboard and viewer

| Need | Source |
|------|--------|
| Dashboard (REQ-F-ADM-06): active couples, logins, items created and visited per week/month | Rollup job writes `backoffice_metric_rollup`; the dashboard never scans raw audit |
| Log viewer (REQ-F-ADM-07): filter by type, date, couple, user | Query service on `audit_event`; **date range required (max 31 days)** so PostgreSQL prunes to 1–2 partitions |

Staff see audit metadata only — by construction there is no content in it (REQ-N-SEC-08).

### 7. Application logs are a different thing

Operational logs (errors, timings) go to stdout as JSON via stdlib `logging` with a small formatter that adds
`request_id`. They are not the audit trail, are not append-only, and never contain content either.

## Consequences

- ✅ Change and audit commit together; rejected attempts still leave a trace.
- ✅ New fields are hidden from the log until classified — privacy fails closed.
- ✅ Retention is a `DROP` per month; queries prune by date.
- ✅ Zero new dependencies.
- ❌ Partition management is our code (one job, one migration pattern) — tested in plan F.
- ❌ Every view costs an insert; measured, not assumed (seam in §3).
- ⚠️ Two DB roles (app, maintenance) must exist in every environment, including Docker Compose.

## Next step

Camilo approves, rejects, or patches by section number. Then ADR-009 (encryption at rest).
