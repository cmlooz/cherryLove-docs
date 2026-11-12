---
id: ADR-011
status: approved
approved_by: Camilo
approved_at: 2026-11-12
---

# ADR-011 — Caching

Implements: REQ-N-PERF-04, REQ-N-PERF-01, REQ-F-CARD-01/02/04, REQ-F-HOME-01/02, REQ-N-SEC-05, REQ-N-SEC-06.
Builds on: ADR-004 §4 (throttles on Redis), ADR-005 §1 (Actor cache), ADR-006 §1 (per-couple `seq`),
ADR-007 §5 (immutable media), ADR-009 (no plaintext outside the DB process).

## Context

REQ-N-PERF-04 asks for a Redis cache for today's card and the home dashboard. Earlier ADRs already put more in
Redis (actor membership, throttle counters). The two classic caching failures are **stale data** and **leaked
data**; here the second matters more, because card and note bodies are encrypted at rest (ADR-009) and a cache
must not quietly store them in plain text next door.

## Decision drivers

1. Never serve stale content after a change.
2. No decrypted content (bodies, descriptions) ever stored in Redis.
3. Invalidation simple enough to be correct without thinking.
4. A Redis outage degrades speed, not correctness.

## Options considered

| Topic | Options | Choice |
|-------|---------|--------|
| Invalidation | Delete keys on every write · TTL only · **version in the key** ⭐ | Key embeds the couple's sync `seq` (ADR-006): any change bumps it, old keys are simply never read again |
| What to store | Full rendered responses · **selections and references only** ⭐ | Bodies stay encrypted in PostgreSQL; the cache holds the expensive *decision* (which IDs, which order) |
| Client data cache | HTTP/query cache library · **the sync local store** ⭐ | Synced entities are read from the local store (ADR-006); no second client cache |

## Decision

### 1. Server cache (Redis, cache-aside behind a `Cache` port)

| Key | Value | Why it is expensive | Invalidated by |
|-----|-------|---------------------|----------------|
| `actor:{user_id}` | `couple_id`, active, archived | Every request resolves it (ADR-005) | TTL 10 min (= access token) + explicit delete on archive/disable |
| `cards:today:{couple_id}:{civil_date}:{seq}` | Ordered card IDs, or the drawn random past card ID | Random draw over all past cards (REQ-F-CARD-02) | New `seq` (publish job and every write bump it), new civil date |
| `home:{couple_id}:{seq}` | Favorites timeline refs: `(item_type, item_id, starred_by, date)` top N | Cross-module composition (ADR-001 §3) | New `seq` |
| throttle counters | DRF throttling (ADR-004 §4) | — | Window expiry |

Reading `seq` is one primary-key lookup on `sync_state` — cheaper than any invalidation bug.
Hydration (bodies, titles) happens after the cache hit, in **batches** through module facades, decrypting in
process (ADR-009). Every key also gets a **24 h TTL** as a safety net against leaks of memory, not of data.

`civil_date` is computed in the couple timezone (REQ-F-CARD-04), so "today" rolls over at the couple's midnight.

### 2. Redis outage

| Use | On Redis failure |
|-----|------------------|
| Cache keys | **Fail open**: log, compute from PostgreSQL. Slower, still correct. |
| Throttles on login, recovery, invite accept | **Fail closed**: 503 `service_unavailable`. Brute force must not become possible because Redis is down. |
| Other throttles | Fail open. |

### 3. HTTP caching

| Response | Header | Why |
|----------|--------|-----|
| Authenticated API responses | `Cache-Control: no-store` | Private data; no shared or browser cache |
| Media objects (set at `ready`, ADR-007) | `Cache-Control: private, max-age=31536000, immutable` | Content behind a `media_id` never changes; the browser keeps it even after the signed URL expires |
| Web static assets (hashed file names) | `public, max-age=31536000, immutable` | Standard for built bundles |
| `index.html` | `no-cache` | Picks up new deployments |

### 4. Client

Synced entities (cards, memories, songs, places, notes, favorites) are read from the local store; it **is** the
client cache (ADR-006). Media bytes live in the device media cache keyed by `media_id + variant` (ADR-007 §5).
Non-synced reads (account settings) are fetched on screen open, no client cache.

### 5. Dependency

`redis` (redis-py) — required by Django's built-in `django.core.cache.backends.redis.RedisCache`; no
`django-redis`. Version pinned in ADR-016.

### 6. Measured, not assumed

The load test (REQ-N-PERF-06) runs the today and home endpoints with cache on and off. If a cache does not move
p95, it is removed: a cache is a liability that must earn its place.

## Consequences

- ✅ Staleness is impossible by construction: a changed couple has a new `seq`, so a new key.
- ✅ Redis never holds plaintext content; a Redis dump reveals IDs and dates only.
- ✅ One client cache (the local store), not two that can disagree.
- ❌ Every cached read pays one `seq` lookup and a batched hydration — cheap, but not free.
- ❌ Old versioned keys linger until their TTL (memory, not correctness).
- ⚠️ Fail-closed auth throttles make login depend on Redis availability.

## Next step

Camilo approves, rejects, or patches by section number. Then ADR-012 (i18n).
