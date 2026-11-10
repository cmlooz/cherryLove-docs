---
id: REQ-DOC-004
status: in_review
approved_by: null
approved_at: null
---

# Open questions

All v0 questions were answered on 2026-11-09. Decisions live in `00-governance/decision-log.md`.

## Still open (blocks freeze)

| ID | Question | Recommendation |
|----|----------|----------------|
| OQ-001 | REQ-N-PERF-03 asks for "large record counts", but the load-data generator went to the backlog (DEC-027). Without it the requirement is not verifiable. | Move the generator into v1 with a **1 M-row** seed per hot table. Small effort; it is the performance lesson itself. |

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
