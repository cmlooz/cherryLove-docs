# cherryLove-docs — Claude Code instructions

This repo is the **constitution** of CherryLove. It holds no application code.
Workspace-level rules live in `../CLAUDE.md`; read them first.

## Document rules

- Every durable doc starts with YAML front matter:
  ```yaml
  ---
  id: <PREFIX-NNN>
  status: draft | in_review | approved | rejected | superseded
  approved_by: null
  approved_at: null
  ---
  ```
- Only Camilo sets `status: approved`, `approved_by`, `approved_at`. Update `STATUS.md` in the same commit.
- `00-governance/decision-log.md` is **append-only**. A changed decision is a new DEC row that supersedes the old one.
- Superseded docs move to a `_superseded/` subfolder of their area with a header comment; never delete history.
- ID schemes: `REQ-F-<MODULE>-NN`, `REQ-N-<AREA>-NN`, `DEC-NNN`, `ADR-NNN`, `OQ-NNN`, `UX-NNN`, `LEARN-NNN`, `IMPL-<APP>-NNN`, `TEST-NNN`.
  After the requirements freeze, IDs are immutable.
- English, UTF-8, LF. Short sections: purpose → quick path → tables → next step.

## Commits in this repo

Conventional Commits with the `docs` type and an area scope, e.g. `docs(requirements): …`, `docs(adr): add ADR-001 …`.
One logical change per commit. Cite IDs in the body.

## Do not

- Write product code, scaffolds, or package manifests here.
- Start a session's durable content before its gate in `00-governance/session-map.md` is met.
