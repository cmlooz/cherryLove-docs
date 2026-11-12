---
id: ADR-009
status: approved
approved_by: Camilo
approved_at: 2026-11-11
---

# ADR-009 — Encryption at rest

Implements: REQ-N-SEC-05, REQ-N-SEC-08, REQ-F-ADM-03, DEC-015, DEC-019.
Builds on: ADR-001 (domain sees plain values; ORM is an adapter), ADR-007 §7 (bucket SSE), ADR-008 (no content in logs).

## Context

REQ-N-SEC-05: card and note bodies are encrypted at rest. The question is *against whom*. Disk encryption
protects a stolen disk, but anyone with SQL access, a database dump, or a backup still reads every love letter in
plain text — including a curious superadmin, which REQ-N-SEC-08 forbids.

DEC-019 adds a second need: after an archived couple's 90 days, their data must really be gone, **including
from backups** we cannot rewrite.

## Decision drivers

1. Bodies unreadable from SQL, dumps and backups.
2. Purge must make data unrecoverable everywhere, backups included.
3. Domain and use cases stay crypto-free (ADR-001).
4. Proven primitives only; one small dependency at most.

## Options considered

| Option | Verdict |
|--------|---------|
| O1 — Disk / volume encryption only | ❌ Fails driver 1: SQL, dumps, backups are plain text. Kept as a baseline in production, not as the answer. |
| O2 — `pgcrypto` in SQL | ❌ The key travels inside SQL statements (logs, `pg_stat_statements`). |
| O3 — Field-level encryption libraries for Django (encrypted model fields) | ❌ Mostly unmaintained; one global key; no per-couple shredding. |
| O4 — **Application-level envelope encryption with a per-couple key** ⭐ | ✅ Meets all drivers; uses `cryptography` (AES-GCM) behind a port. |

## Decision

**O4.**

### 1. Keys (envelope encryption)

| Key | What | Where |
|-----|------|-------|
| **KEK** (key-encryption key) | 256-bit master key; never stored in the DB | Env var locally (Docker Compose `.env`, never committed); secret manager in production (production ADR) |
| **DEK** (data key) | 256-bit, **one per couple**, generated at couple creation | `couple_data_key` table, stored **wrapped** (encrypted with the KEK) |

Decrypted DEKs are cached in process memory for 5 minutes (LRU), never in Redis.

### 2. Cipher

- **AES-256-GCM** (`cryptography.hazmat.primitives.ciphers.aead.AESGCM`), 96-bit random nonce per value.
- **Associated data (AAD)** = `couple_id | entity_type | entity_id | field`. A ciphertext copied to another row,
  field or couple fails to decrypt — swap attacks are impossible.
- Stored as `bytea`: `version (1 byte) | key_id | nonce | ciphertext+tag`. `version` allows a future algorithm change.

### 3. Where it happens

`Cipher` port in `shared/application`, implementation in `shared/infrastructure/crypto/`. Encryption and
decryption live **only in repository mappers** (infrastructure). The domain entity's `body` is a plain `str`;
query services decrypt the DTO fields they return.

### 4. Encrypted fields

| Field | Status |
|-------|--------|
| `card.body` | **Required** by REQ-N-SEC-05 |
| `note.body` | **Required** by REQ-N-SEC-05 |
| `memory.description`, `song.note`, `place.description` | **Recommended extension** — equally intimate. Approving this ADR approves the extension. |

Not encrypted: names, dates, coordinates, IDs, visibility, `publish_at` — needed for filters, sorting, the map and
the scheduler. Consequence: encrypted fields cannot be searched or sorted in SQL (no frozen requirement asks for it).

### 5. Crypto-shredding on purge (DEC-019)

The purge job (REQ-F-ADM-03) deletes the couple's rows, media prefix **and its DEK**. Old backups still contain the
ciphertext, but with the DEK gone it is unrecoverable. The export offered before purge decrypts with the DEK
while it still exists.

### 6. Rotation

| Rotation | How | Cost |
|----------|-----|------|
| KEK | Re-wrap every DEK with the new KEK (one row per couple) | Seconds |
| DEK | New `key_id` for the couple; values re-encrypted lazily on next write, or by a background job | Optional |

### 7. Media

Media bytes use storage server-side encryption (ADR-007 §7). Locally, MinIO SSE needs a KMS setup → optional in
Docker Compose, **mandatory in production**. Media is not crypto-shredded; purge deletes the couple's prefix.
Client-side media encryption is out of scope for v1 (cost, thumbnails, signed URLs).

### 8. Dependency

`cryptography` (PyCA) — the de-facto Python crypto library, maintained by the Python Cryptographic Authority.
It is a new direct dependency (ADR-004 uses `HS256`, which does not need it). Version pinned in ADR-016.

## Consequences

- ✅ A superadmin with SQL access, a leaked dump or an old backup reveals no bodies.
- ✅ Purge is real, backups included, without rewriting them.
- ✅ Domain code never sees crypto.
- ❌ No SQL search or sort on encrypted fields.
- ❌ Losing the KEK loses every couple's bodies → KEK backup procedure is part of the production ADR.
- ⚠️ Load test (REQ-N-PERF-06) must include decryption on list endpoints; AES-GCM is microseconds per field, the
  DEK lookup is cached.

## Next step

Camilo approves, rejects, or patches by section number. Then ADR-010 (error handling, RFC 9457).
