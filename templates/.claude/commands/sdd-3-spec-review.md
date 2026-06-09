---
description: Audit a draft spec.md for gaps (read-only; lists concerns)
argument-hint: [path to spec.md or spec folder] (optional)
---

# Review a spec for missing pieces

Audit a spec without modifying it. Target: `$ARGUMENTS`
If no path is given, use the most recently modified folder under `specs/` and tell me which one you picked.

## Audit checklist

Read the spec, then check:

1. Is the **Goal** phrased as a problem, not a feature?
2. Does **Out of scope** exclude at least three things a reader might assume?
3. Are **acceptance criteria** specific enough to write tests from?
4. Does **Impact on existing code** list real file paths (verify against `src/`)?
5. Are **Open Questions** genuine uncertainties — or decisions the author should already have made?
6. Are **references** current (verify ADR numbers exist in `docs/adr/`)?
7. Does anything **in scope contradict an existing ADR**?

## Output

Return a numbered list of concerns. **Do not modify the file** — list the gaps only.
