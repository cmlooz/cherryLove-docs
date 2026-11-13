---
id: IMPL-CORE-001
status: approved
approved_by: Camilo
approved_at: 2026-11-13
---

# IMPL-CORE-001 — Implementation plan: `cherrylove-core`

Implements: REQ-N-ARC-03, REQ-N-ARC-04, REQ-N-ARC-06, REQ-N-ARC-07, REQ-N-ARC-09, REQ-N-QA-01…06, REQ-N-UX-01/03,
REQ-F-SYNC-01…10, DEC-003, DEC-004, DEC-021, DEC-023, DEC-039.
Builds on: C4-001, ADR-002 §2, ADR-003, ADR-004, ADR-006, ADR-007, ADR-010, ADR-012, ADR-014, ADR-015, ADR-016,
API-001.
Session: H2 (GOV-004-H2). Runs in parallel with plan F.

## Quick path

- 9 phases, **1 PR per phase**, **1 commit per step**, **≥ 1 test per step** whose title starts with the REQ ID it
  proves (DEC-023, ADR-014 §6).
- Order: tooling → contract → api-client → validation → i18n engine → sync (local + push) → sync (pull + cycle) →
  media uploads → "Cherry" content (tokens + copy catalogs, **waits for UX-001**).
- Core has **zero runtime dependencies** and no DOM / React / React Native / Node imports. The compiler and the
  linter enforce it, not discipline (§3).
- 6 gaps found and **answered by Camilo on 2026-11-13** (§6, all option A). Three need plan F changes (§7).

## 1. Version re-verification (REQ-N-QA-05)

Verified on 2026-11-12 with `npm view` (npm registry) and the endoflife.date API.

| Package | ADR-016 pin | Newest in pinned line | Result |
|---------|-------------|-----------------------|--------|
| Node.js | 24.x LTS (24.22.3) | 24.22.3 (2026-11-11), EOL 2028-04-30 | ✅ keep |
| `typescript` | 5.9.3 | 5.9.3 (latest overall is 7.0.2) | ✅ keep; `typescript-eslint` 8.72.2 still requires `<6.1.0`, `openapi-typescript` 7.13.0 requires `^5.x` |
| `eslint` | 9.39.5 | 9.39.5 | ✅ keep |
| `typescript-eslint` | 8.72.2 | 8.72.2 | ✅ keep (peer `eslint ^9`) |
| `vitest` | 5.1.0 | 5.1.0 | ✅ keep (engines `^24.0.0`) |
| `openapi-typescript` | 7.13.0 | 7.13.0 | ✅ keep; CLI offers `--enum-values` (used in phase 2) |
| `fast-check` | 4.10.2 | 4.10.2 | ✅ keep |
| `@vitest/coverage-v8` | **not pinned** | 5.1.0 | ⚠️ needed for the coverage gate → pinned 5.1.0 (OQ-H2-3) |
| `vite` | 8.3.8 (web only) | — | ⚠️ `vitest` 5.1.0 declares `vite` as a **non-optional peer** → also a core dev dependency, pinned 8.3.8 (OQ-H2-3) |

No patch bumps needed today. Phase 1 step 1 repeats this check and records it in the commit body.

## 2. Phase order and why

| # | Phase | Depends on | Why here |
|---|-------|------------|----------|
| 1 | Tooling, CI, publishing | — | The fence before the house: every later step must land on a green `ci` and be installable. Publishing an almost-empty `0.1.0` proves the registry path before anything depends on it. |
| 2 | Contract types | 1 | Every other folder is typed from the contract (REQ-N-ARC-03). |
| 3 | API client | 2 | Highest-risk non-sync code (auth refresh, Problem Details); first thing web/mobile need (login). |
| 4 | Validation | 2 | Needed by forms and by the sync engine (validate before enqueue, REQ-N-ARC-09). |
| 5 | i18n engine | 2 | Small and independent; unblocks web/mobile phase 1 (login screen + errors). Catalog *content* waits for UX (phase 9). |
| 6 | Sync — local writes and push | 3, 4 | ADR-006 split into two PRs: one ~10-step PR would be too large to review well. |
| 7 | Sync — pull and cycle | 6 | Cursor, 410, scheduling, property suite. |
| 8 | Media upload queue | 7 | ADR-006 §6: outbox ops wait for uploads, so the outbox must exist first. |
| 9 | "Cherry" content | 5, **UX-001 approved** | Tokens **and** UI copy (incl. error messages) are UX decisions (REQ-N-UX-04; session B owns empty/error states). |

