---
id: GOV-004
status: in_review
approved_by: null
approved_at: null
---

# Session prompts

Copy-paste prompts to open each Claude Code session. Open Claude Code at the **workspace root**
(`C:\Users\jcos1\Documents\Personal\Code\Arch\CherryLove`) so it loads both `CLAUDE.md` files.

## Quick path

1. Check the gate in the table. Do not open a session whose gate is not met.
2. Paste the prompt file's content as the first message.
3. When the session ends, its outputs are `in_review` until you approve them in `STATUS.md`.

| Session | Prompt | Gate | Can run in parallel with |
|---------|--------|------|--------------------------|
| A — Requirements freeze | `A-requirements-freeze.md` | now | — |
| B — UX/UI | `B-ux-ui.md` | A approved | C, D |
| C — Architecture | `C-architecture.md` | A approved | B, D |
| D — Learning plan | `D-learning-plan.md` | A approved (richer after C) | B, C |
| E — Agents & skills | `E-agents-skills.md` | C approved | F, H2 |
| R — Code repo bootstrap | `R-repo-bootstrap.md` | C approved + GitHub connected | E |
| F — Plan: API | `F-plan-api.md` | C approved | H2, B |
| H2 — Plan: core | `H2-plan-core.md` | C approved | F, B |
| G — Plan: web | `G-plan-web.md` | B + C approved, H2 in review | H |
| H — Plan: mobile | `H-plan-mobile.md` | B + C approved, H2 in review | G |
| I — Test plan + evidence | `I-test-plan.md` | F, G, H, H2 phases in review | — |
