<!-- superseded on 2026-11-09 by the current file of the same name; kept for history (GOV-001) -->
# CherryLove documentation status

Control board for human approval. Nothing below `approved` is a decision.

**Repo:** `C:\Users\jcos1\Documents\Personal\Code\Arch\CherryLove\cherryLove-docs`

**Code layout (later):** polyrepo -- api, web, mobile + this docs repo

**Docs language:** English

## Approval states

`not_started` to `draft` to `in_review` to `approved` or `rejected` or `superseded`

## Board

| ID | Document | Status | Approved by | Date |
|----|----------|--------|-------------|------|
| GOV-001 | `00-governance/approval-protocol.md` | in_review | | |
| GOV-002 | `00-governance/session-map.md` | in_review | | |
| GLO-001 | `glossary.md` | in_review | | |
| AN-001 | `01-analysis/product-brief.md` | in_review | | |
| REQ-DOC-001 | `02-requirements/functional.md` | in_review | | |
| REQ-DOC-002 | `02-requirements/nfr.md` | in_review | | |
| REQ-DOC-003 | `02-requirements/roles-actuators.md` | in_review | | |
| REQ-DOC-004 | `02-requirements/open-questions.md` | in_review | | |
| UX-001 | `03-ux-ui/` | not_started | | |
| ADR-* | `04-architecture/` | not_started | | |
| LEARN-001 | `05-learning/` | not_started | | |
| IMPL-* | `06-implementation/` | not_started | | |
| TEST-001 | `07-testing/` | not_started | | |
| EVD-001 | `08-evidence/` | not_started | | |
| AGENT-001 | `09-agents-skills/proposed.md` | in_review | | |

## Frozen decisions (only what you already confirmed)

| Decision | Value |
|----------|--------|
| This session | Create docs repo + requirements v0 for approval |
| Docs language | English |
| Later code layout | Polyrepo: docs + api + web + mobile, joined by contract |
| Docs path | `CherryLove/cherryLove-docs` |

## Blocked on you

See `02-requirements/open-questions.md`. Architecture, UX, learning, and code stay blocked until those answers exist.

## Checklist for v0 approval

- [ ] Approval protocol accepted
- [ ] Glossary accepted
- [ ] Roles and actuators accepted
- [ ] Functional catalog accepted or patched by ID
- [ ] NFR catalog accepted or patched by ID
- [ ] Each open question answered (accept recommendation or give variant)
- [ ] Agent/skill proposal accepted as a later session, not authored now
