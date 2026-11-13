---
id: ADR-016
status: approved
approved_by: Camilo
approved_at: 2026-11-12
verified_at: 2026-11-12
amended_by: DEC-043 (§4, core test pins, 2026-11-13)
---

# ADR-016 — Pinned versions

Implements: REQ-N-QA-05 (official sources first, exact pinned versions), REQ-N-ARC-06.
Pins every runtime, library and tool chosen in ADR-001…015, DATA-001 and API-001.

## How these were verified (2026-11-12)

| Source | Used for |
|--------|----------|
| PyPI JSON API (`pypi.org/pypi/<pkg>/json`) | Python packages: latest version, `requires_python`, `requires_dist`, Django classifiers |
| npm registry (`npm view`) | JS packages: latest version, `peerDependencies`, `engines` |
| `unpkg.com/expo@58.0.24/bundledNativeModules.json` | Versions Expo SDK 58 pins for React, React Native and Expo modules |
| endoflife.date API | Release and end-of-support dates (Python, Django, PostgreSQL, Node) |
| GitHub Releases API, Docker Hub API | CLI tools and container images |

REQ-N-QA-05 still applies: each implementation plan (F, G, H, H2) re-verifies these at its start and bumps
patch versions.

## The rule that drove the choices: newest is not always right

The newest versions do not fit together yet. Three examples found during verification:

| Newest | Problem | Consequence |
|--------|---------|-------------|
| Django **6.1.3** | `djangorestframework-simplejwt` 5.5.1 declares Django ≤ 5.2; `django-tasks-db`, `pytest-django` and `drf-spectacular` declare ≤ 6.0 | Pin the LTS every library declares |
| TypeScript **7.0.2** | `typescript-eslint` 8.72.2 requires `< 6.1`; `openapi-typescript` 7.13.0 requires `^5.x` | Pin TypeScript 5.9 |
| ESLint **10.14.0** | `eslint-plugin-react` 7.37.5 supports up to `^9.7` | Pin ESLint 9 |

## Decision 1 — Django 5.2 LTS, not 6.1 ✅ (approved)

| Option | Support ends | Ecosystem declares support | Notes |
|--------|--------------|----------------------------|-------|
| **Django 5.2.19 LTS** | 2028-04-30 | **all** chosen libraries | Tasks API through the `django-tasks` backport (same API as `django.tasks`) |
| Django 6.0.10 | 2027-04-30 | all except SimpleJWT | Shortest support window |
| Django 6.1.3 | 2027-12-31 | DRF and django-stubs only | Highest compatibility risk |

**Recommendation: 5.2.19 LTS**, then upgrade LTS → LTS to 6.2 once it ships (expected April 2027) and the libraries
catch up.

Effects on approved ADRs (no contradiction, adapters absorb them):
- ADR-013: the Tasks API is imported from `django_tasks` (backport) instead of `django.tasks`. Only the `JobQueue`
  adapter changes on the 6.2 upgrade. `django-tasks-db` officially supports this path (its `compat` extra targets
  Django `>=5.2,<6.0`).
- REQ-N-SEC-07 (CSP): Django 6.0's built-in CSP is not available on 5.2 → a ~20-line security-headers middleware in
  `shared/interfaces/http/middleware/` (no dependency).

## 2. Runtimes and infrastructure

| Component | Pin | Why |
|-----------|-----|-----|
| Python | **3.14.10** (`python:3.14.10-slim`) | Supported by Django 5.2; stdlib `uuid.uuid7()` exists (checked on 3.14.6) → ADR-006 §8 needs no helper. 3.15 (2026-10) is too new for the ecosystem |
| PostgreSQL | **18.6** (`postgres:18.6-alpine`) | ≥ 15 needed for `ON DELETE SET NULL (col)` (DATA-001); 18.7 is announced but its image is not published yet → bump in plan F |
| Redis | **8.8.4** (`redis:8.8.4-alpine`) | ADR-011 |
| Node.js | **24.x LTS** (24.22.3) | Maintenance LTS until 2028-04-30; accepted by Expo SDK 58 (`^24.3.0`). Node 26 became LTS on 2026-10-28; adopt it after the ecosystem settles |
| Mail catcher | **Mailpit v1.33.1** | REQ-N-OPS-01 |
| Object storage (local) | **chosen by plan F spike** (DEC-037) | See below |

### OQ-004 — MinIO is no longer maintained

