---
id: ADR-002
status: approved
approved_by: Camilo
approved_at: 2026-11-11
---

# ADR-002 — Folder structure per repo

Implements: ADR-001, DEC-003, DEC-004, REQ-N-ARC-01/03/04/07, REQ-N-UX-01/03, REQ-F-SYNC-09.

## Context

ADR-001 fixed modules and layers for the API. This ADR turns them into folders, and gives the three
TypeScript repos an equivalent shape, so that any file's location answers two questions: *which part of
the business is this?* and *which layer is this?*

## Decision drivers

1. Screaming architecture: top-level folders name the business, not the framework.
2. Same mental model in every repo (core logic inward, platform adapters outward).
3. Each responsibility has exactly one home (no `utils/` or `services.py` dumping grounds).
4. Framework conventions respected where they are mandatory (Django app discovery, Expo Router `app/`).

## Options considered

| Option | Verdict |
|--------|---------|
| O1 — Group by technical type (`views/`, `models/`, `components/`, `hooks/` at the top) | ❌ hides the business; every feature touches every folder. |
| O2 — Group by module/feature, layers inside ⭐ | ✅ matches ADR-001; a module can be read, tested and moved alone. |

**Decision: O2 in all four repos.**

---

## 1. `cherrylove-api`

```text
cherrylove-api/
├── pyproject.toml            # deps, mypy, ruff, pytest config
├── manage.py
├── .importlinter             # ADR-001 §6 contracts
├── config/                   # Django project only: settings, root urls, wsgi/asgi
│   ├── settings/{base,local,test,production}.py
│   └── urls.py               # includes each module's interfaces/http/urls.py
├── src/cherrylove/
│   ├── shared/               # shared kernel
│   │   ├── domain/           # ids.py (UUIDv7 VO), couple_id.py, errors.py, events.py, clock.py
│   │   ├── application/      # unit_of_work.py, event_bus.py, audit_log.py (ports)
│   │   ├── infrastructure/   # django_unit_of_work.py, in_process_event_bus.py
│   │   └── interfaces/http/  # middleware/, problem_details.py, pagination.py, permissions.py
│   ├── identity/             # one folder per ADR-001 module (same inner shape)
│   │   ├── domain/           # couple.py, user.py, invite.py, events.py, errors.py
│   │   ├── application/
│   │   │   ├── commands/     # create_couple.py, accept_invite.py … (one use case per file)
│   │   │   ├── ports.py      # CoupleRepository, InviteRepository, EmailSender, PasswordHasher
│   │   │   ├── queries.py    # query-service ports + read DTOs (CQRS-lite)
│   │   │   ├── dtos.py       # command inputs/outputs
│   │   │   └── public.py     # facade other modules may import
│   │   ├── infrastructure/
│   │   │   ├── orm.py        # Django models
│   │   │   ├── mappers.py    # ORM ↔ domain
│   │   │   ├── repositories.py
│   │   │   ├── query_services.py
│   │   │   ├── adapters/     # smtp_email_sender.py, argon2_hasher.py …
│   │   │   └── migrations/
│   │   ├── interfaces/
│   │   │   ├── http/         # views.py, serializers.py, urls.py
│   │   │   ├── admin.py      # Django Admin registration (DEC-005)
│   │   │   ├── jobs.py       # worker entry points
│   │   │   └── event_handlers.py
│   │   ├── models.py         # Django discovery shim: re-exports infrastructure/orm.py
│   │   ├── apps.py           # AppConfig
│   │   └── wiring.py         # composition root: builds use cases with adapters
│   ├── cards/ memories/ diary/ favorites/ notifications/ sync/ audit/ backoffice/
└── tests/
    ├── unit/<module>/        # domain + application with in-memory fakes, no DB
    ├── integration/<module>/ # repositories, views, jobs against PostgreSQL
    ├── contract/             # generated OpenAPI matches the approved spec
    └── fakes/                # in-memory implementations of ports
```

**Why `models.py` sits at the module root:** Django only discovers models in `<app>/models.py`. A one-line
re-export keeps Django happy without moving ORM code out of `infrastructure`.

