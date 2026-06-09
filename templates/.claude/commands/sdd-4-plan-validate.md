---
description: Check plan.md against Accepted ADRs and ARCHITECTURE.md (read-only)
argument-hint: [path to the spec folder] (optional)
---

# Validate a plan against the architecture

Catch plan/architecture conflicts before implementation. Spec folder: `$ARGUMENTS`
If no path is given, use the most recently modified folder under `specs/` and tell me which one.

## Steps

1. Read `plan.md` in that folder, all Accepted ADRs in `docs/adr/`, and `ARCHITECTURE.md`.
2. For each **Technical decision** in the plan:
   - Does an existing ADR speak to this? If so, does the plan **match or contradict** it? Quote the relevant ADR text.
   - Does `ARCHITECTURE.md` establish a boundary the plan crosses?
3. For each **File structure** entry: does the path conform to the existing layout in `ARCHITECTURE.md`?

## Output

Return a markdown table: *plan decision | ADR/section it relates to | status (consistent / contradicts / not addressed)*.
**Do not modify the plan** — list findings only.
