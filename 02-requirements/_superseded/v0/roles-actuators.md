<!-- superseded on 2026-11-09 by the current file of the same name; kept for history (GOV-001) -->
---
status: draft
approved_by: null
approved_at: null
---

# Roles, actuators, actions

Recommended until you change them. Not approved.

## Roles

| Role | Who | Can |
|------|-----|-----|
| `guest_invitee` | Email + invite token, no finished account | Open invite form, submit onboarding |
| `unpaired_user` | Account, no couple | Create couple + invite, or accept invite |
| `partner` | User in an active couple | Use couple modules, sync, manage own account |
| `platform_admin` | Django operator | Couples, access, metrics, logs |
| `system` | Jobs | Publish dated cards, rollup stats |

Partners are equals. The inviter is not an owner role.

## Actuators and actions

| Actuator | Actions |
|----------|---------|
| Partner | login, logout, change password, change avatar, CRUD own content, favorite/unfavorite, read couple-space content, sync push/pull, play song, open module detail |
| Invitee | open invite, submit name + birth date + relationship start |
| Inviter | create couple, send invite email |
| Platform admin | list couples, inspect metrics, filter logs, disable access, technical resource CRUD |
| Scheduler | publish cards by `publish_at`, weekly/monthly stats |
| Sync engine | queue local writes, replay online, apply server truth, surface conflicts |
| Media provider | resolve MediaRef to view/play |
| Mailer | send invite |

## Permission sketch (pending Q-EDIT-001)

| Action | Rule (recommendation) |
|--------|------------------------|
| Read published couple content | Any `partner` of that couple |
| Create | `partner`; content is born in that couple space |
| Update / delete | Author only |
| Favorite | Either partner |
| Admin read of intimate body | Blocked unless audited break-glass (Q-ADM-001) |

## Next step

Approve roles. Then each action gets preconditions, payload, validations, effect, and `action_type` log — in a later pass after freeze, not now.
