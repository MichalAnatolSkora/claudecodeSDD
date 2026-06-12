---
description: Audit a draft PRD for completeness and clarity (read-only; lists concerns)
argument-hint: [path to the PRD] (optional)
---

# Review a PRD for gaps

Audit a PRD without modifying it. Target: `$ARGUMENTS`
If no path is given, use the most recently modified PRD under `docs/prd/` (or `PRD.md` at the root) and tell me which one you picked.

## Audit checklist

Read the PRD, then check:

1. Are the **users** specific (job title + company size or equivalent) — or generic ("users")?
2. Does the **problem** statement describe a specific behavior or cost today, not just "users want X"?
3. Are **success criteria** measurable (numbers and timeframe) — or vague ("users adopt it")?
4. Is **Out of scope** at least 5 items long? Are any of them surprising — things a reader might assume are in scope?
5. Does the solution stay free of **implementation detail** (languages, frameworks, databases, class names)? Flag any leakage — that belongs in specs.
6. Are **risks** named with consequences ("if X, then Y") — or generic?
7. Any **contradictions between sections** (e.g. an out-of-scope item reappearing as a constraint or feature)?
8. Is it still a **page or two**? A PRD that needs a table of contents is doing a spec's job.

## Output

Return a numbered list of concerns. **Do not modify the file** — list the gaps only.
A PRD that passes is ready for `/sdd-2-features-from-prd`.
