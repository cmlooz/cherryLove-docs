---
id: GLO-001
status: in_review
approved_by: null
approved_at: null
---

# Glossary

Code, IDs, and docs use these English terms. User-visible copy is localized (es/en) and may use CherryLove names after the UX session.

| Term | Meaning |
|------|---------|
| CherryLove | Product name. Brand and copy stay warm, specific, couple-centered. |
| Couple | Tenant. At most two users. Exactly one couple per user; no switching. Has a timezone and a relationship start date. |
| Partner | User who belongs to a couple. Both partners have equal rights. |
| Invitee | Email holding a valid invite (72 h) that has not finished onboarding. |
| Staff | Platform roles: `support` (read-only), `admin`, `superadmin`. Never see content. |
| Couple space | All content visible to both partners of one couple. |
| Card | Daily letter with author, timestamps, and `publish_at`. Read-only once published. |
| Memory | Album item (photo or video) with description, date, optional song, optional place. |
| Song | Playlist item with a media source and a personal note; relates to N memories. |
| Place | First-class map point (name, lat/lng, description) grouping N memories. |
| Note | Diary entry by one partner; visibility `shared` or `only_me`. |
| Favorite | Personal star on a Memory, Song, Place, or Note. Home timeline shows both partners' favorites. |
| MediaRef | `{ kind, uri, provider }` pointing at object storage, https URL, Spotify, or YouTube. |
| Quota | Storage limit per couple and per file type, configurable by admin. |
| Archive | Admin action ending a couple: accounts disabled, data kept 90 days, then purged. |
| Contract | OpenAPI spec; the only coupling between API and UIs. |
| Core | `cherrylove-core`: shared TS package (generated types, API client, validation, sync engine, tokens, i18n). |
| Port | Domain-facing interface (repository, mailer, clock, storage). Adapters implement ports. |
| Actuator | Who or what initiates an action (partner, staff, scheduler, sync engine). |
| Action | Named operation with preconditions, permission, payload, effect, and log type. |
| Client op id | Idempotency key of a local mutation replayed during sync. |
| Tombstone | Soft-deleted record propagated by sync. |
| Decision (DEC) | Approved answer recorded in `00-governance/decision-log.md`. |
| Evidence | Stored proof that a phase requirement was tested. |
