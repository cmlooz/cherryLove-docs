---
id: ADR-001
status: approved
approved_by: Camilo
approved_at: 2026-11-11
amended_by: ADR-007 (adds module `media` to §1)
---

# ADR-001 — Modular monolith, hexagonal layers, dependency rules

Implements: DEC-002, REQ-N-ARC-01, REQ-N-ARC-05, REQ-N-ARC-06, REQ-N-ARC-08, REQ-N-ARC-09.
Builds on: C4-001 (one server codebase, API + worker processes).

## Context

DEC-002 already fixed the *style*: modular monolith, pragmatic hexagonal, plain-Python domain, ORM behind
repositories. What is still open is **how that style becomes rules a reviewer (and CI) can check**:

- Which modules exist and where their borders are.
- Which layer may import which.
- How modules talk without cross-module foreign keys (REQ-N-ARC-05).
- Where "pragmatic" stops: what is allowed to skip the domain.

Without written rules, a Django project drifts back to fat models and views that touch every table.

## Decision drivers

1. Domain testable without Django or a database (REQ-N-QA-03: services ≥ 90 % coverage).
2. Split-ready: a module could leave the monolith without rewriting its callers (REQ-N-ARC-05).
3. Read performance: cursor lists and dashboards inside p95 < 200 ms (REQ-N-PERF-01/02).
4. Few dependencies (REQ-N-ARC-06).
5. Rules enforced by a machine, not by memory.

## Options considered

### O1 — Classic Django (fat models, logic in views/serializers)
- ✅ Fastest start, most tutorials.
- ❌ Contradicts DEC-002: domain coupled to ORM, untestable without DB. Rejected.

### O2 — Strict hexagonal everywhere (every read and write goes through domain entities + mappers)
- ✅ Purest separation.
- ❌ Lists and dashboards pay mapping cost and invite N+1; much boilerplate for screens that only read.

### O3 — Pragmatic hexagonal with a split read side (CQRS-lite) ⭐ recommended
- **Writes** (commands) go through use case → domain → repository port. All business rules live here.
- **Reads** (lists, detail views, dashboards) go through *query services* defined as ports in the
  application layer, implemented with the ORM, returning plain DTOs. No rule may live in a query.
- ✅ Rules stay pure and tested; reads stay fast and optimizable (`select_related`, indexes, cursors).
- ❌ Two paths to learn; discipline needed so queries never "decide" anything.

## Decision

**O3.** Concretely:

### 1. Modules (bounded, named by the domain — screaming architecture)

| Module | Owns (aggregates) | Requirements |
|--------|-------------------|--------------|
| `identity` | User, Couple, Invite, credentials, sessions | AUTH, ACC, ADM-01/02/09 |
| `cards` | Card | CARD |
| `memories` | Memory, Song, Place | MEM, SONG, PLACE |
| `diary` | Note | NOTE |
| `favorites` | Favorite | FAV, HOME-02 |
| `notifications` | Device, preference, notification jobs | NOTIF, ACC-05 |
| `sync` | Client operation log, change cursor | SYNC |
| `audit` | Audit event | LOG, ADM-07 |
| `backoffice` | Staff roles, quotas, metrics, admin views | ADM-03…08 |
| `media` *(added by ADR-007)* | MediaAsset, couple storage counter | MEM-08/09/10, SONG-05, ACC-01, ADM-05 |
| `shared` (kernel) | IDs (UUIDv7), `CoupleId`, `UserId`, clock, base errors, event bus port | — |

**Why `memories` holds Memory, Song and Place together:** REQ-F-SONG-07 (one relation edited from both
sides) and REQ-F-PLACE-05 (deleting a place unlinks its memories) need *one transaction*. A module border
is a consistency border, not a screen border. Real foreign keys are allowed inside the module.

### 2. Layers inside every module

| Layer | Contains | May import |
|-------|----------|------------|
| `domain` | Entities, value objects, domain services, domain events. Plain Python (`dataclass`, `Protocol`). | `shared.domain` only. **Never Django.** |
| `application` | Use cases (commands), ports (repository, query service, gateways), DTOs. | own `domain`, `shared` |
| `infrastructure` | ORM models, repository and query implementations, S3/email/push adapters. | own `application` + `domain`, `shared`, Django |
| `interfaces` | DRF views + serializers, admin, job handlers, event handlers. | own `application`, `shared`, Django/DRF |

Dependency rule: **arrows point inward** (`interfaces` → `application` → `domain`; `infrastructure`
implements `application` ports). `interfaces` never touches ORM models or repositories directly.

### 3. How modules talk

| Need | Mechanism | Example |
|------|-----------|---------|
| Ask another module something | Its public facade `<module>/application/public.py` (use cases/queries + DTOs only) | `favorites` asks `memories` for titles of starred items |
| React to something that happened | Domain event on the in-process event bus, dispatched **after commit** | `NoteVisibilityChanged(only_me)` → `favorites` hides it from partner (REQ-F-FAV-04) |
| Reference another module's entity | ID value only (`MemoryId`), **no foreign key** — except `Couple` and `User` (REQ-N-ARC-05) | `Favorite(item_type, item_id)` |

Forbidden: importing another module's `domain`, `infrastructure` or `interfaces`.

### 4. Things that must NOT be eventual

- **Audit** (REQ-F-LOG-01/03) is written through an `AuditLog` port **inside the same transaction** as
  the change. Losing an audit event because a handler failed after commit is not acceptable.
- Events dispatched after commit are allowed to be lost on a crash in v1 (notifications, favorites
  cleanup can be re-derived). A transactional outbox is the documented seam if a module is ever split.

### 5. Transactions

A use case owns its transaction through a `UnitOfWork` port (infrastructure: `transaction.atomic`).
Repositories never open transactions themselves.

### 6. Enforcement

| Tool | Role | Why |
|------|------|-----|
| `import-linter` (dev-only) | CI contracts: `layers` per module, `independence` between modules, `forbidden` Django in `domain` | Declarative, fails the PR; one dev dependency instead of a custom AST script. Version pinned in ADR-016. |
| mypy strict | Ports are `Protocol`s; adapters must satisfy them | REQ-N-ARC-09 |
| Unit tests on `domain` + `application` with in-memory fakes | No DB needed | Driver 1 |

Alternatives for enforcement: custom AST test (no dependency, but we maintain it) or code review only
(not machine-checked). Rejected for driver 5.

## Consequences

- ✅ Business rules testable in milliseconds; adapters tested separately with the DB.
- ✅ Each module has a visible public surface; splitting one later means turning `public.py` + events into HTTP + broker.
- ✅ Reads optimizable without bending the domain.
- ❌ More files per feature than classic Django (entity + ORM model + mapper + port + adapter).
- ❌ Two query paths to keep honest: a query service that contains a rule is a review blocker.
- ⚠️ Cross-module lists (home timeline, favorites) are composed by the `favorites` module via facades;
  watch for N+1 across facades — facades must accept batches of IDs.

## Open points for later ADRs

- Exact folder tree and file names → ADR-002.
- Event bus implementation and job runner → ADR-013.
- How `sync` sees changes from every module (cursor source) → ADR-006.

## Next step

Camilo approves, rejects, or patches by section number.
