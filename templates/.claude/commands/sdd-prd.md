---
description: Draft a lean PRD (from an idea or a kept PoC), or slice an accepted one into features
argument-hint: [1–3 sentence idea | path to a PoC | path to a PRD to slice] (optional)
---

# PRD — draft it, then slice it

One command for the product layer. Input: `$ARGUMENTS`. Work out which job this is, tell me which you picked, and do only that one:

- **Draft from an idea** — `$ARGUMENTS` is a 1–3 sentence idea.
- **Draft from a PoC** — `$ARGUMENTS` is a path to working code, or I tell you a PoC came first.
- **Slice** — `$ARGUMENTS` is empty or names a PRD under `docs/prd/` (or `PRD.md`) that I've accepted. Run it again later to re-slice.

You draft and ask; **I make the product calls.** Keep it **factual and tight** — claims, numbers, and decisions, not marketing prose or filler. A human reviews this next (`/sdd-check`), so every sentence has to earn its place: verbosity is where gaps hide. A PRD stays a page or two (a one-pager if that's enough).

## Draft from an idea

1. **Sketch** a lean PRD at `docs/prd/YYYY-MM-slug.md` with these sections, filling only what the idea actually implies:
   *Summary · The problem · Target users · Solution overview · Success metrics · Constraints · Out of scope · Risks & assumptions.*
   No implementation detail (no language, framework, database, API, class names — that comes later, in specs). Mark anything you guessed `[VERIFY]`.
2. **Ask me** the open questions — the genuinely undecided things first: who exactly the user is, what success looks like in numbers, what's explicitly out of scope, any hard constraints. Group them, a few at a time, and wait for my answers.
3. **Fill** in my answers, show the updated draft, and ask another round only if real gaps remain. Stop when nothing important is still a guess.

## Draft from a PoC

For a working prototype you mean to **keep** — not a large/legacy system (that's `/sdd-architecture` and `guides/legacy-to-sdd-migration-guide.md`). Read `$ARGUMENTS` (default: `src/`, or the repo's main source dir — tell me which). **Code gives the *what*, not the *why*:** scan to sketch the what, then lead the Q&A with the why.

1. **Scan** what the PoC does today: entry points (CLI, API routes, UI, jobs), the capabilities it actually performs, the inputs it takes and outputs it produces, the external things it touches. Enough to describe behavior — not an architecture map.
2. **Sketch** the same lean PRD. Write **Solution overview** from the scan; leave the why-level sections thin until step 4. No implementation detail. Mark anything you inferred from the code `[VERIFY]`.
3. **Don't launder the PoC.** Where it made an implicit, undocumented, or questionable decision — or took a shortcut — surface it as `[VERIFY]`. The draft should read like a spike captured honestly, not like a plan that was followed.
4. **Ask me** what code can't supply, why-level first: the exact problem and its cost today, who precisely the user is (job title + context), success in numbers, what's deliberately out of scope, hard constraints. Then fill and iterate as above.

**The boundary — this stops at the PRD.** A PRD is product-level *what & why*; no agent writes code from it, so backfilling one is legitimate. **Never a spec:** no acceptance criteria, plan, or tasks for code that already exists — a spec reverse-engineered from finished code is fiction (`spec retro-fitted from completed code` in `guides/spec-plan-tasks-guide.md`). Spec-before-code starts with the **next** change: either harden the kept feature through a fresh `/sdd-spec` (the PoC was the throwaway spike), or treat the PoC as legacy from day one (forward-only specs). Offer `/sdd-architecture` to capture its structure — don't run it.

## Slice

1. Read the PRD — especially its success metrics and `Out of scope`.
2. Propose features as **vertical slices** — each independently shippable and user-visible. NOT layers: "DB layer" / "API layer" are not features.
3. Size each to ~a few days (a spec under ~150 lines). Split anything bigger; fold anything trivial into a neighbour.
4. Put the **walking skeleton** first — the thinnest end-to-end slice that proves the PRD's core — then order the rest by value × dependency and mark dependencies.
5. No slice may violate the PRD's `Out of scope` — flag any that would.
6. Lay it out as the PRD's `## Feature breakdown` table — *# · Feature · Serves (PRD outcome) · Size · Depends on* — and **show it to me.** No status column: what's shipped is what's merged under `specs/`.

**Re-slicing** — a breakdown already exists and a shipped slice taught us something (a new slice, a now-unnecessary one, a re-ordering): merge it in, and flag — don't delete — any row missing from the new breakdown. The Feature breakdown is the era's backlog, the one part of the PRD that stays editable; the rest freezes with the PRD.

**Don't write specs here.** I pick a row and run `/sdd-spec` for it.

## Saving (only after I accept)

Show me the full draft (or the new breakdown) first. Save only after I accept it.
