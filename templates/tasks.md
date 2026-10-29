# Tasks: [Feature name]

> Copy this file to `specs/YYYY-MM-feature-slug/tasks.md` and fill in.
> Order matters. Each task has its own verification. Check off as you go.
> No status line — git is the status. Optional past ~5 people: an `Owner:` line — below that, `git blame` covers it.

## Setup

- [ ] [Any prerequisites — branch, DB migration, env setup]

## Implementation (in execution order)

- [ ] 1. [Task description] → verify: [how to confirm]
- [ ] 2. [Task description] → verify: [how to confirm]
- [ ] 3. ...

## Verification (against acceptance criteria)

- [ ] AC1: [from SPEC.md] → [test / manual check]
- [ ] AC2: [from SPEC.md] → [test / manual check]

## Post-merge

- [ ] Update the stable layer **only if this change moved it**: a new/changed convention → one line in CLAUDE.md; a changed component, boundary, or data flow → ARCHITECTURE.md; a new domain term → DOMAIN.md
- [ ] Close related issues / tickets

## Notes (append as you work)

- [YYYY-MM-DD]: [observation, blocker, decision made on the fly — or an acceptance criterion fixed before merge, and why]
