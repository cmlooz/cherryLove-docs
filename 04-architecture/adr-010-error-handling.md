---
id: ADR-010
status: approved
approved_by: Camilo
approved_at: 2026-11-11
---

# ADR-010 — Error handling (RFC 9457 Problem Details)

Implements: REQ-N-QA-02, REQ-N-QA-01, REQ-N-ARC-09, REQ-N-UX-03, REQ-F-SYNC-10, REQ-F-AUTH-08.
Builds on: ADR-003 §5 (`application/problem+json`), ADR-005 §4 (404/403/409), ADR-006 §2 (per-op results).

## Context

Errors cross three borders: domain → HTTP, HTTP → `cherrylove-core`, core → a translated message on screen (in
Spanish or English). Sync adds a fourth: a rejection may arrive minutes later, for an action the user made offline.
REQ-N-ARC-09 adds a style rule: *validation before `try/catch`* — errors are predicted and checked, not
discovered by catching whatever explodes.

## Decision drivers

1. One error shape everywhere: REST responses and sync per-op results.
2. Clients branch on a **stable machine code**, never on English text.
3. Every message the user sees is translatable (es/en) and lives with the rest of the UI copy.
4. Internals never leak (stack traces, SQL, file paths).
5. No new dependency.

## Options considered

| Topic | Options | Choice |
|-------|---------|--------|
| Domain error style | Exceptions with stable codes ⭐ · `Result`/`Either` return types | Exceptions: idiomatic Python, one central mapper. `Result` types would add ceremony to every call without mypy exhaustiveness. Guard clauses validate *before* acting (REQ-N-ARC-09). |
| Who translates | Client by `code` ⭐ · server by `Accept-Language` | Client: UI copy already lives in `cherrylove-core/i18n` (REQ-N-UX-03); offline rejections arrive later, in whatever language the user has *then*. |
| DRF integration | Own exception handler ⭐ · `drf-standardized-errors` | Own handler (~1 file): the shape is small and fixed; a library would be a dependency for it. |

## Decision

### 1. Problem shape

```json
{
  "type": "/problems/card-already-published",
  "title": "Card already published",
  "status": 409,
  "detail": "Published cards are read-only.",
  "instance": "/api/v1/cards/0193…",
  "code": "card_already_published",
  "request_id": "9f1c…",
  "params": {},
  "errors": []
}
```

| Member | Rule |
|--------|------|
| `type` | Relative URI `/problems/<code-kebab>`; one per code. A docs page may live there later. |
| `title`, `detail` | English, for developers and logs. **Never shown to users.** |
| `code` | Stable `snake_case` identifier; part of the contract (ADR-003). Renaming = breaking change. |
| `params` | Values for message interpolation (`{ "max_mb": 20 }`). Never content. |
| `errors` | Validation only: `[{ "pointer": "/body", "code": "too_long", "params": { "max": 2000 } }]` (JSON Pointer). |
| `request_id` | Always present (REQ-N-QA-01); shown to the user on unexpected errors for support. |

### 2. Status codes

| Status | When | Example codes |
|--------|------|---------------|
| 400 | Malformed or invalid input (shape, format, ranges) | `validation_failed` |
| 401 | Missing/expired/invalid token | `not_authenticated`, `token_expired` |
| 403 | Visible item, forbidden action (ADR-005 §4) | `not_author` |
| 404 | Not found **or not visible** (ADR-005 §4) | `not_found` |
| 409 | Business rule conflict with current state | `card_already_published`, `quota_exceeded`, `invite_expired`, `email_already_in_couple` |
| 410 | Resource or cursor gone | `cursor_expired`, `entity_deleted` |
| 429 | Throttled; `Retry-After` header set | `rate_limited` |
| 500 | Unexpected — generic body, details only in server logs | `internal_error` |
| 503 | Maintenance / dependency down | `service_unavailable` |

Exception: password recovery returns the same success response whether or not the email exists (REQ-F-AUTH-08);
it never produces a "user not found" problem.

### 3. Server flow

1. Serializers validate **shape** → DRF `ValidationError` → 400 with `errors[]`.
2. Use cases check **preconditions** with guard clauses and raise typed errors from `<module>/domain/errors.py`
   (each class carries `code`, `status`, default `params`).
3. One handler, `shared/interfaces/http/problem_details.py` (DRF `EXCEPTION_HANDLER`), maps: domain errors, DRF
   errors (`NotAuthenticated`, `Throttled`, `ValidationError`, `Http404`) and anything else → 500.
4. **No `try/except` in views or use cases** to turn errors into responses. `try` is allowed only in adapters,
   around I/O (storage, email, push), to translate a vendor error into a port error.
5. 500s log the stack trace with `request_id` (ADR-008 §7); the response body contains only `code`, `title`,
   `status`, `request_id`.

### 4. Sync

A `rejected` op in `/sync/push` (ADR-006 §2) embeds the **same problem object**. The device stores it on the
record (`rejected` state) so the reason can be shown later, translated at display time.

### 5. Client flow (`cherrylove-core`)

- `api-client` parses every `application/problem+json` into a typed `ApiProblem` (generated `code` union).
- UI resolves the message from `i18n` key `errors.<code>` with `params`.
- Unknown `code` (older app, newer server) → generic "something went wrong" message + `request_id`.
- Field errors map by `pointer` to form fields.

### 6. Catalog check

Error codes are listed in the contract (API-001, an enum per response). CI in `cherrylove-core` fails if any code
lacks an `es` or `en` translation (REQ-N-UX-03).

## Consequences

- ✅ One shape for REST and sync; clients branch on codes that cannot change silently.
- ✅ Messages translated with the rest of the UI, including late offline rejections.
- ✅ Views and use cases stay free of error-plumbing `try/except`.
- ✅ No new dependency.
- ❌ Every new error is three edits: domain class, contract enum, two translations — enforced by CI.
- ⚠️ `title`/`detail` are English only; staff in Django Admin see Django's own messages, not these.

## Next step

Camilo approves, rejects, or patches by section number. Then ADR-011 (caching).