Verified on GitHub: the `minio/minio` repository is **archived**, its README says "THIS REPOSITORY IS NO LONGER
MAINTAINED", the last release is `RELEASE.2025-10-15T17-29-55Z`, and the `minio/minio` image is no longer on Docker
Hub. DEC-009 says "S3-compatible object storage (MinIO locally)". The S3-compatible part stands; the MinIO part
needs a new decision. Recorded as **OQ-004** in `02-requirements/open-questions.md`.

## 3. API (`cherrylove-api`, Python)

| Package | Pin | Role (ADR) |
|---------|-----|------------|
| `Django` | 5.2.19 | 001 |
| `djangorestframework` | 3.18.3 | 001, 003 |
| `drf-spectacular` | 0.30.0 | 003 |
| `djangorestframework-simplejwt` | 5.5.1 | 004 |
| `argon2-cffi` | 25.1.0 | 004 |
| `psycopg[binary]` | 3.3.9 | PostgreSQL driver |
| `cryptography` | 50.0.2 | 009 |
| `boto3` | 1.44.1 | 007 |
| `Pillow` | 13.0.0 | 007 |
| `redis` | 8.1.0 | 011 |
| `django-tasks` (backport) | 0.12.0 | 013 |
| `django-tasks-db` | 0.13.0 | 013 |
| `pywebpush` | 2.5.0 | 013 |
| `requests` | 2.34.2 | 013 (Expo Push) |

Dev only: `mypy` 2.4.1, `django-stubs` 6.1.2, `djangorestframework-stubs` 3.18.2, `ruff` 0.17.4, `import-linter` 2.16,
`pytest` 9.2.0, `pytest-django` 4.14.0, `pytest-cov` 7.1.0, `hypothesis` 6.170.15, `pip-audit` 2.10.1.

System binary in the worker image: `ffmpeg` from the base image's Debian packages (ADR-007 §8).

## 4. TypeScript repos

| Package | Pin | Where |
|---------|-----|-------|
| `typescript` | **5.9.3** | all |
| `react`, `react-dom` | 19.3.1 (web); **19.3.0** (mobile, Expo SDK 58 pin) | web, mobile |
| `@types/react` | 19.3.0 | web, mobile |
| `expo` | **58.0.24** (SDK 58) | mobile |
| `react-native` | 0.88.1 (Expo SDK 58 pin) | mobile |
| `expo-router` 58.0.31, `expo-secure-store` 58.0.3, `expo-notifications` 58.1.9, `expo-sqlite` 58.0.16, `expo-image-manipulator` 58.0.16 | Expo SDK 58 pins | mobile |
| `vite` | 8.3.8 | web; core (dev, peer of `vitest`, DEC-043) |
| `@vitejs/plugin-react` | 6.3.0 | web |
| `openapi-typescript` | 7.13.0 | core (dev) |

Dev only: `vitest` 5.1.0 (core, web), `@vitest/coverage-v8` 5.1.0 (core, DEC-043), `jest` 30.5.2 + `jest-expo` 58.0.14 (mobile), `@testing-library/react` 16.3.3,
`@testing-library/react-native` 14.1.0, `fast-check` 4.10.2, `@playwright/test` 1.65.0, `@axe-core/playwright` 4.14.0,
`eslint` **9.39.5**, `typescript-eslint` 8.72.2, `eslint-plugin-react` 7.37.5.

Why `vite` in core (DEC-043, verified 2026-11-13 with `npm view`): `vitest` 5.1.0 declares `vite`
`^6.4.0 || ^7.0.0 || ^8.0.0` as a **non-optional** peer, so npm installs it anyway; pinning keeps it exact.
`@vitest/coverage-v8` 5.1.0 requires `vitest` 5.1.0 exactly and enables the ADR-014 §7 coverage gate.

## 5. CLI tools (CI and local, not project dependencies)

| Tool | Pin | ADR |
|------|-----|-----|
| `uv` | 0.13.7 | 015 |
| `oasdiff` | v1.37.0 | 003 |
| `k6` | v2.3.0 | 014 |
| Maestro CLI | 2.12.1 | 014 |
| Redocly CLI (contract lint) | 2.77.0 | 003 (used to validate API-001) |

## Consequences

- ✅ Every pinned library declares support for the pinned runtime/framework — no "it probably works".
- ✅ Two-year support window on the backend framework (5.2 LTS until 2028-04).
- ❌ We start one Django feature line behind (no built-in CSP or `django.tasks`); both bridged without dependencies.
- ❌ TypeScript 5.9 forgoes the native TS 7 compiler speed until the tooling supports it.
- ⚠️ Local object storage server is pinned by the plan F spike (DEC-037).

## Next step

Approved with Django 5.2 LTS. Next: OQ-004.
