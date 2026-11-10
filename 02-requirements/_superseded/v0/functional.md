<!-- superseded on 2026-11-09 by the current file of the same name; kept for history (GOV-001) -->
# Functional requirements

Status: **draft** (not approved)

Every ID is pending your accept / reject / change. Senior recommendations that are **not** in the original proposal are marked `REC` and must be accepted before they count.

## Quick path

1. Read IDs.
2. Answer `open-questions.md` where an ID depends on a gap.
3. Reply with a list: `REQ-F-010 accept`, `REQ-F-041 change to ...`, etc.

## Cross-cutting

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-001 | Active couple space has at most two users. A user has at most one couple. | you |
| REQ-F-002 | Web and mobile expose the same couple capabilities. | you |
| REQ-F-003 | Couple apps work offline and sync when connected. | you |
| REQ-F-004 | Home dashboard shows a timeline of favorites and `{user1.name} y {user2.name}` plus days-together. | you |
| REQ-F-005 | Modules are list+management with detail on click, except where a module defines a dedicated today/player/map view. | you |
| REQ-F-006 | UIs and APIs couple only through an approved contract. A new UI or a new API that honors the contract must work. | you |
| REQ-F-007 | Visual and verbal design follow CherryLove, not a generic template. | you |
| REQ-F-008 | Future modules can be added without rewriting the kernel (couple, identity, sync, media refs, favorites). | you (extension) |

## Daily cards

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-010 | Screen A shows today's card: content, author, created_at. | you |
| REQ-F-011 | Screen B lists cards the current user created, and allows create/edit plus scheduling via publish_at. | you |
| REQ-F-012 | Both partners see the same card content for the same couple civil date. | you |
| REQ-F-013 | Create/edit while offline is stored locally and synced later. | you |
| REQ-F-014 | REC: Today's card is the card whose publish_at date equals today in the couple timezone. If several match, newest wins. If none, empty state. | rec, blocked on Q1 and Q8 |

## Album

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-020 | Gallery uses infinite auto-scroll until the user interacts with a photo. | you |
| REQ-F-021 | Tap a photo toggles its story/description. | you |
| REQ-F-022 | Viewing a photo with a linked song plays that song in the background. At most one song per memory. | you |
| REQ-F-023 | Star on the photo adds/removes the memory from favorites. | you |
| REQ-F-024 | Create form: MediaRef for photo/video (local upload with local reference, or remote URL preferred), description, photo date, optional song selector (one), optional location pin. | you |
| REQ-F-025 | User-created memories appear in a list that expands the create/edit form. User can edit and delete those they created. | you |

## Songs

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-030 | Player screen plays the couple's important songs list. | you |
| REQ-F-031 | Each song may have a personal note, a gallery of related memories while playing, and a star to favorite the song. | you |
| REQ-F-032 | Create form: MediaRef (local file reference, remote URL preferred, Spotify/YouTube/etc.), description, optional multi-select of memories. | you |

## Map

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-040 | Module dashboard summarizes locations; click opens detail (photos, story, date, song). | you |
| REQ-F-041 | Star on the viewer favorites the **place**. | rec, blocked on Q2 (proposal text said song) |
| REQ-F-042 | REC: Place is a first-class entity, not only a pin on a memory. | rec |

## Diary

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-050 | Note list shows author and created_at; click opens the reader. | you |
| REQ-F-051 | Partners can write feelings and special moments and read them in the app. | you |
| REQ-F-052 | REC: Notes are shared in the couple space (not private to author). | rec, blocked on Q5 |

## Favorites and account

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-060 | Favorites list includes memories, songs, places, notes. Click opens that item's detail. | you |
| REQ-F-061 | Memories on the same date or location are shown grouped. Grouping key blocked on Q6. | you |
| REQ-F-070 | Account: change profile photo; see name, birthday, partner; change password. | you |

## Auth, couple, invites

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-080 | Inviter sends an email invitation. Invitee fills name, birth date, relationship start date. | you |
| REQ-F-081 | REC: Invite tokens are single-use and expire. | rec |
| REQ-F-082 | REC: If the invitee already belongs to a couple, the invite is rejected. No couple-switch in v1. | rec, blocked on Q7 |
| REQ-F-083 | REC: Days together count civil days from relationship_start_date in couple timezone. | rec, blocked on Q8 |

## Ownership (blocked on Q5)

| ID | Statement | Source |
|----|-----------|--------|
| REQ-F-090 | REC: Author can edit/delete own content. Partner can read published couple-space content. | rec |

## Admin / API capabilities (functional view)

| ID | Statement | Source |
|----|-----------|--------|
| REQ-ADM-001 | API offers CRUD for all couple-app modules and persists them. | you |
| REQ-ADM-002 | Admin manages couple-user relations under REQ-F-001. | you |
| REQ-ADM-003 | Admin dashboard: active couples, logins, items created and visited, by week and month. | you |
| REQ-ADM-004 | Action logs with filter by type. | you |
| REQ-ADM-005 | Django admin/API does not import React/RN. UIs do not import Django models. | you |

## Checklist

- [ ] You returned accept/reject/change per ID (or per section)
- [ ] REC rows you want are explicitly accepted

## Next step

`nfr.md` then `open-questions.md`
