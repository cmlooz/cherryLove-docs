<!-- superseded on 2026-11-09 by the current file of the same name; kept for history (GOV-001) -->
# Glossary

status: `in_review`  
id: GLO-001

| Term | Meaning |
|------|---------|
| CherryLove | Product name. Brand and copy stay warm, specific, and couple-centered. |
| Couple | Tenant. At most two users. Exactly one couple per user in v1. |
| Partner | User who belongs to an active couple. Both partners have equal rights. |
| Couple space | All content visible to both partners of one couple. |
| Card | Daily letter/message with author, timestamps, and optional `publish_at`. |
| Memory | Album item (photo or video) with description, date, optional song, optional place. |
| Song | Shared playlist item identified by a media URI plus a personal note. |
| Place | Map point with associated memories, story, date, optional song. |
| Note | Diary entry authored by one partner, readable in the couple space. |
| Favorite | Starred Card, Memory, Song, Place, or Note. Dashboard timeline is favorites-first. |
| MediaRef | `{ kind, uri, provider }` pointing at local cache, https, Spotify, YouTube, or file. |
| Contract | OpenAPI (or equivalent) that is the only coupling between API and UIs. |
| Port | Domain-facing interface (repository, mailer, clock). Adapters implement ports. |
| Actuator | Who or what initiates an action (partner, admin, scheduler, sync engine). |
| Action | Named operation with preconditions, permission, payload, effect, and log type. |
| Client op id | Idempotency key for a local mutation replayed during sync. |
| Evidence | Stored proof that a phase requirement was tested (logs, screenshots, notebook tests). |

## Naming note (proposed, not approved)

Code and IDs stay English. User-visible copy can be localized later. CherryLove may name modules in the UI (for example "Today's cherry") only after the UX session. Domain types keep the table above.
