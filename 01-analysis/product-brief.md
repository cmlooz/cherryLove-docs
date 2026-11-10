# Product brief

Status: **draft**

CherryLove is a private two-person space for daily letters, memories, songs, places, and notes, with a platform that invites couples and an admin that observes usage — built so the author learns architecture by doing it correctly.

## Quick path

- Audience: one couple per tenant, plus platform staff (support, admin, superadmin). The admin creates couples (DEC-028).
- Surfaces: React web, React Native mobile, Django API/admin.
- Constraint: offline work with later sync; layers must not know each other.
- Learning target: architecture decisions, not framework tourism.

## Problem

Couples want a shared, calm place for meaning (letters, photos, songs, maps, a diary). The builder wants a demanding but bounded system to learn how to start a greenfield: where files go, where validation lives, how UIs and APIs stay replaceable, how errors and security and sync are designed on purpose.

## Systems

| System | Actor | Capability |
|--------|-------|------------|
| Couple web app | Partner | Modules a–g, dashboard, offline + sync |
| Couple mobile app | Partner | Same capabilities as web |
| API | All authenticated actors | CRUD, authz, persistence, sync ingest |
| Admin | Platform admin | Couples, resources, invites, dashboards, logs |
| Docs | You + agent | Constitution |

## Modules (couple apps)

| Code | Module | Screens (from proposal) |
|------|--------|-------------------------|
| cards | Daily cards | Today view; create/edit/history of own cards |
| album | Interactive memory album | Infinite auto-scroll gallery; create/edit list+form |
| songs | Our songs player | Player; create/edit with MediaRef + related memories |
| map | Important places | Location dashboard; place detail |
| diary | Love notebook | Note list; reader |
| favorites | My favorites | Unified list, grouped memories; detail |
| account | My account | Profile photo, name, birthday, partner, password |
| home | Dashboard | Favorite timeline + names + days together |
| notifications | Push + reminders | Card published, anniversary, partner birthday (DEC-026) |

## CherryLove naming intent

The name is a constraint: cherry (fruit, red, blossom, sweetness with a pit/core) + love (care, pair, daily ritual).

Recommended (pending UX session, not assumed in UI copy yet):

| Area | Direction |
|------|-----------|
| Color | Cherry red, cream, seed/stone dark, blossom pink as accent |
| Motion | Album auto-scroll feels like falling petals until a touch stops it |
| Voice | Short, warm, specific. No "social feed" language |
| Domain names | Prefer glossary terms (Card, Memory, Song, Place, Note) in code |

## Out of scope for v0 freeze

- Extra couple modules not listed above (leave extension seams only)
- Hosting/production vendor lock-in
- Real Spotify/YouTube OAuth (visible embeds only, DEC-010)
- More than two people in a couple
- Switching couples

## Next step

`02-requirements/functional.md` and `open-questions.md`
