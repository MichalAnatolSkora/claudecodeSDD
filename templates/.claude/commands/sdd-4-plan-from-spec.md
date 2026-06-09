---
description: Draft plan.md (the "how") from a reviewed spec.md
argument-hint: [path to the spec folder] (optional)
---

# Draft a plan from a spec

Translate a reviewed spec into a technical plan. Spec folder: `$ARGUMENTS`
If no path is given, use the most recently modified folder under `specs/` and tell me which one.

## Steps

1. Read `spec.md` in that folder. Also read `CLAUDE.md`, `ARCHITECTURE.md`, and the active ADRs in `docs/adr/`.
2. Draft `plan.md` in the same folder using `templates/plan.md` as the structure:
   - **Technical decisions** — must align with active ADRs; cite them by number.
   - **Data model** — only if persistent state is involved.
   - **File structure** — concrete paths under `src/`.
   - **Constraints** — carry forward the spec's "Out of scope" plus any plan-level "do nots".
3. Keep **Open Questions** only for *how* questions that remain after the spec's open questions are resolved.

## Constraints

- Show me the draft. Mark anything uncertain with `[VERIFY]`.
- Don't re-decide what the spec already settled — cite the spec instead.
