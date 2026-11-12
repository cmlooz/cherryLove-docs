---
id: ADR-006
status: approved
approved_by: Camilo
approved_at: 2026-11-11
---

# ADR-006 — Offline sync model

Implements: REQ-F-SYNC-01…10, REQ-F-CARD-08, REQ-F-FAV-04, REQ-F-NOTE-02, REQ-N-PERF-02, DEC-014, DEC-024.
Builds on: ADR-001 (modules, transactional side effects), ADR-003 (contract conventions), ADR-005 (Actor,
invisible = not found).

## Context

The frozen requirements already fix the ingredients: client-generated UUIDv7 IDs, `client_op_id` for idempotent
replay, last-write-wins with a server `version`, tombstones, cursor pull, one engine in `cherrylove-core`,
server rules win. What is open is **how they fit together** so that no change is ever silently missed.

Key fact that simplifies everything: **only the author writes a record** (roles-actuators). Two people never
edit the same row; conflicts can only come from **one author on two devices**.

## Decision drivers

1. **No missed changes**: a device that pulls with cursor *N* must eventually see every change after *N*.
2. Replays are safe (network drops after the server applied but before the client heard back).
3. The partner's private items (`only_me` notes, scheduled cards) never reach the partner's device.
4. Server rules win and the user sees why (REQ-F-SYNC-10).
5. Module borders from ADR-001 stay intact: `sync` does not read other modules' tables.

## The hard part: choosing the cursor

| Option | Problem |
|--------|---------|
| O1 — `updated_at` timestamp | Clock skew, equal timestamps, and **commit order ≠ timestamp order**: a transaction stamped 10:00:01 that commits after one stamped 10:00:02 is skipped by a client that already pulled past 10:00:02. ❌ |
| O2 — Global PostgreSQL sequence | Same commit-order gap: seq 10 can commit after seq 11 is read. ❌ |
| O3 — **Per-couple counter incremented under a row lock** ⭐ | Each write does `SELECT … FOR UPDATE` on the couple's `sync_state` row and takes `seq + 1`. Writes **of one couple** are serialized, so commit order = seq order. Two users per couple → contention is negligible. ✅ |

## Decision

### 1. Server data

| Table (module `sync`) | Content | Key / index |
|-----------------------|---------|-------------|
| `sync_state` | `couple_id`, `last_seq`, `purged_through_seq` | PK `couple_id` |
| `sync_entity_index` | `couple_id`, `entity_type`, `entity_id`, `seq`, `author_id` | unique `(couple_id, entity_type, entity_id)`; index `(couple_id, seq)` |
| `sync_client_op` | `couple_id`, `client_op_id`, stored result, `created_at` | unique `(couple_id, client_op_id)`; kept 30 days |

`sync_entity_index` holds **one row per entity** (upserted on every change), not a growing log.

Every syncable entity row in its own module has: `id` (UUIDv7), `version` (int), `deleted_at` (tombstone).

**Recording a change is transactional**, like audit (ADR-001 §4): each use case calls the `ChangeRecorder` port
inside its unit of work. That port bumps `sync_state.last_seq` and upserts `sync_entity_index`. No
after-commit event is used here; a lost event would mean a missed change (driver 1).

Time-based changes also bump `seq`: the job that publishes cards at `publish_at` records a change, so the
partner's device learns about the newly visible card.

### 2. Push — `POST /api/v1/sync/push`

Request: an ordered batch (≤ 100) of operations:

```json
{ "client_op_id": "uuidv7", "entity_type": "memory", "entity_id": "uuidv7",
  "op": "upsert | delete", "base_version": 3, "payload": { "...full record..." } }
```

Per operation, in order:

1. If `client_op_id` exists in `sync_client_op` → return the stored result (idempotent replay, REQ-F-SYNC-04).
2. Otherwise run the **same use case** as the REST endpoint, with the same `Actor` and policies (ADR-005 §6).
3. Store and return one result per op:

| Result | Meaning | Client action |
|--------|---------|---------------|
| `applied` + new `version` | Write accepted | mark synced |
| `rejected` + Problem Details | Server rule failed (published card, quota, permission, validation) | restore server truth, show reason |
| `gone` | Entity was deleted (tombstone) | drop local copy |

