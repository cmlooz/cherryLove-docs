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
| REQ-DOC-001 | `02-requirements/functional.md` | in_review | | |
| REQ-DOC-002 | `02-requirements/nfr.md` | in_review | | |
| REQ-DOC-003 | `02-requirements/roles-actuators.md` | in_review | | |
| REQ-DOC-004 | `02-requirements/open-questions.md` | in_review (1 open: OQ-001) | | |
| UX-001 | `03-ux-ui/` | not_started | | |
| ADR-* | `04-architecture/` | not_started | | |
| LEARN-001 | `05-learning/` | not_started | | |
| IMPL-* | `06-implementation/` | not_started | | |
| TEST-001 | `07-testing/` | not_started | | |
| EVD-001 | `08-evidence/` | not_started | | |
| AGENT-001 | `09-agents-skills/README.md` | in_review | | |

## Frozen decisions

33 decisions approved → `00-governance/decision-log.md` (DEC-001…DEC-033).

## Blocked on you

1. **OQ-001** — load-data generator in v1 or not.
2. **Requirements freeze** — reply `freeze` (or patches by ID) for REQ-DOC-001…004.
3. Approve GOV-001, GOV-002, GOV-004, GLO-001, AN-001, AGENT-001.
4. Connect GitHub (needed by session R).

## Checklist for freeze

- [ ] Approval protocol accepted
- [ ] Glossary accepted
- [ ] Roles and actuators accepted
- [ ] Functional catalog accepted or patched by ID
- [ ] NFR catalog accepted or patched by ID
- [ ] OQ-001 answered
- [ ] Agent/skill proposal accepted as a later session (E), not authored now
