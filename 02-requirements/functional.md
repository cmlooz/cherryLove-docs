---
id: REQ-DOC-001
status: in_review
approved_by: null
approved_at: null
supersedes: functional.md draft v0 (REQ-F-001…REQ-ADM-005)
---

# Functional requirements

Every ID below is pending the **requirements freeze**. Product rules behind each ID were already approved
one by one on 2026-11-09 and are logged in `00-governance/decision-log.md` (DEC-*).

## Quick path

1. Read by module. Each row cites the decision it comes from (`DEC-*`) or `you` (original proposal).
2. Reply `freeze` to approve the whole catalog, or `REQ-F-XXX-NN change to …` per row.
3. After freeze, IDs are immutable; changes need a new ID and a decision-log entry.

## ID scheme

`REQ-F-<MODULE>-<NN>`. Modules: `AUTH`, `HOME`, `CARD`, `MEM` (album), `SONG`, `PLACE` (map), `NOTE` (diary),
`FAV`, `ACC` (account), `NOTIF`, `SYNC`, `ADM`, `LOG`.

---

## AUTH — Invitation, onboarding, session

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-AUTH-01 | A platform admin creates a couple with **two emails**; the system emails an invitation to each. | you, DEC-028 |
| REQ-F-AUTH-02 | An invite is a single-use link that expires after **72 h**; admin can resend or revoke it. | DEC-017 |
| REQ-F-AUTH-03 | Onboarding form asks: name, birth date, password, relationship start date. | you |
| REQ-F-AUTH-04 | Relationship start date belongs to the couple: if the first partner filled it, **the second is not asked**. | you |
| REQ-F-AUTH-05 | Max two users per couple; one couple per user; **no couple switching**. Enforced by database constraints, not only code. | you |
| REQ-F-AUTH-06 | The first registered partner can already use the app; the second sees all shared content once onboarded. | DEC-018 |
| REQ-F-AUTH-07 | Login and logout with email + password; every attempt (success or failure) is logged. | you |
| REQ-F-AUTH-08 | Password recovery by email with a short-lived single-use token; identical response whether the email exists or not. | DEC-026 |
| REQ-F-AUTH-09 | Users of an archived couple cannot log in. | DEC-019 |
| REQ-F-AUTH-10 | An invite for an email that already belongs to a couple is rejected. | DEC-028 |

## HOME — Dashboard

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-HOME-01 | Header: `"{user1.name} y {user2.name}"` + days-together counter (civil days from relationship start, couple timezone). | you, DEC-024 |
| REQ-F-HOME-02 | Timeline of favorites of **both** partners, ordered by date, showing who starred each item. | DEC-011 |
| REQ-F-HOME-03 | Clicking a timeline item opens its detail in its module. | you |
| REQ-F-HOME-04 | Entry points to every module. | you |

## CARD — Daily cards (2 screens)

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-CARD-01 | **Today screen:** shows cards published today; if several, a **carousel** in the same order for both partners. | you, DEC-006 |
| REQ-F-CARD-02 | If none is published today, show one **random past card**, the same for both partners (server-side deterministic draw, seed = couple + civil date). | you |
| REQ-F-CARD-03 | Each card shows content, author, and created_at. | you |
| REQ-F-CARD-04 | "Today" is computed in the **couple timezone** (configurable, default `America/Bogota`). | DEC-024 |
| REQ-F-CARD-05 | **My cards screen:** create and edit; list of the cards each partner created. | you |
| REQ-F-CARD-06 | `publish_at` scheduling. The partner sees only a **counter** of pending cards (no content, no dates). | you, DEC-007 |
| REQ-F-CARD-07 | A card is editable/deletable only while scheduled; **once published it is read-only**. | DEC-024 |
| REQ-F-CARD-08 | Create/edit offline (see SYNC). | you |

