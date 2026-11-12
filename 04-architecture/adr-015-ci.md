---
id: ADR-015
status: approved
approved_by: Camilo
approved_at: 2026-11-12
amended_by: DEC-038 (§2 on GitHub Free), superseded by DEC-039 (public repos)
---

> **Note (DEC-039, 2026-11-12):** repos are public for now, so §2 branch protection works on GitHub Free as
> written. DEC-038 (hook-only enforcement for private repos) is superseded and kept for history. The versioned
> `pre-push` hook stays as a cheap second barrier.

# ADR-015 — Continuous integration

Implements: REQ-N-QA-04, REQ-N-QA-06, REQ-N-QA-03, REQ-N-ARC-03, DEC-004, DEC-023.
Builds on: ADR-001 §6, ADR-003 §2–3, ADR-005 §3, ADR-010 §6, ADR-012 §2, ADR-014 (suites and gates).

## Context

REQ-N-QA-04: CI on every PR runs lint, types, tests, dependency audit and build. Earlier ADRs added more checks
(contract drift, breaking changes, import contracts, translation parity, coverage gates, REQ matrix). The repos
are private on GitHub (account `cmlooz`) and `cherrylove-core` is published to GitHub Packages (DEC-004).

DEC-023 also has a consequence people often miss: **1 commit per step, 1 PR per phase** means the merge strategy
must keep every step commit. A squash merge would erase them.

## Decision drivers

1. One required check per repo that blocks merge when any gate fails.
2. Step commits survive the merge (DEC-023).
3. Supply-chain hygiene: pinned actions, least-privilege tokens, dependency audit.
4. Private-repo minutes are limited: expensive jobs only where they pay off.
5. No extra platform beyond GitHub.

## Options considered

| Topic | Options | Choice |
|-------|---------|--------|
| CI platform | **GitHub Actions** ⭐ · GitLab CI · CircleCI | Code, packages and PRs already on GitHub; no second platform |
| Merge strategy | Squash · merge commit · **rebase merge** ⭐ | Rebase keeps each step commit and a linear history |
| Python packaging | pip + pip-tools · Poetry · **uv** ⭐ | One fast tool for venv, lockfile (`uv.lock`) and exact pins |
| Node package manager | **npm** ⭐ · pnpm · Yarn | Ships with Node; publishes to GitHub Packages natively; one tool fewer |
| Dependency updates | **Dependabot** ⭐ · Renovate | Built into GitHub; no app to install |

## Decision

### 1. Pipelines per repo (on every PR and on `main`)

| Repo | Lint / format | Types | Tests | Extra gates | Audit | Build |
|------|---------------|-------|-------|-------------|-------|-------|
| `cherrylove-api` | ruff check + format check | mypy strict | pytest unit, then integration with PostgreSQL, Redis, MinIO service containers | import-linter; `makemigrations --check`; contract drift + `oasdiff breaking` (ADR-003); coverage gates (ADR-014 §7) | `pip-audit` | API and worker images |
| `cherrylove-core` | ESLint | `tsc --noEmit` strict | Vitest + coverage | i18n parity + error-code translations (ADR-010/012); generated `contract/` matches the pinned contract | `npm audit --audit-level=high` | package build |
| `cherrylove-web` | ESLint (`react/jsx-no-literals`) | `tsc` | Vitest | Playwright smoke + axe against the API image (§4) | `npm audit` | production bundle |
| `cherrylove-mobile` | ESLint | `tsc` | Jest (`jest-expo`) | `expo-doctor`; JS bundle export | `npm audit` | — (native builds are a production decision) |
| `cherryLove-docs` | Markdown lint | — | Front-matter check (`id`, `status`), STATUS ↔ files consistency, internal links | — | — | — |

Every repo also checks **Conventional Commits** on each commit of the PR and that the PR body cites REQ/DEC/ADR IDs
(REQ-N-QA-06), with a small script (no commitlint dependency). The REQ → tests matrix (ADR-014 §6) is uploaded as
a build artifact.

### 2. Merge rules (`main`, every repo)

- Branch protection: PR required, required check `ci` green, branch up to date, no force-push.
- **Rebase merge only** (squash and merge commits disabled), so DEC-023's step commits land intact.
- Review: Camilo approves every PR (approval protocol).

### 3. Security of the pipeline

- Third-party actions pinned to a **full commit SHA**, not a tag (tags can be moved).
- Every workflow declares minimal `permissions:` (default `contents: read`; `packages: write` only in publish jobs).
- Secrets only in GitHub Actions secrets; none in logs or images. The KEK (ADR-009) never exists in CI — tests
  generate a throwaway key.
- Dependabot weekly for pip, npm and GitHub Actions, grouped per ecosystem; its PRs pass the same `ci`.

### 4. Cross-repo flow

| Event | Workflow |
|-------|----------|
| API merge touching `contract/` | Publishes contract asset `contract-v1.MINOR.PATCH` (ADR-003 §3) |
| API merge to `main` | Publishes API image to GitHub Container Registry (`ghcr.io`), tagged with the commit SHA and `main` |
| Core tag `vX.Y.Z` | Publishes `@cmlooz/cherrylove-core` to GitHub Packages |
| Web PR | E2E pulls the API image matching the contract version core was generated from |

Consuming the private core package from web and mobile needs the package's repository access granted to those
repos (GitHub Packages setting); session R configures it.

### 5. Cost control (driver 4)

- Linux runners only; no macOS runners in v1.
- Jobs ordered cheap → expensive with fail-fast (lint, types, unit before integration and E2E).
- Dependency caches for uv and npm.
- Mobile Maestro E2E (Android emulator) runs **nightly and on PRs labeled `e2e`**, plus once per phase PR — not on
  every push.

## Consequences

- ✅ One green `ci` check means every earlier ADR's rule held.
- ✅ Step commits preserved; history linear and bisectable.
- ✅ Pinned actions and scoped tokens close the usual CI supply-chain holes.
- ❌ Rebase merge requires branches to be rebased before merge (more care than squash).
- ❌ Mobile E2E feedback is not instant (nightly / labeled).
- ⚠️ Private-repo Actions minutes are capped by the GitHub plan; the integration and E2E jobs are the ones to watch.

## Next step

Camilo approves, rejects, or patches by section number. Then DATA-001 (data model).
