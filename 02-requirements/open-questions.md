---
id: REQ-DOC-004
status: approved
approved_by: Camilo
approved_at: 2026-11-11
---

# Open questions

All v0 questions were answered on 2026-11-09. Decisions live in `00-governance/decision-log.md`.

## Still open (blocks freeze)

None.

## Still open (post-freeze, does not block)

| ID | Question | Recommendation |
|----|----------|----------------|
| OQ-002 | Staff accounts can archive couples and see metadata (REQ-F-ADM-*), but no frozen requirement asks for a second factor on staff login (raised by ADR-004 §5). | Add TOTP 2FA for staff in v1 as a new REQ ID. Staff are few, the admin is the most privileged surface, and Django has mature TOTP support. |
| OQ-005 | REQ-F-ADM-03 offers an export before the 90-day purge, but REQ-F-AUTH-09 forbids login for an archived couple and DEC-015 forbids staff from seeing content. Who receives the export, how, and in what format? (raised by IMPL-API-001 F9.6) | On archive, a job builds a ZIP per couple (JSON of the couple's content, decrypted with the DEK, plus media originals) and emails each partner a single-use download link valid 7 days. An admin action "resend export link" issues a new link without ever showing content to staff. |
| OQ-006 | ADR-003 §2 says CI fails when the generated spec ≠ the approved contract, but the API is built phase by phase, so early phases cannot implement every API-001 operation. (raised by IMPL-API-001 F0.9) | Drift check compares every **implemented** operation exactly and fails on any operation not in the contract; operations not yet built are listed in `contract/pending-operations.txt`, a list that may only shrink and must be empty at the end of F9. |
| OQ-007 | REQ-F-LOG-01 logs every "view" and REQ-F-ADM-06 counts "items visited", but clients read synced content from the local store (ADR-006, ADR-011 §4), so the server rarely sees a view. (raised by IMPL-API-001 F0.8/F9.7) | Add an additive contract endpoint for batched client-reported views (`entity_type`, `entity_id`, `viewed_at`; no content), queued offline like other ops; server-observed GETs are logged too. Alternative: server-observed views only, accepting near-zero "visited" counts. |
| OQ-008 | ADR-007 §2 says a media `ready`/`rejected` status reaches devices through a sync change, but API-001's `SyncEntityType` has no media type. (raised by IMPL-API-001 F0.8/F4.7) | Add `media_asset` to `SyncEntityType` (additive change inside v1, ADR-003 §4) so pull delivers status and rejection code; `media_get_asset` stays for on-demand checks. |
| OQ-009 | REQ-F-SONG-06/07 let the song form link several memories, but REQ-F-MEM-07 allows editing own memories only and ADR-006 relies on "only the author writes a record". May a partner link the other partner's memories to their song? (raised by IMPL-API-001 F4.12) | No: the song form links only the author's own memories; the song's gallery still shows linked memories of both partners. Keeps the single-writer rule that makes sync conflict-free across partners. |
| OQ-010 | Yearly reminders (REQ-F-NOTIF-02, DEC-036) for a relationship start or birth date on 29 February have no date in non-leap years. (raised by IMPL-API-001 F8.9) | Send on 28 February in non-leap years. |
| OQ-011 | REQ-F-ACC-03 and ADR-004 §3 revoke every session **except the current one** on password change, but `identity_change_password` (`/me/password`, 204) receives no refresh token: the web cookie is scoped to `Path=/api/v1/auth/` and mobile does not send it. (raised by IMPL-API-001 F2.16) | Add a `sid` claim (refresh token family id) to the access token, so the server keeps that family and revokes the rest; no contract change, small amendment to ADR-004 §1 "minimal claims". Alternative: revoke all sessions and make the user log in again. |
| OQ-012 | REQ-F-CARD-06 schedules cards by `publish_at`, but nothing says what happens when a card is created or edited with a `publish_at` already in the past ("send now", or an offline card replayed late). (raised by IMPL-API-001 F3.3) | Allow it: the same use case publishes the card at once through the publish code path (change recorded, push enqueued from F8). Alternative: reject with 400, simpler but it penalizes offline writes. |

## Resolved open questions

| ID | Resolved by | Outcome |
|----|-------------|---------|
| OQ-001 | DEC-034, REQ-N-PERF-06 | Load-data generator in v1 with a 1 M-row seed per hot table (2026-11-11). |
| OQ-003 | DEC-036 | Anniversary reminder is yearly in v1 (2026-11-12). |
| OQ-004 | DEC-037 | MinIO dropped; local S3-compatible server chosen by a plan F spike with a presigned-POST acceptance test (2026-11-12). |

## Resolved v0 questions

| v0 ID | Resolved by | Outcome |
|-------|-------------|---------|
| Q-CARD-001 | DEC-006, REQ-F-CARD-01/02 | Carousel if several; random past card if none (v0 rec "newest wins / empty" rejected). |
| Q-MAP-001 | REQ-F-PLACE-04 | Star favorites the place (proposal text already says so). |
| Q-OFFLINE-001 | REQ-F-SYNC-09 | SQLite on mobile, IndexedDB on web. |
| Q-MEDIA-001 | DEC-009, DEC-010 | Object storage (not Django disk) + local cache; mixed music sources; no OAuth in v1. |
| Q-EDIT-001 | REQ-F-* "own only" rows | Author edits/deletes; both read and favorite. |
| Q-DIARY-001 | DEC-014 | `shared` or `only_me` (v0 rec "all shared" rejected). |
| Q-FAV-001 | DEC-011, REQ-F-FAV-01 | Personal favorites; memories grouped by date. |
| Q-TZ-001 | DEC-024 | Couple timezone, civil dates. |
| Q-INVITE-001 | DEC-028, REQ-F-AUTH-10 | Hard reject. |
| Q-ADM-001 | DEC-015 | Metadata only; no break-glass in v1. |
| Q-DB-001 | REQ-N-ARC-08 | PostgreSQL behind repositories. |
| Q-API-001 | DEC-029 | Django REST Framework. |
| Q-ADMIN-UI-001 | DEC-005 | Django Admin + custom views. |
| Q-I18N-001 | DEC-021, DEC-030 | UI Spanish + English; docs and code in English. |
| Q-MAP-002 | DEC-013 | OpenStreetMap. |
| Q-SOFTDEL-001 | REQ-F-SYNC-05 | Soft delete everywhere (tombstones). |
| X-001 | REQ-N-ARC-03 | Accepted. |
| X-002 | DEC-003 | Accepted in spirit: shared `cherrylove-core` (types, client, validation, sync, tokens, i18n); no React Native Web. |
| X-003 | REQ-N-QA-01, REQ-F-LOG-01 | Accepted. |
| X-004 | REQ-N-UX-01 | Accepted. |
