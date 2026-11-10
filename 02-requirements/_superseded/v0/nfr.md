<!-- superseded on 2026-11-09 by the current file of the same name; kept for history (GOV-001) -->
# Non-functional requirements

status: `in_review`  
id: REQ-DOC-002

## Quality

| ID | Statement | v1 recommendation (not approved) |
|----|-----------|----------------------------------|
| REQ-N-010 | Layers must not know each other. Database engine is swappable. | Ports: repositories; engine selected in adapter settings. |
| REQ-N-020 | Fewest dependencies that still honor official docs of React, RN, Django. | Django + one API layer + PostgreSQL server; SQLite (or equivalent) on clients. |
| REQ-N-030 | Typed at boundaries. Validate before `try/catch`. | Serializers/schema on API; TypeScript on UIs. |
| REQ-N-040 | SOLID, readable, traceable actions. | One action = one use case + structured log. |
| REQ-N-050 | High security. | Authn + couple-scoped authz on every query. Never trust client `couple_id`. |
| REQ-N-060 | Fast lists under large record counts. | Index `(couple_id, occurred_at)`; no lazy N+1 on lists. |
| REQ-N-070 | Ready for extra modules later. | One Django app / one UI feature folder per bounded context. |
| REQ-N-080 | Official documentation first when teaching (notebooks later). | Each lesson cites the official page it exercises. |

## Sync

| ID | Statement | v1 recommendation |
|----|-----------|-------------------|
| REQ-SYNC-001 | Every local mutation carries `client_op_id` (idempotent replay). | UUID per op, stored on server |
| REQ-SYNC-002 | Two-partner write conflict has a defined rule. | Last-write-wins on server `updated_at` (Q7) |

## Security extras (recommended)

| ID | Statement |
|----|-----------|
| REQ-SEC-001 | Authorization question on every read/write: does this user belong to this couple? |
| REQ-SEC-002 | Platform admin sees metadata and counts by default. Intimate payloads require an explicit, audited reveal. |
| REQ-SEC-003 | Short-lived access tokens + refresh. Invite tokens are single-use and expire. |
| REQ-SEC-004 | Soft delete for memories, songs, notes, places, cards. |

## Media

| ID | Statement | v1 recommendation |
|----|-----------|-------------------|
| REQ-MED-001 | Persist a MediaRef, not a Django-hosted blob, in v1. | `{ kind, uri, provider }` |
| REQ-MED-002 | Local files remain available in the couple environment after upload/reference. | Cache locally; sync the reference, not necessarily the bytes, unless you later approve blob sync |
