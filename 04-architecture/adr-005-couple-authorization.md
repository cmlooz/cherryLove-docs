---
id: ADR-005
status: approved
approved_by: Camilo
approved_at: 2026-11-11
---

# ADR-005 — Couple-scoped authorization

Implements: REQ-N-SEC-01, REQ-N-SEC-02, REQ-N-SEC-08, REQ-F-AUTH-05, REQ-F-NOTE-02, REQ-F-CARD-06/07,
REQ-F-ADM-04, REQ-F-ADM-08, DEC-014, DEC-015, DEC-016, roles-actuators (permission matrix + API chain).
Builds on: ADR-001 (layers, CQRS-lite), ADR-004 (no `couple_id` in the token).

## Context

The couple is the tenant: nothing may cross couples (REQ-N-SEC-01), and a resource from another couple must
look like it does not exist (404, REQ-N-SEC-02). The frozen API chain is
`IsAuthenticated → IsCouplePartner → IsSameCouple → IsAuthor`.

Problem: in ADR-001 views never load ORM objects — use cases do. DRF *object* permissions
(`has_object_permission`) would therefore never see the object. The chain must be mapped onto the layers,
not pasted into views.

## Decision drivers

1. **Fail closed**: forgetting a filter must produce an error, not a leak.
2. The tenant comes from the server, never from the request.
3. Ownership rules are business rules → domain, tested without HTTP.
4. Existence of private things (partner's `only_me` notes, scheduled cards) must not leak.

## Options considered

| Option | Verdict |
|--------|---------|
| O1 — DRF permission classes only | ❌ Object permissions never run in ADR-001 views; a missed filter leaks. |
| O2 — Server-resolved actor + couple-scoped ports + fail-closed ORM access + domain policies ⭐ | ✅ Three independent layers, each testable. |
| O3 — O2 + PostgreSQL Row-Level Security | ✅ Strongest guarantee. ❌ Per-request `set_config` on every connection, admin/worker/migration roles, harder debugging. **Documented as a backlog seam**, not v1. |

## Decision

**O2.** The frozen chain maps to layers like this:

| Chain step | Where | Mechanism |
|------------|-------|-----------|
| `IsAuthenticated` | `interfaces` | JWT authentication class (ADR-004). |
| `IsCouplePartner` | `interfaces` | DRF permission: user is active, has a couple, couple not archived. Builds the **`Actor`**. |
| `IsSameCouple` | `application` + `infrastructure` | Every repository/query port **requires** `couple_id`; ORM access without it raises. |
| `IsAuthor` | `domain` | Entity policies, e.g. `memory.ensure_editable_by(actor)`. |

### 1. The `Actor`

`Actor(user_id: UserId, couple_id: CoupleId, locale: str)` lives in `shared/domain`. It is built once per
request from the authenticated user's membership (DB, cached in Redis for the access-token lifetime), **never**
from a URL, body or header. Use cases receive it as their first argument. Jobs build it from the job payload
and re-check that the user still belongs to that couple.

### 2. Scoped ports (types enforce the tenant)

```python
class MemoryRepository(Protocol):
    def get(self, couple_id: CoupleId, memory_id: MemoryId) -> Memory | None: ...
    def save(self, couple_id: CoupleId, memory: Memory) -> None: ...
```

There is no method without `couple_id`. mypy strict rejects a call that forgets it.

### 3. Fail-closed ORM access

Couple-owned models use a manager whose default queryset **raises** unless scoped:
`Memory.objects.for_couple(couple_id)`. An explicit `Memory.unscoped` manager exists only for `backoffice`
metadata queries and migrations; a CI test fails if `unscoped` appears outside `backoffice/infrastructure/`.

### 4. 404 vs 403

| Situation | Response | Why |
|-----------|----------|-----|
| Resource belongs to another couple | **404** | REQ-N-SEC-02 — existence is secret. |
| Partner's `only_me` note | **404** | DEC-014 — the partner must not learn it exists. |
| Partner's scheduled card (content/date) | **404**; only the counter endpoint shows it | REQ-F-CARD-06, DEC-007 |
| Partner's shared item, edit/delete attempted | **403** | The partner can already see it; only the action is forbidden. |
| Own card already published, edit attempted | **409** | Rule conflict (REQ-F-CARD-07), not a permission problem. |

Repositories return `None` for invisible items, so "not visible" and "not found" are the same code path.

### 5. Staff

| Role | Implementation | Can reach content? |
|------|----------------|--------------------|
| `support` | Django group with `view_*` permissions on metadata models only | No |
| `admin` | `support` + change permissions on Couple, Invite, quotas, birth date | No |
| `superadmin` | `is_superuser`, manages staff and groups | No |

Content models (Card, Memory, Song, Place, Note) are **not registered** in Django Admin. Backoffice views read
through `backoffice` query services that return counts, sizes and dates only (REQ-F-ADM-04, REQ-N-SEC-08).

### 6. Sync

`POST /sync/push` applies each queued operation through the same use cases, so every operation passes the same
`Actor` and policies. An operation on an entity outside the actor's couple is rejected individually as
not-found; it never aborts the whole batch (detail in ADR-006).

### 7. Tests required (plan F)

- One test per row of the permission matrix in `roles-actuators.md`.
- Cross-couple test for **every** endpoint: couple B's token on couple A's ID → 404 (generated, not handwritten).
- Unit tests for each `ensure_*` policy with in-memory fakes.

## Consequences

- ✅ Three independent defenses: actor resolution, scoped ports + failing managers, domain policies.
- ✅ Ownership rules are pure domain code, covered by fast unit tests.
- ✅ Partner privacy and tenant isolation share one rule: invisible = not found.
- ❌ Every port signature carries `couple_id` — more verbose, intentionally.
- ❌ Actor cache must be invalidated when a couple is archived (REQ-F-AUTH-09); plan F gives it a step.
- ⚠️ RLS remains the upgrade path if the project ever has untrusted code paths or multiple services.

## Next step

Camilo approves, rejects, or patches by section number. Then ADR-006 (offline sync model).
