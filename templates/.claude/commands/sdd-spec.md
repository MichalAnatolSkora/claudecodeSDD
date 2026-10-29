---
description: Write the next piece of a feature's trio — spec, then plan, then tasks — or add tasks to it
argument-hint: [feature idea | path or slug of an existing spec | tasks to add] (optional)
---

# The trio — spec → plan → tasks

Write the next missing piece of a feature's trio, in order. Input: `$ARGUMENTS`

## Find the target

- `$ARGUMENTS` names an existing spec (a `specs/<slug>/` folder or a one-file trio `specs/<slug>.md`) → continue that trio.
- `$ARGUMENTS` is empty → continue the most recently modified trio under `specs/`, and tell me which one.
- `$ARGUMENTS` describes a new feature → start a new spec. If it reads like an addition to an existing trio (*"tasks for the Retry-After header"*), ask which one.

Then do the **next step only**: no spec → **spec**; spec but no plan → **plan**; plan but no tasks → **tasks**; all three exist → **add the tasks** `$ARGUMENTS` describes. Show me the draft and wait; save only after I approve it, then offer the next step. Mark every guess `[VERIFY]` — especially file paths you inferred.

**Shape.** Default: a folder `specs/YYYY-MM-slug/` holding `spec.md`, `plan.md`, `tasks.md` (this month's date; a short slug from the idea). For a small feature (1–3 files, ~half a day) propose a **one-file trio** instead — `specs/YYYY-MM-slug.md` with `## Spec`, `## Plan`, `## Tasks` sections. A bug fix is a short spec only. Not sure → ask.

## 1 · Spec — the what & why (structure: `templates/spec.md`)

- **Goal** — 1–3 sentences: what problem, for whom, in which system.
- **In scope** — concrete, observable outcomes.
- **Out of scope (deliberately, not now)** — at least three things a reader might otherwise assume.
- **Acceptance criteria** — testable checkboxes; phrase each so it could become a test name. They become the tests, so I confirm them before we move on.
- **Impact on existing code** — specific file paths, after scanning `src/`.
- **Open questions** — do NOT answer them yourself; list what's genuinely undecided.
- **References** — related ADRs by number, prior specs by slug, integrations if relevant.

If the idea is too vague to scope, ask one round of clarifying questions first. If the **acceptance criteria can't be written yet** — a new UX, an unfamiliar integration, anything you'd only be guessing at — stop and say so: spike first on a throwaway branch, then come back and write the spec from what the spike taught.

## 2 · Plan — the how (structure: `templates/plan.md`)

If the spec still has unresolved Open questions, say so and stop — a plan written now would lock in a guess. Otherwise read `CLAUDE.md`, `ARCHITECTURE.md`, and the Accepted ADRs in `docs/adr/`, then draft:

- **Technical decisions** — must align with the Accepted ADRs; cite them by number.
- **Data model** — only if persistent state is involved.
- **File structure** — concrete paths under `src/`.
- **Constraints** — carry forward the spec's Out of scope plus any plan-level "do nots".
- **Open questions** — only *how* questions the spec left open. Don't re-decide what the spec settled — cite it.

## 3 · Tasks — in what order (structure: `templates/tasks.md`)

- Order red→green: an early task writes a failing test, later tasks make it pass. Group under headings past ~8 tasks.
- Every task ends with ` → verify: [how to confirm]`. A task without a verification step is incomplete.
- The **Verification** section maps every acceptance criterion to the task(s) that prove it.
- Use only file paths from the plan's File structure.

## Adding tasks to an existing list

Insert each new task **in execution order** — not blindly at the end; renumber if needed. Don't rewrite or reorder unrelated tasks. If a new task proves an acceptance criterion, add or adjust its Verification row. A task that needs a path not in the plan → flag it (the plan may need updating first). Show me the diff before saving.

## Next

Run `/sdd-check` before `/sdd-build` — it's the gate before code. Running it right after the spec, too, catches the gaps while they're cheap.