**Why `wiring.py` and no DI library:** the composition root is ~20 lines of plain constructors per module.
A container library adds a dependency (REQ-N-ARC-06) for no gain at this size.

## 2. `cherrylove-core` (TypeScript, platform-agnostic)

```text
cherrylove-core/src/
├── contract/          # GENERATED from OpenAPI — never edited by hand
├── api-client/        # typed client, request_id header, Problem Details parsing, AuthTransport port
├── validation/        # input schemas aligned with the contract
├── sync/              # engine: queue, replay, cursor pull, conflict handling
│   └── ports.ts       # LocalStore, Network, Clock — implemented by web/mobile
├── tokens/            # "Cherry" design tokens (REQ-N-UX-01)
├── i18n/              # es.json, en.json, typed keys (REQ-N-UX-03)
└── index.ts           # public exports (subpath exports per folder)
```

Rule: **no DOM, no React Native, no React imports in core.** Platforms plug in through ports
(`LocalStore` = IndexedDB on web, SQLite on mobile — REQ-F-SYNC-09). Core stays testable in plain Node.
Cost: each app writes its own thin hooks over core (accepted; they are ~1 file per feature).

## 3. `cherrylove-web` (React DOM)

```text
cherrylove-web/src/
├── app/               # entry, providers, router, error boundary
├── features/          # cards/ memories/ songs/ places/ diary/ favorites/ home/ account/ auth/
│   └── cards/
│       ├── pages/         # route-level containers (TodayCardsPage.tsx)
│       ├── containers/    # wire hooks → components (container-presentational)
│       ├── components/    # presentational, props only, no data fetching
│       ├── hooks/         # useTodayCards.ts → calls cherrylove-core
│       └── index.ts       # what the router may import
├── ui/                # design system from core tokens: atoms/ molecules/ organisms/ (atomic design)
└── platform/          # web adapters for core ports: indexeddb-store.ts, cookie-auth-transport.ts, web-push.ts
```

Tests colocated: `Component.test.tsx` next to the component.

## 4. `cherrylove-mobile` (Expo / React Native)

```text
cherrylove-mobile/
├── app/               # Expo Router routes — thin files that render a feature page
└── src/
    ├── features/      # same shape as web (pages/ containers/ components/ hooks/)
    ├── ui/            # atoms/ molecules/ organisms/ from core tokens
    └── platform/      # sqlite-store.ts, secure-storage-auth-transport.ts, expo-notifications.ts
```

`app/` at the root is mandatory for Expo Router; keeping it thin preserves the feature structure.

## 5. Where does each responsibility go?

| Responsibility | API | Web / Mobile | Core |
|----------------|-----|--------------|------|
| Controller / view | `<module>/interfaces/http/views.py` (parse → call use case → serialize) | `features/*/pages/` | — |
| Serializer | `interfaces/http/serializers.py` — shape and format validation only, no rules | — | `validation/` |
| Use case | `application/commands/<verb_noun>.py` | — | — |
| Domain rule | `domain/` (entity method or domain service) | never | never |
| Repository port | `application/ports.py` | — | `sync/ports.ts` |
| Repository impl | `infrastructure/repositories.py` | `platform/*-store.ts` | — |
| Adapter (S3, email, push) | `infrastructure/adapters/` | `platform/` | — |
| Middleware | `shared/interfaces/http/middleware/` (request_id, locale) | — | `api-client/` interceptors |
| Hook | — | `features/*/hooks/` | — |
| Presentational component | — | `features/*/components/` or `ui/` | — |
| "Service" | Not a folder. Domain service → `domain/`; application service = use case. | — | — |

## Consequences

- ✅ A PR's paths tell the reviewer the module and layer touched.
- ✅ Web and mobile share vocabulary; only `platform/` and route files differ.
- ❌ Deeper paths and more files than a starter template.
- ⚠️ Assumed framework conventions to confirm in the plans: Expo Router (Expo's default) for mobile;
  web router library is chosen in plan G with its own justification (REQ-N-ARC-06).

## Next step

Camilo approves, rejects, or patches by section number.
