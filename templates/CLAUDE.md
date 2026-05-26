# CLAUDE.md

> **Attribution:** Sections 1–4 below ("Think Before Coding", "Simplicity First",
> "Surgical Changes", "Goal-Driven Execution") are a **literal copy** from
> https://github.com/multica-ai/andrej-karpathy-skills/blob/main/CLAUDE.md —
> kept verbatim as this project's "behavioral layer." Project-specific
> conventions follow below the `---` separator. If you fork or adapt this
> file, please keep this attribution so the source remains traceable.

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---

# Project-specific conventions

> Fill in the brackets below. Cut sections that don't apply; keep the shape.
> Aim for under ~300 lines total (including the behavioral layer above).

## What this project is

[1 paragraph. What the system does, who uses it, what success looks like.]

## Stack

- [Language + version]
- [Framework + version]
- [Data layer: ORM choice, database, etc.]
- [Logging library + contract / correlation rules]
- [Testing framework + conventions]
- [Anything else the agent must treat as fixed]

## Conventions

- [Pattern 1, with reference to an example file in this repo]
- [Pattern 2, with reference to an example file in this repo]
- [Naming convention with concrete examples]
- [Cross-cutting concern — logging, error handling, validation — and where it lives]

## Do NOT

- [Anti-pattern by name, with ADR reference if applicable]
- [Library or feature explicitly not used here]
- [Common LLM default that's wrong for this project]

## Active decisions (Accepted ADRs)

- **ADR-NNN** — [topic] — [one-line summary]
- **ADR-NNN** — [topic] — [one-line summary]

## Documentation map

| Task | Read first |
|------|------------|
| [Recurring task type] | [Doc and section to read] |
| [Recurring task type] | [Doc and section to read] |

## What to skip

- [Folder or file that looks authoritative but is not — e.g., `docs/_archived/`]
- [Spike / research material not for implementation use]

## How to update this file

- Add a line to **Conventions** the third time you correct the agent on the same thing.
- Update **Active decisions** when an ADR is added or superseded.
- Trim any line older than 6 months you can't justify in one sentence.
- If this file passes ~300 lines, move detail into a linked doc.
- **Don't edit the attribution block or sections 1–4** — those are upstream from
  multica-ai/andrej-karpathy-skills. If you want to change behavioral rules,
  fork those into your own clearly-marked section below this separator.
