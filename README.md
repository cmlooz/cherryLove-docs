# CherryLove documentation

Source of truth for CherryLove: product, architecture learning, and delivery evidence.

This repository does **not** contain application code. Web, mobile, and API live in sibling repos later, joined only by an approved contract.

## Quick path

1. Read `STATUS.md` — what is draft vs approved.
2. Review `02-requirements/` and `02-requirements/open-questions.md`.
3. Mark each open question **accepted / rejected / alternative**.
4. Do not start UX, ADRs, notebooks, skills, or code until the requirement freeze is approved.

## What this repo is

CherryLove is a personal programming challenge whose **primary goal is software architecture expertise**. React, React Native, and Django are the delivery channels, not the goal.

| System | Later repo (proposed) | Role |
|--------|------------------------|------|
| Docs | `cherryLove-docs` (this repo) | Requirements, ADRs, learning, tests, evidence |
| API + admin | `cherrylove-api` | Django, persistence, invites, logs |
| Web app | `cherrylove-web` | React, responsive, offline + sync |
| Mobile app | `cherrylove-mobile` | React Native (Expo), Android + iOS, offline + sync |
| Shared core | `cherrylove-core` | Generated contract types, API client, validation, sync engine, design tokens, i18n (GitHub Packages) |

## Document set

| Folder | Purpose |
|--------|---------|
| `00-governance/` | Approval protocol and parallel session map |
| `01-analysis/` | Product brief and naming |
| `02-requirements/` | Functional, NFR, roles, actuators, open questions |
| `03-ux-ui/` | Screens, tokens, flows (after requirement freeze) |
| `04-architecture/` | ADRs and ports (after requirement freeze) |
| `05-learning/` | Learning plan and lesson notebooks |
| `06-implementation/` | One plan per app: phases, commits, PRs |
| `07-testing/` | Test plan mapped to REQ IDs |
| `08-evidence/` | Proof per phase |
| `09-agents-skills/` | Proposed then authored agents/skills |

## Working rules

- Never assume. If a field is missing, it stays an open question.
- Every implementation traces to an approved `REQ-*` ID.
- Prefer validation over `try/catch`. Prefer typed contracts.
- One commit per implementation step. One PR per phase. One notebook per lesson. One test per learning phase.

## Next step

Product rules are decided (`00-governance/decision-log.md`). Answer OQ-001 and reply `freeze` on `02-requirements/` (see `STATUS.md`).