## MEM — Interactive memory album (2 screens)

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-MEM-01 | **Gallery:** infinite auto-scroll; advances photo by photo within a date, then jumps to the next date; single-photo dates show for the same time and move on. | you, DEC-008 |
| REQ-F-MEM-02 | Auto-scroll pauses on interaction and resumes after **~8 s** idle. | DEC-008 |
| REQ-F-MEM-03 | Tapping a photo toggles its story (description). | you |
| REQ-F-MEM-04 | Linked song: file or direct URL plays **in the background**; Spotify/YouTube shows a **visible embedded player** (no background audio). | you, DEC-010 |
| REQ-F-MEM-05 | Background audio starts only after a first user gesture (browser autoplay policy). | DEC-010 |
| REQ-F-MEM-06 | Star on the photo corner toggles a personal favorite. | you, DEC-011 |
| REQ-F-MEM-07 | **Management screen:** list of both partners' memories; edit/delete **own only**; the form opens from the list. | you |
| REQ-F-MEM-08 | Form: media (photo/video) by external URL or upload; description; date; song (optional, max 1); place (optional: existing, or new with a pin). | you, DEC-012 |
| REQ-F-MEM-09 | Uploads go to object storage and are cached on the device for offline viewing. | DEC-009 |
| REQ-F-MEM-10 | Quota and per-file limits (REQ-F-ADM-05) are validated before upload. | DEC-020 |

## SONG — Our songs

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-SONG-01 | Player screen plays the couple's song list. | you |
| REQ-F-SONG-02 | Each song has a personal note. | you |
| REQ-F-SONG-03 | While a song plays, a gallery of its related memories is shown. | you |
| REQ-F-SONG-04 | Star on the player corner toggles a personal favorite. | you |
| REQ-F-SONG-05 | Sources: upload, direct audio URL, Spotify, YouTube; the source model is extensible to new providers. | you, DEC-010 |
| REQ-F-SONG-06 | Form: source, description, related memories (optional, multiple). | you |
| REQ-F-SONG-07 | One consistent relation: memory → 0..1 song; song → 0..N memories. Editing from either side updates the same relation. | DEC-012 |
| REQ-F-SONG-08 | Edit/delete own only. | you |

## PLACE — Map of places

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-PLACE-01 | **Place** is a first-class entity (name, coordinates, description); one place groups N memories. | DEC-012 |
| REQ-F-PLACE-02 | Module home = dashboard: OpenStreetMap map + list of places with memory counts. | you, DEC-013 |
| REQ-F-PLACE-03 | Clicking a place opens its detail: memories with photos, story, date, song. | you |
| REQ-F-PLACE-04 | Star on the viewer favorites the **place**. | you |
| REQ-F-PLACE-05 | Edit/delete own only; deleting a place unlinks (does not delete) its memories. | — (needs freeze) |

## NOTE — Love diary

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-NOTE-01 | Write notes (feelings, special moments). | you |
| REQ-F-NOTE-02 | Visibility: `shared` (default) or `only_me`. | DEC-014 |
| REQ-F-NOTE-03 | List shows author and created_at; click opens the reader. | you |
| REQ-F-NOTE-04 | Star toggles a personal favorite (appears in FAV as a note). | DEC-025 |
| REQ-F-NOTE-05 | Edit/delete own only. | you |

## FAV — My favorites

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-FAV-01 | Personal unified list: memories (grouped by date), songs, places, notes. | you, DEC-011 |
| REQ-F-FAV-02 | Filter by type. | — (needs freeze) |
| REQ-F-FAV-03 | Click opens the item's detail in its module. | you |
| REQ-F-FAV-04 | If an item is deleted or becomes `only_me`, it disappears from the partner's favorites. | DEC-014 |

## ACC — My account

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-ACC-01 | Change profile photo. | you |
| REQ-F-ACC-02 | See name, birthday, partner. **Name is editable**; birth date is corrected only by an admin. | you, DEC-025 |
| REQ-F-ACC-03 | Change password (requires current one; revokes other sessions). | you |
| REQ-F-ACC-04 | Choose UI language (Spanish / English). | DEC-021 |
| REQ-F-ACC-05 | Notification preferences. | DEC-026 |
| REQ-F-ACC-06 | Log out. | you |

