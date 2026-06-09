# SDD in 5 files

> Spec-driven development, the whole thing, on one page. **Five files** get you most of the benefit; everything else in this repo is reference you reach for *when a specific need fires* — not now. If this page is all you ever read, you're already doing SDD.

The point in one sentence: **write down what you want — your conventions, your intent, and how you'll know it works — before the agent writes code**, so its output matches your codebase instead of drifting away from it. AI agents fill silence with assumptions; these files remove the silence.

---

## The five files

| File | What it holds | When it's read |
|------|---------------|----------------|
| **`CLAUDE.md`** | conventions, stack, what NOT to do | the agent, **every session** — the one file it always loads |
| **`README.md`** | what this project is, how to run it | humans first; the agent as a fallback |
| **`ARCHITECTURE.md`** | layers, modules, boundaries — a diagram + ~200 words | the agent, when deciding *where* code goes |
| **`DOMAIN.md`** | your jargon and abbreviations (skip if the domain is generic) | the agent, to name things the way you do |
| **`specs/<date-slug>/spec.md`** | the one change you're making right now | the agent, **first**, before it writes code |

The first four are the **stable layer** — write them once, grow them slowly. The fifth is **per change** — a new little spec each time, frozen after it merges. That's the whole shape: a stable base the agent always reads, plus one spec per change.

> Truly minimal? Three files — `CLAUDE.md`, `README.md`, one `spec.md` — is the floor. Add `ARCHITECTURE.md` and `DOMAIN.md` the moment the agent starts guessing at structure or vocabulary. Below three, you're hoping, not specifying.

---

## The two that carry the method

**`CLAUDE.md`** — the agent's instruction hub. Keep it short (aim under ~300 lines). Skeleton:

```markdown
## Stack
- [language + version, framework, data layer, logging, test framework]

## Conventions
- [pattern 1, with a reference to an example file in this repo]
- [naming convention, with a concrete example]

## Do NOT
- [anti-pattern by name] · [a library you don't use] · [the wrong default the agent reaches for]

## Active decisions
- ADR-001 — [one-line summary]      ← only once you start writing ADRs
```

**`spec.md`** — one change, written *before* the code. The acceptance criteria are the part that matters most: they become your tests. Skeleton:

```markdown
# [Change name]

## Goal
[1–3 sentences: what problem, for whom, in which system.]

## Out of scope
- [what you are deliberately NOT doing — the highest-leverage line in the file]

## Acceptance criteria
- [ ] [testable + specific, e.g. "POST /api/orders over the per-partner limit → 429 with Retry-After"]
- [ ] ...
```

Copy-pasteable full versions: [`templates/CLAUDE.md`](../templates/CLAUDE.md), [`templates/spec.md`](../templates/spec.md).

---

## The loop, per change

```
1. Write spec.md          goal + acceptance criteria + out of scope
2. Agent implements it    each acceptance criterion → one failing test → make it pass (red → green)
3. Break the code         flip a value on purpose; the test must go red. Don't trust green.
4. Merge & freeze         append `STATUS: shipped (PR #N, date)`; never edit the spec again
5. Corrected the agent?   add ONE line to CLAUDE.md — that's how it stops drifting next time
```

That's it. For a bigger change, the spec grows two siblings in the same folder — `plan.md` (the *how*: decisions, file layout) and `tasks.md` (the *order*, each step ending in `→ verify:`). Spec → plan → tasks, written in that order, is the core loop; the [Feature Trio guide](spec-plan-tasks-guide.md) is the full treatment.

---

## Compress to fit the change

| Change | Write |
|--------|-------|
| one-line / config tweak | just a PR description — no spec |
| bug fix | a short `spec.md` (goal + acceptance criteria) |
| small feature | one file, three sections: spec / plan / tasks |
| real feature | the three-file trio in a `specs/<date-slug>/` folder |

When in doubt, write the shorter one. You can always promote a one-file trio into three files later — nothing is rewritten, only relocated.

---

## When you outgrow 5 files

Add the next document only when **something pulls it into being** — a real reader, a real recurring need — never because a methodology mentions it.

| Trigger | Add |
|---------|-----|
| third time you explain the test conventions to the agent | `TESTING.md` |
| first hard-to-reverse decision (ORM, auth, message broker) | `docs/adr/ADR-001-….md` |
| first production incident | a runbook entry |
| more than one person has to agree on the *what* | a one-page PRD |

A repo with 5 always-fresh files beats one with 30 stale ones. Discipline over completeness.

When you genuinely need the depth: the map to all of it is the [overview](spec-driven-development-guide.md) (this page distills its [Absolute Minimum](spec-driven-development-guide.md#the-absolute-minimum) section), and the one deep guide worth reading is the [Feature Trio](spec-plan-tasks-guide.md).

---
*Part of the [SDD guides repo](../README.md). This page is the front door; the method's heart is [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md).*
