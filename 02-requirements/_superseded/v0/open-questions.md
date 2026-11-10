<!-- superseded on 2026-11-09 by the current file of the same name; kept for history (GOV-001) -->
---
status: draft
approved_by: null
approved_at: null
---

# Open questions — blocks requirements freeze

Answer APPROVE rec, CHANGE, or DEFER to ADR per ID. Until then, no implementation and no architecture freeze.

| ID | Gap | Recommendation |
|----|-----|----------------|
| Q-CARD-001 | Daily card: scheduled publish vs daily draw from a pool? | Pool of cards with publish_at. Today = those dated couple-today; if many, latest; if none, empty. |
| Q-MAP-001 | Map star text said favorite the song. | Star favorites the place. |
| Q-OFFLINE-001 | Local store: JSON vs SQLite. | SQLite (mobile) + same logical schema on web (SQLite WASM or IndexedDB adapter). Not JSON files. |
| Q-MEDIA-001 | Local upload vs remote URL vs Spotify/YouTube. | v1 store MediaRef only. No blobs in Django. No OAuth in v1. |
| Q-EDIT-001 | Who may edit/delete? | Author only. Either partner may read published couple content and favorite. |
| Q-DIARY-001 | Shared diary vs private notes. | All notes shared with the couple. No private notes in v1. |
| Q-FAV-001 | Favorite grouping key. | Group memories by (couple_id, memory_date); subgroup by place_id when set. |
| Q-TZ-001 | Whose timezone is today and days together? | Couple timezone, set at onboarding. Civil dates, not hours. |
| Q-INVITE-001 | Invitee already in a couple? | Hard reject. No couple-switch in v1. |
| Q-ADM-001 | May platform admin read letter/diary bodies? | Counts + metadata only; break-glass audited if ever needed. |
| Q-DB-001 | Server database engine. | PostgreSQL. Domain talks to repositories, not to Postgres. |
| Q-API-001 | Django API style. | Prefer Django Ninja for typing. DRF is the alternative. You pick. |
| Q-ADMIN-UI-001 | Admin UI. | Django Admin + custom metric views. Not a second React admin in v1. |
| Q-I18N-001 | UI language. | Spanish UI, English docs/code identifiers — unless you want English UI. |
| Q-MAP-002 | Map tiles provider. | Domain stores lat/lng + label. Web: Leaflet/OSM in v1. |
| Q-SOFTDEL-001 | Hard vs soft delete. | Soft delete on memories, songs, notes, places. |

## Extra proposals (yes/no)

| ID | Proposal |
|----|----------|
| X-001 | OpenAPI as constitution of the polyrepo; generate TS types for web and mobile. |
| X-002 | Shared package later = types + policies only, not React-Native-Web in v1. |
| X-003 | Request id on every API call; action log from the first endpoint. |
| X-004 | Design tokens (cherry, cream, album motion) before JSX. |