## NOTIF — Notifications

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-NOTIF-01 | Push "your card for today is ready 🍒" when a card is published for the user. | DEC-026 |
| REQ-F-NOTIF-02 | Reminders for anniversary (from relationship start) and the partner's birthday. | DEC-026 |
| REQ-F-NOTIF-03 | Mobile: Expo Notifications; web: Web Push with explicit user permission. | DEC-022, DEC-026 |
| REQ-F-NOTIF-04 | Each notification type can be toggled per user. | DEC-026 |
| REQ-F-NOTIF-05 | Push payloads never include private content. | DEC-015 |

## SYNC — Offline and sync

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-SYNC-01 | Read offline whatever was already downloaded. | you |
| REQ-F-SYNC-02 | Create/edit/delete offline → persistent local queue → replay when online. | you |
| REQ-F-SYNC-03 | Entity IDs are generated on the client (UUIDv7). | — (needs freeze) |
| REQ-F-SYNC-04 | Every queued mutation carries a `client_op_id`; the server stores it so replays are idempotent. | v0 REQ-SYNC-001 |
| REQ-F-SYNC-05 | Conflict rule: last-write-wins per record using a server `version` (only authors write); deletes travel as tombstones (soft delete). | — (needs freeze) |
| REQ-F-SYNC-06 | Incremental pull by cursor (`since`), never full downloads. | — (needs freeze) |
| REQ-F-SYNC-07 | Media uploads use a separate queue with retries and quota validation. | DEC-009, DEC-020 |
| REQ-F-SYNC-08 | Visible state: offline, pending, synced. | — (needs freeze) |
| REQ-F-SYNC-09 | Local store: mobile = SQLite; web = IndexedDB. One sync engine in `cherrylove-core`. | you, DEC-003 |
| REQ-F-SYNC-10 | Server rules win (published card lock, quota, permissions); the client shows the rejection clearly. | — (needs freeze) |

## ADM — Administration (customized Django Admin + custom views)

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-ADM-01 | CRUD of couples and users under REQ-F-AUTH-05. | you |
| REQ-F-ADM-02 | Invites: send, resend, revoke, see status. | you, DEC-017 |
| REQ-F-ADM-03 | **Archive couple:** accounts disabled, data retained **90 days** (export offered), then purged by the scheduler. | DEC-019 |
| REQ-F-ADM-04 | Staff see **metadata only** (counts, dates, sizes), never content. | DEC-015 |
| REQ-F-ADM-05 | Configurable quotas. Defaults: 5 GB per couple; 20 MB photo, 200 MB video, 30 MB audio per file. | DEC-020 |
| REQ-F-ADM-06 | Dashboard: active couples, logins, items created and visited, per week and month. | you |
| REQ-F-ADM-07 | Log viewer filtered by type, date, couple, user. | you |
| REQ-F-ADM-08 | Staff roles `superadmin`, `admin`, `support` (read-only) per `roles-actuators.md`. | DEC-016 |
| REQ-F-ADM-09 | Admin can correct a user's birth date. | DEC-025 |
| REQ-F-ADM-10 | The API exposes CRUD for every couple-app module. | you |

## LOG — Audit

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-LOG-01 | Log every action: create, update, delete, view, login, logout, auth failure, admin action, sync. | you |
| REQ-F-LOG-02 | Each event stores actor, couple, action, entity type + id, timestamp, IP/device, `request_id`, and a field diff **without sensitive content**. | you, DEC-015 |
| REQ-F-LOG-03 | Append-only. | — (needs freeze) |
| REQ-F-LOG-04 | Partitioned by month with configurable retention. | — (needs freeze) |

## Backlog (not v1; seams only)

Export "our book" (PDF/ZIP), dark mode "Black Cherry", OpenTelemetry, feature flags, extra couple modules,
microservices/extra APIs. (DEC-027) The load-data generator moved to v1 (DEC-034, REQ-N-PERF-06).

## Next step

`nfr.md`, then `roles-actuators.md`, then reply `freeze` or patches by ID.
