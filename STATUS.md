# CherryLove documentation status

Control board for human approval. Nothing below `approved` is a decision.

**Repo:** `C:\Users\jcos1\Documents\Personal\Code\Arch\CherryLove\cherryLove-docs`
**Code layout (later):** private polyrepo — `cherrylove-api`, `cherrylove-web`, `cherrylove-mobile`, `cherrylove-core` + this docs repo (DEC-004)
**Docs language:** English (DEC-030) · **Encoding:** UTF-8

## Approval states

`not_started` → `draft` → `in_review` → `approved` | `rejected` | `superseded`

## Board

| ID | Document | Status | Approved by | Date |
|----|----------|--------|-------------|------|
| GOV-001 | `00-governance/approval-protocol.md` | in_review | | |
| GOV-002 | `00-governance/session-map.md` | in_review | | |
| GOV-003 | `00-governance/decision-log.md` | **approved** | Camilo | 2026-11-10 |
| GOV-004 | `00-governance/session-prompts/` | in_review | | |
| GLO-001 | `glossary.md` | in_review | | |
| AN-001 | `01-analysis/product-brief.md` | in_review | | |
| REQ-DOC-001 | `02-requirements/functional.md` | **approved** (frozen) | Camilo | 2026-11-11 |
| REQ-DOC-002 | `02-requirements/nfr.md` | **approved** (frozen) | Camilo | 2026-11-11 |
| REQ-DOC-003 | `02-requirements/roles-actuators.md` | **approved** (frozen) | Camilo | 2026-11-11 |
| REQ-DOC-004 | `02-requirements/open-questions.md` | **approved** (1 open post-freeze: OQ-002) | Camilo | 2026-11-11 |
| UX-001 | `03-ux-ui/` | not_started | | |
| C4-001 | `04-architecture/c4-context-containers.md` | **approved** | Camilo | 2026-11-11 |
| ADR-001 | `04-architecture/adr-001-modular-monolith.md` | **approved** | Camilo | 2026-11-11 |
| ADR-002 | `04-architecture/adr-002-folder-structure.md` | **approved** | Camilo | 2026-11-11 |
| ADR-003 | `04-architecture/adr-003-openapi-contract.md` | **approved** | Camilo | 2026-11-11 |
| ADR-004 | `04-architecture/adr-004-authentication.md` | **approved** | Camilo | 2026-11-11 |
| ADR-005 | `04-architecture/adr-005-couple-authorization.md` | **approved** | Camilo | 2026-11-11 |
| ADR-006 | `04-architecture/adr-006-offline-sync.md` | **approved** | Camilo | 2026-11-11 |
| ADR-007 | `04-architecture/adr-007-media-storage.md` | **approved** | Camilo | 2026-11-11 |
| ADR-008 | `04-architecture/adr-008-audit-log.md` | **approved** | Camilo | 2026-11-11 |
| ADR-009 | `04-architecture/adr-009-encryption-at-rest.md` | **approved** | Camilo | 2026-11-11 |
| ADR-010 | `04-architecture/adr-010-error-handling.md` | **approved** | Camilo | 2026-11-11 |
| ADR-011 | `04-architecture/adr-011-caching.md` | **approved** | Camilo | 2026-11-12 |
| ADR-012 | `04-architecture/adr-012-i18n.md` | **approved** | Camilo | 2026-11-12 |
| ADR-013 | `04-architecture/adr-013-notifications-jobs.md` | **approved** | Camilo | 2026-11-12 |
| ADR-014 | `04-architecture/adr-014-testing-strategy.md` | **approved** | Camilo | 2026-11-12 |
| ADR-015 | `04-architecture/adr-015-ci.md` | **approved** | Camilo | 2026-11-12 |
| DATA-001 | `04-architecture/data-001-data-model.md` | **approved** | Camilo | 2026-11-12 |
| API-001 | `04-architecture/api-001-openapi-v1.yaml` | **approved** | Camilo | 2026-11-12 |
| ADR-016 | `04-architecture/adr-016-pinned-versions.md` | **approved** | Camilo | 2026-11-12 |
| ADR-* | `04-architecture/` (index in README) | not_started | | |
| LEARN-001 | `05-learning/` | not_started | | |
| IMPL-API-001 | `06-implementation/api/impl-api-001-plan.md` | draft — **F0…F7 approved**; F8, F9 draft | Camilo (F0…F7) | 2026-11-13 |
| IMPL-* | `06-implementation/` (web, mobile, core) | not_started | | |
| TEST-001 | `07-testing/` | not_started | | |
| EVD-001 | `08-evidence/` | not_started | | |
| AGENT-001 | `09-agents-skills/README.md` | in_review | | |

## Frozen decisions

40 decisions approved → `00-governance/decision-log.md` (DEC-001…DEC-040).
Requirements frozen on 2026-11-11 (DEC-035). Sessions B, C, D, E are unblocked.
Architecture (session C) approved on 2026-11-12: C4-001, ADR-001…016, DATA-001, API-001. Sessions F, H2, R are unblocked; G and H also need B.

## Repositories (session R, 2026-11-12)

All public for now (DEC-039), owner `cmlooz`, default branch `main`, rebase merge only, `main` protected.

| Repo | URL | Content |
|------|-----|---------|
| `cherryLove-docs` | https://github.com/cmlooz/cherryLove-docs | this repo |
| `cherrylove-api` | https://github.com/cmlooz/cherrylove-api | bootstrap files only |
| `cherrylove-core` | https://github.com/cmlooz/cherrylove-core | bootstrap files only |
| `cherrylove-web` | https://github.com/cmlooz/cherrylove-web | bootstrap files only |
| `cherrylove-mobile` | https://github.com/cmlooz/cherrylove-mobile | bootstrap files only |

The required `ci` status check is added in the PR that introduces each repo's CI workflow (ADR-015).
The older empty repo `cmlooz/CherryLove` is not part of DEC-004 and was left untouched.

## Blocked on you

1. Approve GOV-001, GOV-002, GOV-004, GLO-001, AN-001, AGENT-001.
2. Answer OQ-002 (staff 2FA).

## Checklist for freeze

- [ ] Approval protocol accepted
- [ ] Glossary accepted
- [x] Roles and actuators accepted
- [x] Functional catalog accepted or patched by ID
- [x] NFR catalog accepted or patched by ID
- [x] OQ-001 answered (DEC-034)
- [ ] Agent/skill proposal accepted as a later session (E), not authored now