Corrections to the session prompt's suggested order: **validation** (ADR-002 §2 lists `validation/`, the prompt did
not) is added as phase 4; sync is split in two; **media uploads** become their own phase (REQ-F-SYNC-07 has its own
queue); **copy catalogs** move to phase 9 with the tokens, because error messages are UI copy that session B approves.

Plan F dependency: core never needs the running API. Phases 3, 7 and 8 are integration-tested against a
contract-shaped fake server; real end-to-end runs live in web CI (ADR-015 §4).

## 3. Platform-agnostic rule (ADR-002 §2)

Three fences, each checked in CI:

| Fence | How | Catches |
|-------|-----|---------|
| Compiler | `tsconfig`: `"lib": ["ES2022"]` (no `DOM`), `"types": []` (no `@types/node`) | `window`, `document`, `localStorage`, `setTimeout`, `fetch`, `crypto`: they do not type-check |
| Linter | `no-restricted-imports`: `react`, `react-dom`, `react-native`, `expo*`, `@react-native*`, `node:*` | Imports of platform packages, even type-only |
| Manifest | Test: `package.json` has no `dependencies` and no `peerDependencies` | A runtime dependency sneaking in (REQ-N-ARC-06) |

Consequence: everything platform-specific enters through a **port** — `HttpTransport` (wraps the app's `fetch`),
`Clock` (time and timers), `RandomSource` (bytes for UUIDv7), `LocalStore`, `SecureTokenStore`, `FileSource`,
`StorageUploader`. This is also what makes ADR-014 §4 determinism free: tests pass fakes.

The linter also enforces the inner dependency direction (one `no-restricted-imports` block per folder):

```text
contract ← validation ← sync
contract ← api-client ← sync
contract ← i18n
tokens   (imports nothing)
```

`target`/`lib` is ES2022 because Hermes (mobile) does not implement every newer built-in; plan H confirms it.

## 4. Versioning and consumption

- Package `@cmlooz/cherrylove-core` on GitHub Packages; a tag `vX.Y.Z` publishes it (ADR-015 §4).
- The package exports `CONTRACT_VERSION` (the contract it was generated from).
- SemVer rule (OQ-H2-2): `0.y.z` while building (each phase = minor) until phase 9 and `contract-v1.0.0`; from
  `1.0.0`: core major = contract major; contract additive or core feature → minor; fix → patch. A change that would
  break only core's TS API inside contract v1 is shipped as a deprecation, not a major.
- GitHub Packages needs a token **even for public packages** (verified in GitHub docs, "Working with the npm
  registry"):

| Where | How web/mobile install core |
|-------|-----------------------------|
| Repo `.npmrc` (committed) | `@cmlooz:registry=https://npm.pkg.github.com` and `//npm.pkg.github.com/:_authToken=${NODE_AUTH_TOKEN}` — no secret in git (DEC-039) |
| Local machine | Classic PAT with `read:packages` in the env var, never in a file inside a repo |
| CI | `GITHUB_TOKEN` with `packages: read`, after granting `cherrylove-web` and `cherrylove-mobile` read access in the package's "Manage Actions access" settings |

Co-development before a release: `npm pack` a tarball and install it; never `file:` links committed to app repos.

## 5. Phases

Step format: `commit message` — what changes — **test** (title prefix = REQ ID).

### Phase 1 — Tooling, CI and publishing

**Goal:** an installable, empty-but-real package with every CI gate of ADR-015 §1 running.
**REQ:** REQ-N-QA-03, REQ-N-QA-04, REQ-N-QA-05, REQ-N-QA-06, REQ-N-ARC-06, REQ-N-ARC-07, REQ-N-ARC-09, DEC-004,
DEC-023, DEC-039.

| Step | Commit | Change | Test |
|------|--------|--------|------|
| 1.1 | `chore(core): scaffold package with pinned toolchain` | `package.json` (exact devDependencies, `engines.node` 24, `"type": "module"`), lockfile, `.nvmrc`, strict `tsconfig` (§3), `src/index.ts` | REQ-N-QA-05 every devDependency is an exact version |
| 1.2 | `build(core): emit ESM and declarations with subpath exports` | `tsc` build to `dist/`, `exports` map: `.`, `./contract`, `./api-client`, `./validation`, `./sync`, `./sync/testing`, `./i18n`, `./tokens` | REQ-N-ARC-07 each subpath resolves from the packed tarball |
| 1.3 | `chore(core): enforce platform-agnostic boundaries` | ESLint flat config (`typescript-eslint` strict type-checked) + §3 restricted imports and folder direction | REQ-N-ARC-07 lint rejects fixture files importing `react`, `react-native`, `node:fs` and `sync` from `contract` |
| 1.4 | `test(core): guard against runtime dependencies` | Manifest test | REQ-N-ARC-06 package has no runtime or peer dependencies |
| 1.5 | `ci(core): run lint, types, tests, coverage, audit and build` | `.github/workflows/ci.yml` (job `ci`, actions pinned by SHA, `permissions: contents: read`), coverage thresholds (≥ 90 % `sync/` + `api-client/`, ≥ 80 % overall), `npm audit --audit-level=high`, Dependabot (npm + actions), `.githooks/pre-push` | REQ-N-QA-03 coverage thresholds are configured as ADR-014 §7 requires |
| 1.6 | `ci(core): check Conventional Commits and cited IDs` | `scripts/check-commits.ts` (run by Node 24 type stripping, no extra tool) for each PR commit and the PR body | REQ-N-QA-06 rejects non-conventional subjects and PR bodies without a REQ/DEC/ADR ID |
| 1.7 | `ci(core): export the REQ-to-tests matrix` | `scripts/req-matrix.ts` reads test titles → JSON artifact | REQ-N-QA-06 extracts IDs from nested `describe`/`it` titles |
| 1.8 | `ci(core): publish to GitHub Packages on version tags` | `release.yml` on `v*`: tag must equal `package.json` version, then `npm publish` with `packages: write` only in that job | REQ-N-ARC-07 release check rejects a tag that differs from the package version |

**Done when:** PR green; Camilo adds `ci` as required check; tag `v0.1.0` publishes and a scratch project installs it
with a `read:packages` token.
**Concepts:** npm package anatomy and `exports` maps; ESM vs CJS; `tsconfig` `lib` as an architecture fence;
supply-chain basics (SHA-pinned actions, least-privilege tokens); registries and auth.

### Phase 2 — Contract types

**Goal:** `src/contract/` generated from one pinned, checksummed contract version.
**REQ:** REQ-N-ARC-03, REQ-N-ARC-04, ADR-003 §3 and §5. **Source (OQ-H2-1):** API-001 from `cmlooz/cherryLove-docs` pinned at a commit SHA + sha256
until plan F publishes the first `contract-v1.*` asset (F0.12); then `contract.lock.json` switches to it.

| Step | Commit | Change | Test |
|------|--------|--------|------|
| 2.1 | `build(core): pin the contract source with a checksum` | `contract.lock.json` (`version`, `url`, `sha256`), `scripts/fetch-contract.ts` | REQ-N-ARC-03 fetch fails on checksum mismatch |
| 2.2 | `feat(core): generate contract types with openapi-typescript` | Generated `src/contract/openapi.ts` (`--enum-values`) and generated `index.ts` exporting `CONTRACT_VERSION` | REQ-N-ARC-03 type tests: `ErrorCode` includes `cursor_expired`; fields stay `snake_case` |
| 2.3 | `ci(core): fail when generated types drift from the pinned contract` | CI regenerates and runs `git diff --exit-code` | REQ-N-ARC-03 drift script reports a hand edit in `contract/` |

**Done when:** types compile; drift check green; release `0.2.0`.
**Concepts:** contract-first vs code-first; types-only codegen (zero runtime); reproducible builds through
checksums; why `snake_case` in TS (ADR-003 §5).

### Phase 3 — API client

**Goal:** a thin hand-written client typed by `paths`, with request_id, typed problems and pluggable auth.
**REQ:** REQ-N-QA-01, REQ-N-QA-02, REQ-N-SEC-03, REQ-N-ARC-04, REQ-N-ARC-09, ADR-004 §1–2, ADR-010 §5.

| Step | Commit | Change | Test |
|------|--------|--------|------|
| 3.1 | `feat(core): add HttpTransport port and request pipeline` | Minimal own `HttpRequest`/`HttpResponse` types; `createApiClient({ baseUrl, http, auth, ids })` | REQ-N-ARC-04 builds method, path params, query and JSON body from contract types |
| 3.2 | `feat(core): send and capture X-Request-ID` | ID per request; response echo exposed on results and problems | REQ-N-QA-01 header sent and server value surfaced |
| 3.3 | `feat(core): parse Problem Details into a typed ApiProblem` | Guard clauses on content type and shape (no `try/catch` for flow); unknown `code` → `unknown_code` with `request_id`; non-JSON 5xx → synthetic `internal_error` | REQ-N-QA-02 fast-check: any body yields a value, never a throw |
| 3.4 | `feat(core): add AuthTransport port with single-flight refresh` | 401 `token_expired` → one refresh → retry once; concurrent 401s share one refresh; failure → `onSessionEnded` | REQ-N-SEC-03 ten parallel 401s trigger exactly one refresh |
| 3.5 | `feat(core): add web cookie and mobile secure-store strategies` | Web: `credentials: include` + `X-CherryLove-Client: web`, access in memory; mobile: refresh in body via `SecureTokenStore` port | REQ-N-SEC-03 web never reads a refresh token; mobile rotates the stored one |
| 3.6 | `feat(core): add typed endpoint functions per module` | One file per API-001 tag; names follow `operationId` | REQ-N-ARC-03 table test: every API-001 `operationId` has one function |
| 3.7 | `test(core): integrate client with a contract-shaped fake server` | Phase integration test (DEC-023) | REQ-N-QA-02 login → 401 → refresh → retry → problem flow end to end |

**Done when:** ≥ 90 % coverage of `api-client/`; release `0.3.0`.
**Concepts:** ports and adapters on the client; why `fetch` is injected; single-flight concurrency; RFC 9457;
validation-before-`try/catch` in practice.

### Phase 4 — Validation

**Goal:** client-side input checks that mirror contract constraints and speak the server's error shape.
**REQ:** REQ-N-ARC-09, REQ-N-ARC-03, REQ-F-SYNC-03, ADR-010 §1. Entity REQ IDs (REQ-F-CARD/MEM/SONG/PLACE/NOTE/FAV)
cited per validator in step 4.3.

| Step | Commit | Change | Test |
|------|--------|--------|------|
| 4.1 | `feat(core): add field error model aligned with Problem errors` | `{ pointer, code, params }` (same as ADR-010 `errors[]`), tiny combinators, no library | REQ-N-ARC-09 client and server errors map to fields the same way |
| 4.2 | `feat(core): validate contract primitives` | UUIDv7, civil date (real calendar day), timestamp, email, `https` URL, enums | REQ-F-SYNC-03 fast-check: generated UUIDv7 always valid, mutated version nibble rejected |
| 4.3 | `feat(core): validate inputs of syncable entities` | One validator per `*Input` schema | per-entity REQ: limits and required fields |
| 4.4 | `test(core): check constraint parity with the pinned contract` | Reads limits from the contract's JSON asset with `JSON.parse` (OQ-H2-4); enabled once the first `contract-v1.*` release exists | REQ-N-ARC-03 every `maxLength`/`enum` in an `*Input` matches its validator |

**Done when:** release `0.4.0`.
**Concepts:** where validation belongs (shape on client, rules on server — server wins, REQ-F-SYNC-10); JSON Pointer;
property-based testing basics.

### Phase 5 — i18n engine

**Goal:** the ADR-012 translator and checks, proven on fixture catalogs (real copy in phase 9).
**REQ:** REQ-N-UX-03, REQ-F-ACC-04, DEC-021, DEC-024, ADR-012 §1–3, ADR-010 §5–6.

| Step | Commit | Change | Test |
|------|--------|--------|------|
| 5.1 | `feat(core): add typed translator with interpolation and plurals` | Keys typed from `en.json`; `{param}` names typed with template literal types; `_one`/`_other` | REQ-N-UX-03 unknown key fails to compile (type test); plurals pick by `count === 1` |
| 5.2 | `feat(core): resolve the active language` | Device locale `es*` → `es`, else `en`; `user.language` overrides | REQ-F-ACC-04 account language beats device locale |
| 5.3 | `feat(core): format dates and numbers with Intl` | `es-CO` / `en-US`; civil dates formatted without timezone shifts | DEC-024 a civil date shows the same day in a DST timezone |
| 5.4 | `feat(core): translate ApiProblem by code` | `errors.<code>` + `params`; unknown code → generic message + `request_id` | REQ-N-QA-02 unknown code shows the generic message with request_id |
| 5.5 | `ci(core): check catalog parity, params and contract codes` | Script + compile-time `Record<ErrorCode, …>`; runs on fixtures until phase 9 | REQ-N-UX-03 fails on a missing key, a mismatched `{param}` and an untranslated code |

**Done when:** release `0.5.0`.
**Concepts:** typed keys without codegen; template literal types; why civil dates must not touch the device
timezone; "build vs buy" for i18n (ADR-012).

### Phase 6 — Sync: local writes and push

**Goal:** offline writes land atomically in a persistent outbox and replay safely.
**REQ:** REQ-F-SYNC-02, REQ-F-SYNC-03, REQ-F-SYNC-04, REQ-F-SYNC-05, REQ-F-SYNC-09, REQ-F-SYNC-10, REQ-F-CARD-08,
ADR-006 §2–3, §6, §8.

| Step | Commit | Change | Test |
|------|--------|--------|------|
| 6.1 | `feat(core): define sync ports and in-memory fakes` | `sync/ports.ts` (`LocalStore` with transactions, `Network`, `Clock`, `RandomSource`); fakes and a **port conformance suite** exported from `./sync/testing` for the IndexedDB and SQLite adapters | REQ-F-SYNC-09 in-memory store passes the conformance suite |
| 6.2 | `feat(core): generate UUIDv7 identifiers` | RFC 9562 layout, monotonic within a millisecond | REQ-F-SYNC-03 fast-check: version 7, variant bits, sortable by time |
| 6.3 | `feat(core): map entity types to typed records` | `entity_type` → contract schema map; guard for untyped `payload`/`record` (API-001 types them as open objects) | REQ-F-SYNC-05 malformed record is reported, not applied |
| 6.4 | `feat(core): apply local writes through the outbox` | Validate → optimistic apply + outbox entry (`client_op_id`) in **one** store transaction; record state `pending` | REQ-F-SYNC-02 failed transaction leaves neither record nor outbox entry |
| 6.5 | `feat(core): push the outbox in ordered batches` | ≤ 100 ops; `applied` → synced + version; `rejected` → server truth restored, problem stored; `gone` → drop; later ops on a rejected entity rejected too | REQ-F-SYNC-10 rejected op keeps its problem for later display |
| 6.6 | `test(core): replay pushes idempotently` | Same batch delivered twice / response lost | REQ-F-SYNC-04 replay applies each `client_op_id` once |

**Done when:** ≥ 90 % coverage of `sync/`; release `0.6.0`.
**Concepts:** outbox pattern; atomicity on the client; idempotency keys; last-write-wins vs field merge; contract
tests for ports (one suite, many adapters).

### Phase 7 — Sync: pull and cycle

**Goal:** no missed change, safe recovery from an expired cursor, visible state.
**REQ:** REQ-F-SYNC-01, REQ-F-SYNC-05, REQ-F-SYNC-06, REQ-F-SYNC-08, REQ-F-CARD-06, REQ-F-FAV-04, REQ-F-NOTE-02,
ADR-006 §4–7, ADR-014 §2 and §4.

| Step | Commit | Change | Test |
|------|--------|--------|------|
| 7.1 | `feat(core): pull changes by cursor` | Loop until `next_cursor = null`; changes + cursor saved in one transaction; `pending_card_count` stored | REQ-F-SYNC-06 interrupted pull resumes from the last saved cursor |
| 7.2 | `feat(core): apply deletes and invisible records` | `op: delete` removes local copy (tombstone or no longer visible) | REQ-F-NOTE-02 partner's `only_me` note vanishes on pull |
| 7.3 | `feat(core): recover from cursor_expired` | 410 → reset entities and cursor, full resync, **outbox kept** and replayed afterwards (OQ-H2-5) | REQ-F-SYNC-06 410 with pending ops: full resync, then the ops replay; nothing written offline is lost |
| 7.4 | `feat(core): schedule the sync cycle` | Push then pull; triggers (foreground, reconnect, local write debounced 2 s, 60 s timer); backoff with jitter capped at 5 min for network/5xx/429 (`Retry-After` respected) | REQ-F-SYNC-02 fake clock: debounce, timer and backoff cap |
| 7.5 | `feat(core): expose sync status` | Global `offline / pending / synced`, per record `pending / synced / rejected`; tiny own observer (no RxJS) | REQ-F-SYNC-08 status transitions follow the cycle |
| 7.6 | `test(core): property-test sync invariants` | fast-check model-based commands against a model server: random interleavings of writes, drops, duplicates, restarts | REQ-F-SYNC-05 no skipped change, deletes win, outbox order kept, replay idempotent |
| 7.7 | `test(core): integrate engine, client and fake server` | Phase integration test | REQ-F-SYNC-01 offline write → reconnect → partner device sees it |

**Done when:** ≥ 90 % coverage of `sync/`; property suite runs with a fixed seed in CI; release `0.7.0`.
**Concepts:** cursors and why timestamps fail (ADR-006); model-based property testing; backoff with jitter;
deterministic time through a port.

### Phase 8 — Media upload queue

**Goal:** the separate upload queue of ADR-007 §2–3, gating dependent outbox ops.
**REQ:** REQ-F-SYNC-07, REQ-F-MEM-10, REQ-N-SEC-06, REQ-N-PERF-02, DEC-020, ADR-006 §6, ADR-007 §2–5.

| Step | Commit | Change | Test |
|------|--------|--------|------|
| 8.1 | `feat(core): add upload ports and persistent queue` | `FileSource`, `StorageUploader` ports (FormData differs on web and React Native); queue in `LocalStore` | REQ-F-SYNC-07 queue survives a restart |
| 8.2 | `feat(core): run the three-step upload flow` | Reserve (`media_create_upload`) → presigned POST → `media_complete_upload` → `ready`/`rejected` (by pull if plan F's OQ-008 adds `media_asset` to `SyncEntityType`, else polling `media_get_asset`) | REQ-F-MEM-10 happy path reaches `ready` |
| 8.3 | `feat(core): hold outbox ops until their media is ready` | Dependency check in push | REQ-F-SYNC-07 memory op waits for its upload |
| 8.4 | `feat(core): classify upload failures` | Retry network/5xx with backoff; `quota_exceeded`, `file_too_large`, `media_rejected` are final → record `rejected` | DEC-020 quota rejection is final and visible |
| 8.5 | `feat(core): batch signed URL requests` | Batch IDs per frame, cache until expiry minus margin | REQ-N-SEC-06 twenty thumbnails → one request; expired URL refetched |
| 8.6 | `test(core): integrate uploads with the sync cycle` | Phase integration test | REQ-F-SYNC-07 offline memory with photo syncs after upload |

**Done when:** release `0.8.0`.
**Concepts:** presigned POST and why the server never proxies bytes; dependency ordering between queues; request
batching to avoid N+1 on the client.

### Phase 9 — "Cherry" content (starts only after UX-001 is approved)

**Goal:** the approved tokens and copy, with the catalog checks turned into hard gates.
**REQ:** REQ-N-UX-01, REQ-N-UX-02, REQ-N-UX-03, REQ-N-UX-04, DEC-021.

| Step | Commit | Change | Test |
|------|--------|--------|------|
| 9.1 | `feat(core): add Cherry design tokens` | Typed constants (color, typography, spacing, radius) as plain numbers/strings, no CSS or RN style objects | REQ-N-UX-01 tokens match the UX-001 table |
| 9.2 | `test(core): check WCAG contrast of token pairs` | Contrast ratio computed for every declared foreground/background pair | REQ-N-UX-02 every text pair ≥ 4.5:1 (≥ 3:1 large text) |
| 9.3 | `feat(core): add es and en catalogs` | Approved screen copy + `errors.<code>` for every contract code | REQ-N-UX-03 catalogs parse and cover approved screens |
| 9.4 | `ci(core): make catalog checks block on real catalogs` | Phase 5 checks switch from fixtures to `es.json`/`en.json` | REQ-N-UX-03 CI fails on a removed `errors.*` key |

**Done when:** release `1.0.0` with `contract-v1.0.0` pinned (OQ-H2-2).
**Concepts:** design tokens as a platform-neutral contract; WCAG contrast math; copy as a reviewed artifact.

Later copy and token changes follow the same gate (REQ-N-UX-04) and ship as minor releases.

## 6. Open questions raised by this plan

All answered by Camilo on 2026-11-13. Provisional IDs: plan F already took OQ-005…010, so final numbers are assigned
when these rows are copied into `02-requirements/open-questions.md` (this session writes only under
`06-implementation/core/`).

| ID | Question | Answer (option A in every case) | Applies to |
|----|----------|---------------------------------|------------|
| OQ-H2-1 | ADR-003 §3 says core pins a released `contract-v1.x.y` asset, but none exists until plan F publishes one. What does phase 2 pin meanwhile? | Pin API-001 from `cmlooz/cherryLove-docs` at a **commit SHA** (raw URL, public repo) with its sha256; switch `contract.lock.json` to the first `contract-v1.*` asset when F publishes it. Temporary bridge; the contract stays fixed and verifiable. | Phase 2 |
| OQ-H2-2 | ADR-003 §3 says core "SemVer mirrors the contract", but core also changes on its own. Exact rule? | `0.y.z` while building; from `1.0.0` (phase 9 + `contract-v1.0.0`): core major = contract major; contract additive or core feature → minor; fix → patch; core-only breaking changes inside v1 go through deprecation; `CONTRACT_VERSION` exported. | §4, steps 1.8, 9.x |
| OQ-H2-3 | Coverage needs `@vitest/coverage-v8`; `vitest` needs `vite` as a peer; ADR-016 pins neither for core. | Patch ADR-016: core dev deps `@vitest/coverage-v8` **5.1.0** and `vite` **8.3.8**. Dev only. The patch is written by the owner of `04-architecture/`, not this session. | Phase 1 |
| OQ-H2-4 | Constraint checks must read the contract at test time; Node has no YAML parser. | Plan F publishes the contract asset **also as JSON** (F0.12); core reads it with `JSON.parse`, zero dependencies. Step 4.4 is enabled when that asset exists. | Step 4.4 |
| OQ-H2-5 | ADR-006 §4: on 410 "the client wipes its local store". Does that include unpushed outbox ops? | Wipe entities and cursor, **keep the outbox**, full resync, then replay it. Ops carry full records and are idempotent by `client_op_id`; the server answers `gone`/`rejected` where they no longer apply. Interpretation of ADR-006 §4, consistent with REQ-F-SYNC-02. | Step 7.3 |
| OQ-H2-6 | API-001 types `Problem.errors[].code` as a free string, so field-error translations cannot be checked. | Plan F adds an additive `FieldErrorCode` enum to the contract (F0.8, contract change approved by Camilo in that PR); core validators may emit only those codes and CI checks `validation.<code>` in es and en. | Steps 4.1, 5.5, 9.3, 9.4 |

## 7. Notes for other plans

| Plan | Note |
|------|------|
| F | F0.12: also publish the contract asset as JSON (OQ-H2-4); core switches its pin to the first `contract-v1.*` release (OQ-H2-1). F0.8: add `FieldErrorCode` (OQ-H2-6). F's OQ-008 (`media_asset` sync type) decides how step 8.2 learns the upload status. |
| G, H | Run the `./sync/testing` conformance suite against the IndexedDB / SQLite adapters. Inject `fetch`, timers and random bytes through core ports. |
| H | `RandomSource` needs `crypto.getRandomValues` on Hermes (e.g. an Expo SDK module, not in ADR-016 yet); confirm Metro package `exports` support and Jest transform of an ESM package; confirm ES2022 built-ins on Hermes. |

## Next step

Approved by Camilo on 2026-11-13. Phase 1 starts in `cherrylove-core` (re-verify pins first, REQ-N-QA-05); phase 9
starts only after UX-001 is approved.
