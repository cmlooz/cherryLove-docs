# Session map

How to split work across chats without losing the constitution. Status: **draft**.

## Quick path

Keep **this docs repo** as the shared context. Each later session reads only `approved` files plus the one draft it owns.

## Gate

Session 0 (this repo v0) must exist before any parallel work. Requirement freeze (session A) must exist before B–I produce durable content.

## Sessions

| ID | Session | Produces | Parallel? | Needs |
|----|---------|----------|-----------|-------|
| 0 | Docs bootstrap | This tree, v0 requirements | done when you approve path | your path |
| A | Requirements freeze | Closed open questions, REQ IDs locked | no | 0 + your answers |
| B | UX/UI CherryLove | Screens, tokens, empty/error states | after A | A |
| C | Architecture ADRs | Ports, DB, auth, sync, OpenAPI skeleton | after A; parallel with B | A |
| D | Learning plan | Lessons, notebook index, official-doc sources | after A; richer after C | A, better with C |
| E | Agents and skills | Project skills + agent briefs | after A | A |
| F | Implementation plan — API | Phases, 1 commit/step, 1 PR/phase, tests | after C | A + C |
| G | Implementation plan — Web | same | after B + C | A + B + C |
| H | Implementation plan — Mobile | same | after B + C | A + B + C |
| H2 | Implementation plan — Core (`cherrylove-core`) | same | after C; parallel with F | A + C |
| I | Test plan + evidence template | REQ to test matrix | after F/G/H outlines | A + plans |

## Do not parallelize

- Web code and mobile code for a phase before that phase's API contract is approved.
- Skills that encode architecture before ADRs exist.
- Notebooks that teach a stack choice before the ADR exists.

## Context pack per session (copy into the new chat)

```text
Repo: C:\Users\jcos1\Documents\Personal\Code\Arch\CherryLove\cherryLove-docs
Read: STATUS.md, 00-governance/approval-protocol.md, glossary.md
Then read only the folder this session owns.
Do not implement product code.
Do not assume unanswered items in 02-requirements/open-questions.md.
Language of files: English.
```

## Next step

You approve the split. Then session A after open questions are answered.
