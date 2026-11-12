---
id: REQ-DOC-003
status: approved
approved_by: Camilo
approved_at: 2026-11-11
supersedes: roles-actuators.md draft v0
---

# Roles, actuators, actions

Rule of the system: **each partner sees everything shared in their couple, but edits and deletes only their own.
Staff never see content. Nothing crosses couples. Everything is logged.**

## Roles

| Role | Who | Can |
|------|-----|-----|
| `invitee` | Email with a valid invite token, onboarding not finished | Open invite, submit onboarding |
| `partner` | User in a couple (active, or waiting for the second partner) | Use couple modules, sync, manage own account |
| `support` | Staff, read-only | View metadata, metrics, logs |
| `admin` | Staff | Everything `support` can + couples, invites, archive, quotas, birth-date correction |
| `superadmin` | Staff | Everything `admin` can + manage staff and system settings |
| `system` | Jobs | Publish cards, expire invites, send reminders, roll up metrics, purge archived couples |

Partners are equals. There is no "owner" partner. There is **no self-service couple creation** (DEC-028):
the v0 roles `unpaired_user` and `inviter` are removed. Staff roles use Django's native groups and permissions (DEC-016).

## Actuators and actions

| Actuator | Actions |
|----------|---------|
| Invitee | open invite, submit name + birth date + password + relationship start (skipped if already set) |
| Partner | login, logout, recover password, change password/avatar/name/language/notification prefs, CRUD own content, favorite/unfavorite, read couple content, sync push/pull, play song, open detail |
| Support | list couples (metadata), view dashboards, filter logs |
| Admin | create couple with 2 emails, send/resend/revoke invites, archive couple, set quotas, correct birth date |
| Superadmin | manage staff accounts and groups, system settings |
| Scheduler | publish cards by `publish_at`, expire invites (72 h), anniversary/birthday reminders, weekly/monthly rollups, purge archived couples after 90 days |
| Sync engine | queue local writes with `client_op_id`, replay online, apply server truth, surface rejections |
| Mailer | send invites, password recovery |
| Push service | card-published and reminder notifications (no private content) |
| Media storage | store and serve media through signed URLs |
| Music providers | Spotify / YouTube embeds |
| Map provider | OpenStreetMap tiles |

## Permission matrix

| Action | Invitee | Partner (own) | Partner (partner's) | Support | Admin | Superadmin |
|--------|---------|---------------|---------------------|---------|-------|------------|
| Complete onboarding | ✅ | — | — | — | — | — |
| Read shared content | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Read `only_me` note | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Read scheduled card | ❌ | ✅ | ❌ (counter only) | ❌ | ❌ | ❌ |
| Create content | ❌ | ✅ | — | ❌ | ❌ | ❌ |
| Update / delete content | ❌ | ✅ (cards only while scheduled) | ❌ | ❌ | ❌ | ❌ |
| Favorite (personal) | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Metadata, metrics, logs | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| Create couple / invite / archive | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Quotas, birth-date correction | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Manage staff | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

## API permission chain

`IsAuthenticated` → `IsCouplePartner` (request) → `IsSameCouple` (object, read) → `IsAuthor` (object, write).
Out-of-couple resources return 404 (REQ-N-SEC-02).

## Next step

After freeze, each action gets preconditions, payload, validations, effect, and `action_type` log entry (session C).
