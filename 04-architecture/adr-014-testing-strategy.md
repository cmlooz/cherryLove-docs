---
id: ADR-014
status: approved
approved_by: Camilo
approved_at: 2026-11-12
---

# ADR-014 — Testing strategy

Implements: DEC-023, REQ-N-QA-03, REQ-N-QA-06, REQ-N-PERF-02, REQ-N-PERF-06, REQ-N-SEC-09, REQ-N-UX-02,
REQ-N-ARC-06/09.
Builds on: ADR-001 (fakes for ports), ADR-002 (`tests/` layout), ADR-003 (drift check), ADR-005 §7, ADR-006,
ADR-008 §2, ADR-010 §6, ADR-012 §2.

## Context

DEC-023 fixes the rules: ≥ 1 test per code step, integration tests per phase, coverage services ≥ 90 % and overall
≥ 80 %, and every test cites the REQ IDs it proves. Earlier ADRs already demanded specific tests (cross-couple
404s, contract drift, field classification, translation parity). This ADR decides the levels, tools and
conventions so those demands become one coherent suite in four repos.

"Services" in REQ-N-QA-03 needs a definition, because ADR-002 forbids a `services` folder: here it means the
**`domain` and `application` layers** of every API module, and `sync/` + `api-client/` in `cherrylove-core`.

## Decision drivers

1. The riskiest code gets the strongest tests: authorization, sync, encryption, quota.
2. Fast feedback: most tests run without a database or a device.
3. Real infrastructure where behavior depends on it (constraints, row locks, partitions).
4. Every test is traceable to a REQ ID automatically, not by hand.
5. Few dependencies; deterministic tests.

## Decision

### 1. Levels (API)

| Level | Scope | Infrastructure | Share of tests |
|-------|-------|----------------|----------------|
| Unit | Domain + application, with in-memory fakes of ports (`tests/fakes/`) | none | most |
| Integration | Repositories, query services, views, jobs, migrations | **real PostgreSQL**, MinIO, Redis (Docker) | per module and phase |
| Contract | Generated OpenAPI = approved contract (ADR-003) | none | 1 suite |
| Architecture | import-linter (ADR-001), `unscoped` outside backoffice (ADR-005), audit field classification (ADR-008) | none | 1 suite |
| Property-based | Sync invariants: replay idempotency, cursor never skips, deletes win (ADR-006) | PostgreSQL | focused |
| Load | Today, home, lists, pull at 1 M rows (REQ-N-PERF-06), cache on/off (ADR-011) | full stack | per phase that touches them |

SQLite is **never** used as a stand-in for PostgreSQL: partitions, `FOR UPDATE`, constraints and `inet`/`jsonb`
behave differently.

### 2. Levels (TypeScript)

| Repo | Unit / component | End-to-end smoke |
|------|------------------|------------------|
| `cherrylove-core` | Vitest; sync engine against an in-memory `LocalStore` and a fake `Network` port; property tests for the queue and cursor | — |
| `cherrylove-web` | Vitest + Testing Library; presentational components by props only | Playwright: one happy path per phase + accessibility scan with axe (REQ-N-UX-02) |
| `cherrylove-mobile` | Jest with `jest-expo` (Expo's supported preset) + Testing Library for React Native | Maestro flows: one happy path per phase |

### 3. Tools and why

| Tool | Where | Why | Rejected alternative |
|------|-------|-----|----------------------|
| `pytest` + `pytest-django` + `pytest-cov` | API | Fixtures, `django_assert_num_queries` for N+1 (REQ-N-PERF-02), coverage gates | `unittest` only: more boilerplate, no fixtures |
| Hypothesis / fast-check | API sync / core sync | Finds the interleavings humans miss in replay and cursor logic | Hand-written cases only |
| Own test-data builders | API, core | Readable, typed; no dependency | `factory_boy`: another dependency and ORM-coupled |
| Vitest | core, web | Native TS/ESM, fast | Jest (kept only where Expo requires it) |
| Playwright + `@axe-core/playwright` | web E2E | Cross-browser, built-in tracing, accessibility checks | Cypress: one browser engine family |
| Maestro | mobile E2E | YAML flows, external CLI (no project dependency) | Detox: heavy native setup |
| k6 | load | External binary with JS scripts; no project dependency | Locust: Python dependency in the API repo |

All versions pinned in ADR-016.

### 4. Determinism

- Time through a `Clock` port (no `freezegun`); civil-date tests include a timezone **with** DST, even though the
  default `America/Bogota` has none (REQ-F-CARD-04 makes the zone configurable).
- IDs through an injectable UUIDv7 generator; randomness (random past card, REQ-F-CARD-02) through a seeded port.
- No test depends on another test's data or order.

### 5. Required suites (from earlier ADRs)

| Suite | Source |
|-------|--------|
| One test per row of the permission matrix | ADR-005 §7 |
| Cross-couple 404 for **every** endpoint, generated from the URL conf | ADR-005 §7 |
| Refresh-token reuse revokes all sessions | ADR-004 §3 |
| Sync: replay, LWW `overwrote_newer`, deletes win, invisible → delete, `410 cursor_expired` | ADR-006 |
| Quota under concurrent reservations | ADR-007 §4 |
| Audit append-only (grants + trigger) and field classification | ADR-008 §2, §4 |
| Encryption: AAD swap fails, purge removes the DEK | ADR-009 |
| Every error code translated; i18n parity | ADR-010 §6, ADR-012 §2 |
| Job idempotency keys and scheduler slot keys | ADR-013 §2–3 |

### 6. Traceability (REQ-N-QA-06)

- Python: `@pytest.mark.req("REQ-F-CARD-07")` (registered marker; unknown markers fail).
- TypeScript: test titles start with the ID, e.g. `describe("REQ-F-CARD-07 published card is read-only", …)`.
- CI exports a REQ → tests matrix from markers and titles; session I uses it as the test plan's backbone, and a
  frozen REQ with no test is reported.

### 7. Gates

| Gate | Threshold | Source |
|------|-----------|--------|
| Coverage of `domain` + `application` (API) and `sync/` + `api-client/` (core) | ≥ 90 % lines | REQ-N-QA-03 |
| Overall coverage per repo | ≥ 80 % lines | REQ-N-QA-03 |
| Each code step | ≥ 1 new or changed test citing a REQ ID | DEC-023 |
| Each phase | ≥ 1 integration test + E2E smoke where a UI exists | DEC-023 |
| Security | OWASP ASVS L2 checklist reviewed per phase (owned by session I) | REQ-N-SEC-09 |

## Consequences

- ✅ The riskiest rules (tenancy, sync, crypto, quota) have dedicated, named suites.
- ✅ Most tests run in milliseconds thanks to ports and fakes (ADR-001).
- ✅ Traceability is generated, not maintained by hand.
- ❌ Integration tests need Docker locally and in CI.
- ❌ Two TS test runners (Vitest, Jest for Expo) — accepted because Expo's supported preset is Jest-based.
- ⚠️ Coverage is a floor, not a goal; a test without an assertion on behavior does not count in review.

## Next step

Camilo approves, rejects, or patches by section number. Then ADR-015 (CI).
