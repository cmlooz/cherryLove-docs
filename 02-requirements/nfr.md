---
id: REQ-DOC-002
status: approved
approved_by: Camilo
approved_at: 2026-11-11
supersedes: nfr.md draft v0 (REQ-N-010…REQ-MED-002)
---

# Non-functional requirements

ID scheme: `REQ-N-<AREA>-<NN>`. Sources: `you` (proposal), `DEC-*` (decision log), `—` (senior recommendation, accepted at freeze, DEC-035). IDs are frozen and immutable.

## ARC — Architecture

| ID | Statement | Source |
|----|-----------|--------|
| REQ-N-ARC-01 | **Modular monolith, pragmatic hexagonal:** domain and use cases in plain Python; Django ORM is an adapter behind repository ports; DRF views are inbound adapters. | you, DEC-002 |
| REQ-N-ARC-02 | API built with **Django REST Framework**; OpenAPI generated and typed (drf-spectacular + mypy). | DEC-029 |
| REQ-N-ARC-03 | **Contract-first:** the OpenAPI spec is the only coupling between API and UIs; TS types in `cherrylove-core` are generated from it. | you |
| REQ-N-ARC-04 | Any UI works against any API that honors the contract; no layer imports another layer's internals. | you |
| REQ-N-ARC-05 | Bounded modules: no cross-module foreign keys except to `Couple` and `User`; internal domain events between modules → split-ready for services. | you |
| REQ-N-ARC-06 | Fewest dependencies: every new library needs an ADR. | you |
| REQ-N-ARC-07 | Separate UIs (React DOM / React Native) + shared `cherrylove-core` (types, API client, validation, sync, design tokens, i18n), SemVer, published to GitHub Packages. | DEC-003, DEC-004 |
| REQ-N-ARC-08 | PostgreSQL server-side; the domain talks to repositories, not to Postgres. | you |
| REQ-N-ARC-09 | SOLID; strict typing (mypy strict, TS strict); validation before `try/catch`. | you |
| REQ-N-ARC-10 | Architecture documented with ADRs and C4 diagrams. | — |

## SEC — Security

| ID | Statement | Source |
|----|-----------|--------|
| REQ-N-SEC-01 | Couple-scoped authorization on every read/write (permission + queryset filter; defense in depth). Never trust a client `couple_id`. | you |
| REQ-N-SEC-02 | A resource from another couple returns **404**, not 403. | — |
| REQ-N-SEC-03 | Short-lived JWT access + rotating refresh with blacklist; web refresh in `HttpOnly` + `Secure` + `SameSite` cookie; mobile in OS secure storage. | — |
| REQ-N-SEC-04 | Argon2 password hashing; rate limiting on login, recovery, invites. | — |
| REQ-N-SEC-05 | Card and note bodies encrypted at rest. | — |
| REQ-N-SEC-06 | Private media served through short-lived signed URLs; real MIME, size, and quota validation. | DEC-009, DEC-020 |
| REQ-N-SEC-07 | HTTPS, HSTS, CSP, strict CORS, security headers. | — |
| REQ-N-SEC-08 | Staff never see content (no break-glass in v1). | DEC-015 |
| REQ-N-SEC-09 | OWASP ASVS level 2 checklist in the test plan. | — |

## PERF — Performance and scale

| ID | Statement | Source |
|----|-----------|--------|
| REQ-N-PERF-01 | p95 < 200 ms on read endpoints without media. | you |
| REQ-N-PERF-02 | Cursor pagination on every list; N+1 queries forbidden and checked in tests. | you |
| REQ-N-PERF-03 | Composite indexes `(couple_id, <date>)`; queries verified with `EXPLAIN ANALYZE`. Data volume for the check: 1 M rows per hot table (REQ-N-PERF-06). | you, DEC-034 |
| REQ-N-PERF-04 | Redis cache for today's card and home dashboard. | — |
| REQ-N-PERF-05 | Thumbnails, optimized formats, lazy loading in the gallery. | — |
| REQ-N-PERF-06 | Load-data generator seeds 1 M rows per hot table for local performance checks. | DEC-034 |

## QA — Quality and traceability

| ID | Statement | Source |
|----|-----------|--------|
| REQ-N-QA-01 | `request_id` end to end (client → API → logs). | you |
| REQ-N-QA-02 | Errors as RFC 9457 Problem Details with translatable messages. | — |
| REQ-N-QA-03 | Coverage: services ≥ 90 %, overall ≥ 80 %. | DEC-023 |
| REQ-N-QA-04 | CI on every PR: lint, types, tests, dependency audit, build. | — |
| REQ-N-QA-05 | Official documentation first; exact pinned versions verified when each plan starts. | you |
| REQ-N-QA-06 | Conventional Commits; 1 commit per step; 1 PR per phase; ≥ 1 test per code step + integration tests per phase. Every commit/PR/test cites REQ IDs. | you, DEC-023 |

## UX

| ID | Statement | Source |
|----|-----------|--------|
| REQ-N-UX-01 | "Cherry" design system (tokens) shared through `cherrylove-core`. | you |
| REQ-N-UX-02 | Responsive, mobile-first; WCAG 2.2 AA. | you |
| REQ-N-UX-03 | UI in **Spanish and English from v1**; no hard-coded strings. Docs and code identifiers in English. | DEC-021, DEC-030 |
| REQ-N-UX-04 | All UX/UI is approved before implementation. | you |

## LEG — Legal and platform

| ID | Statement | Source |
|----|-----------|--------|
| REQ-N-LEG-01 | Spotify/YouTube only through visible embedded players (their terms forbid background-only playback). | DEC-010 |
| REQ-N-LEG-02 | Privacy policy and terms covering storage, 90-day purge, and staff-no-content rule. | — |

## OPS — Environments

| ID | Statement | Source |
|----|-----------|--------|
| REQ-N-OPS-01 | Local: Docker Compose (PostgreSQL, Redis, MinIO, mail catcher). | DEC-022 |
| REQ-N-OPS-02 | Production target decided in a later ADR. | DEC-022 |

## Next step

`roles-actuators.md`.