A rejected op never aborts the batch; later ops **on the same entity** in that batch are rejected too
(they depend on it).

### 3. Conflict rule

- **Whole-record last-write-wins by server arrival order.** Client clocks are never trusted.
- `base_version` is not a lock: if it is older than the server's `version`, the write still applies but the
  result and the audit event carry `overwrote_newer: true` (diagnostics, and a hint in the UI).
- **Deletes win**: once tombstoned, an entity cannot be resurrected by a late update → `gone`.
- Field-level merge is rejected: only one author per record makes it cost without benefit.

### 4. Pull — `GET /api/v1/sync/pull?cursor=…&limit=500`

1. Read `sync_entity_index` where `couple_id = actor.couple_id AND seq > cursor ORDER BY seq LIMIT n`.
2. Group IDs by type and load them in **batches** through each module's `public.py` facade (driver 5).
3. Filter by what **this actor** may see (ADR-005):

| Entity state for this actor | Pull returns |
|-----------------------------|--------------|
| Visible | `{ "op": "upsert", "record": {...} }` |
| Tombstoned | `{ "op": "delete", "id": … }` |
| Exists but not visible (partner's `only_me` note, partner's scheduled card) | `{ "op": "delete", "id": … }` — the client removes any copy; existence is not revealed beyond what the client already had |

4. Response: `{ "changes": [...], "next_cursor": "…" | null, "pending_card_count": n }` — the counter of the
   partner's scheduled cards travels here (REQ-F-CARD-06), never their content.

The cursor is opaque (encoded `seq`). `cursor = null` means "from the beginning" (first sync, same mechanism).
If `cursor < purged_through_seq` → **410** `cursor_expired`: the client wipes its local store and resyncs.

### 5. Tombstone retention

A daily job hard-deletes tombstones older than **90 days** and advances `purged_through_seq`. A device offline
for more than 90 days does a full resync; nothing else breaks.

### 6. Client engine (`cherrylove-core/src/sync/`)

| Part | Rule |
|------|------|
| Local store | Entity tables + `outbox` + `meta(cursor)` behind the `LocalStore` port (SQLite / IndexedDB, REQ-F-SYNC-09) |
| Local write | Apply optimistically, append to `outbox` with a new `client_op_id`, record state `pending` |
| Cycle | **Push outbox first, then pull** until `next_cursor = null` |
| Triggers | App foreground, connectivity regained, local write (debounced 2 s), every 60 s while in foreground |
| Retries | Network / 5xx / 429: exponential backoff with jitter (cap 5 min). Per-op 4xx results are final |
| Media | An op whose record references a not-yet-uploaded file waits in the outbox until its upload (separate queue, REQ-F-SYNC-07) succeeds — upload flow in ADR-007 |
| Visible state | Global `offline / pending / synced`; per record `pending / synced / rejected` (REQ-F-SYNC-08) |

### 7. What syncs

Cards (own + partner's published), memories, songs, places, notes (shared + own `only_me`), favorites of
**both** partners (REQ-F-HOME-02), and the couple header (names, relationship start). **Not synced offline:**
login, password, account settings, notification preferences, anything staff-side.

### 8. IDs

UUIDv7 is generated by the client (`cherrylove-core`) and validated by the server (version nibble = 7,
embedded timestamp not more than 24 h in the future). Python side: stdlib `uuid.uuid7()` if the pinned Python
has it (ADR-016), otherwise a small helper — no new dependency either way.

## Consequences

- ✅ No missed changes by construction: per-couple serialized `seq` + transactional recording.
- ✅ Replays are free; offline creates use final IDs, so relations work before the server sees them.
- ✅ Privacy rules apply to pull exactly as to REST — one set of policies.
- ✅ `sync` reads other modules only through facades.
- ❌ Every write in every module takes the couple's `sync_state` row lock (short; per couple only).
- ❌ LWW can silently overwrite an edit made on another device by the same author; mitigated by
  `overwrote_newer` and the fact that it is always the same person.
- ⚠️ The pull path must load in batches per type; a per-row facade call would be an N+1 (REQ-N-PERF-02).

## Next step

Camilo approves, rejects, or patches by section number. Then ADR-007 (media storage).
