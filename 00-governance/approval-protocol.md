# Approval protocol

Nothing is true until you mark it approved. Status: **draft**.

## Quick path

1. Agent writes a document with `status: draft`.
2. You answer with accept / reject / change per ID or section.
3. Agent updates the file and sets `status: approved` only for what you accepted.
4. Implementation, UX pixels, ADRs, notebooks, and skills may use **approved** IDs only.

## Rules

| Rule | Meaning |
|------|---------|
| No assumption | Missing data becomes an open question, not a silent default in code. |
| ID traceability | Every commit/PR/test cites REQ IDs. |
| You approve | The agent recommends; it does not self-approve product or architecture. |
| English docs | All documents in this repo are English. Chat with you may stay in Spanish. |
| Docs before code | Application repos start after requirement freeze + enough ADRs to code the first phase. |
| Validation first | Prefer schema/domain validation over try/catch. Prefer typed contracts. |

## Document header (required)

Each durable doc starts with:

```markdown
# Title
Status: draft | in_review | approved | superseded
Approved by: (empty until you)
Approved at: (ISO date)
```

## States

| State | Meaning |
|-------|---------|
| draft | Agent proposal. Do not implement. |
| in_review | You were asked to decide. |
| approved | You accepted. Safe to use. |
| superseded | Replaced by a newer approved doc. Keep for history. |

## What you approve vs what the agent may decide

| You must approve | Agent may recommend and wait |
|------------------|------------------------------|
| Product behavior, roles, permissions | File layout inside an already approved shape |
| UX flows and visual system | Wording nits inside approved meaning |
| Database engine, auth, sync conflict rule | Library versions that implement an approved ADR |
| Repo layout, contract format | Commit message text that matches the step |
| Skills/agents to install | Skill body after you approve the skill list |

## Next step

Use this protocol on `02-requirements/`.
