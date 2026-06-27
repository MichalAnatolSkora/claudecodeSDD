# The whole flow, end to end

> The complete SDD loop as a runnable sequence — what you do, which command, and what it produces, at each step. The overview has the [map](spec-driven-development-guide.md#the-whole-flow); this is the step-by-step. Built for teams of **1–10**, so most steps are skippable: **you enter where your change starts.** Depth for each step lives in the linked guides — this page is just the spine.

---

## At a glance

```
 set up:  the commands + a starter CLAUDE.md   (then keep it updated)
     │
     ▼
    /sdd-1-prd-new · /sdd-1-prd-review     idea → PRD   (skip if small/solo)
     │
     ▼
 ┌─ /sdd-2-features-from-prd               PRD → vertical slices
 │   │
 │   ▼
 │  /sdd-3-spec-new … /sdd-6-trio-check    the trio — gated, no code yet
 │   │
 │   ▼
 │  /sdd-7-implement                       agent, red→green per task
 │   │
 │   ▼
 │  ADR?                                   record a decision worth keeping
 │   │
 │   ▼
 │  merge & freeze
 │   │
 └───┘ loop: a shipped slice re-ranks the next slice
```

**Enter where your change starts** — you rarely run the whole thing:

| You have… | Start at | Skip |
|-----------|----------|------|
| a new product / a real change of direction | step 1 (PRD) | nothing |
| a working PoC, nothing written down yet | step 1 (PRD), via `/sdd-1-prd-from-poc` | the blank-page draft (it reads the PoC instead) |
| a rough idea, or an accepted PRD | step 2 (slice) | sometimes the PRD |
| a known single feature | step 3 (trio) | steps 1–2 |
| a one-line fix | step 3, short spec only | steps 1–2, plan, tasks |

---

## Step 0 — Set up the foundation

The docs the agent reads every session. Set them up on day one — then grow them as you go; `CLAUDE.md` especially is **living**, not write-once.

- **`CLAUDE.md`** at the repo root — conventions, stack, what NOT to do. The one file the agent always reads. Start it now, then **add a line every time you correct the agent or accept an ADR** — it grows with the project (steps 5–6 feed back into it). → [`claude-md-guide.md`](claude-md-guide.md)
- **`README.md`** — what the project is, how to run it.
- Copy the slash commands into your repo: `npx degit MichalAnatolSkora/claudecodeSDD/templates/.claude .claude` — the shipped files are namespaced and phase-numbered (`sdd-1-prd-new.md` → `/sdd-1-prd-new`) so they sort in pipeline order; this guide refers to every command by its full name.
- Add `ARCHITECTURE.md`, `DOMAIN.md`, `TESTING.md` **when they earn it** — not before. When `ARCHITECTURE.md` does earn its place, bootstrap it in one pass: `/sdd-2-architecture-from-prd` (greenfield — Q&A the foundational choices like hosting and datastore, stubbing an ADR per hard decision) or `/sdd-2-architecture-from-code` (existing codebase — reverse-engineer it). Either runs **once per project**; `/sdd-4-plan-validate` then checks every plan against the result. → [`adr-guide.md`](adr-guide.md)

That's the floor. Everything below is per change.

---

## Step 1 — Idea → PRD

- **Before, optionally:** informal user/market research feeds the PRD — see [overview § Before the PRD](spec-driven-development-guide.md#before-the-prd-research-and-discovery). Most 1–10 teams skip it (a paragraph from a few customer conversations is plenty).
- **Do:** `/sdd-1-prd-new "<1–3 sentences>"` — it sketches a lean PRD, then asks you the open questions and fills them in (you make the product calls). Then `/sdd-1-prd-review` — a read-only audit (specific users? measurable success criteria? ≥5 out-of-scope items?) before you slice.
- **Already have a working PoC?** Skip the blank page: `/sdd-1-prd-from-poc` reads the prototype, asks you the *why* it can't infer (users, problem, success), and writes the lean PRD — then continue at `/sdd-1-prd-review`.
- **Get:** `docs/prd/YYYY-MM-slug.md` — product-level *what & why*, humans-only (the agent implements from specs, not the PRD).
- **Skip if:** solo, small, or the what/why already fits in your head — a one-paragraph issue is enough.

→ [`prd-guide.md`](prd-guide.md)

## Step 2 — PRD → features

- **Do:** `/sdd-2-features-from-prd` — slices the PRD into **vertical, independently-shippable features**, prioritized, walking-skeleton first.
- **Get:** a short prioritized list, saved to `specs/FEATURES.md` on your OK — the project's feature index. Each row becomes one feature → one trio, and its `Status` tracks `planned → spec'd → in progress → shipped`.
- **Loop:** the slice list is a first guess, not a contract. A shipped slice often re-ranks it — re-run `/sdd-2-features-from-prd` to merge new or dropped slices in (it never resets your progress). The PRD freezes; the slice list doesn't.
- **Skip if:** you already know the single feature you're building.

→ [`prd-guide.md`](prd-guide.md) § "Slicing the PRD into features"

## Step 3 — Feature → trio

One `spec → plan → tasks` per feature, written **in order** — each locks down what the next needs. (The order is the discipline *within* a slice; across slices you loop — ship the thinnest, learn, slice the next.)

1. **`/sdd-3-spec-new "<feature>"`** → `specs/YYYY-MM-slug/spec.md` — goal, **acceptance criteria** (the test contract), out of scope.
2. **`/sdd-3-spec-review`** → audits it for gaps (read-only). Resolve the open questions.
3. **`/sdd-4-plan-from-spec`** → `PLAN.md` — technical decisions, file structure, constraints (the *how*).
4. **`/sdd-4-plan-validate`** → checks the plan against your ADRs + `ARCHITECTURE.md` (read-only).
5. **`/sdd-5-tasks-from-plan`** → `TASKS.md` — ordered steps, each ending in `→ verify:`; a Verification section maps every AC.
6. **`/sdd-6-trio-check`** → the gate: every AC has a task, no surprise files, no contradicted ADR. Run it **before** any code.

**Compress to fit the change:** a small feature → the **one-file trio** (all three sections in one file); a bugfix → a short `SPEC.md` only.

→ [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md) — the core guide, with full worked examples.

## Step 4 — Implement + test

- **Do:** `/sdd-7-implement` — the agent works `TASKS.md` task by task, **red→green** (write the failing test from the AC first, implement the minimum, run, green) and **commits at each green task** (a cheap rollback point). It ends with the break-the-code check below.
- **Don't trust green:** break the code on purpose; the test must go red. Review the agent's tests, especially the assertions.
- **Stuck?** If the agent can't build from a finished trio, that's a *spec gap*, not a cue to vibe-code — shrink the task, feed the missing context, or resolve an open question.
- **Before the PR (Step 6):** run the built-in `/code-review` for generic bugs and quality (and `/security-review` if the change touches a trust boundary), then read the diff yourself against the spec's acceptance criteria, `CLAUDE.md` conventions, and `plan.md` § File structure — does every AC actually hold, did the diff stay inside the planned files, any scope creep or dead code? This is a human gate, not a new pipeline command.

→ [`working-with-agents-guide.md`](working-with-agents-guide.md) · [`testing-guide.md`](testing-guide.md)

## Step 5 — Decide (ADR), when one shows up

- **Do:** hit a real, hard-to-reverse decision (ORM, auth, a message broker, a new pattern)? Write an **ADR** — context, decision, alternatives, consequences — and add it to the active-ADR list in `CLAUDE.md`.
- **Skip if:** the change makes no decision worth not relitigating later.

→ [`adr-guide.md`](adr-guide.md)

## Step 6 — Merge + freeze

- The PR description links the spec: `Implements: specs/YYYY-MM-slug/`.
- Update the stable layer **only if** a convention or boundary changed — a line in `CLAUDE.md`, an edit to `ARCHITECTURE.md`.
- `TASKS.md` post-merge: append `STATUS: shipped (PR #N, date)` to the spec, update the runbook if relevant, close the ticket.
- **The spec freezes.** If the feature changes later, write a *new* spec — never edit a shipped one. (Frozen as the intent you had at merge, not as a description of today's behavior — when spec and code later disagree, the code wins.)
- **Then loop, don't stop.** Merging a slice usually teaches you something about the next — let it re-rank the backlog before you start the next trio (Step 2). The flow loops back here; it doesn't end at freeze.

---

## Iterative loops — the same commands, run in cycles

The pipeline above is one pass; real work loops. Three loops reuse the commands you already have — **no new command needed**, you just re-enter the pipeline at the right point:

**Discovery loop** — *you can't write the acceptance criteria yet (a new UX, an unfamiliar integration, an algorithm you have to feel out).*

```
/sdd-3-spec-new "<feature>"   →  mark the spec `discovery`, leave the AC as Open Questions
spike  (a throwaway branch — no command; it teaches you the AC, then you delete it)
   →  fill in the AC the spike surfaced  →  /sdd-6-trio-check   (before the spike it reports
      "discovery — pending spike" instead of failing; now it must pass)
   →  /sdd-4-plan-from-spec  →  /sdd-5-tasks-from-plan  →  /sdd-7-implement
```
The spike is throwaway and never becomes the code. → [`spec-plan-tasks-guide.md` § Two modes](spec-plan-tasks-guide.md#two-modes-delivery-and-discovery)

**Code→spec loop** — *implementation shows an acceptance criterion was wrong.*

```
/sdd-7-implement   →  an AC turns out wrong (spec still Active, pre-merge)
   →  edit the spec in place + a dated `CHANGED during implementation:` note  →  /sdd-6-trio-check  →  back to /sdd-7-implement
```
The freeze starts at merge; until then the spec is allowed to change. → [`spec-plan-tasks-guide.md` § When the code shows the spec was wrong](spec-plan-tasks-guide.md#when-the-code-shows-the-spec-was-wrong)

**Re-slice loop** — *a shipped slice reshapes the backlog.*

```
/sdd-7-implement ships slice N   →  /sdd-2-features-from-prd   (re-run: merges new / dropped / reordered slices, never resets progress)
   →  pick the next slice  →  /sdd-3-spec-new …
```
The PRD freezes; the slice list doesn't. → [`prd-guide.md` § Slicing](prd-guide.md#slicing-the-prd-into-features)

These aren't extra ceremony — they're the same commands, entered at the point your change actually starts and again for the next slice. Most features are pure **delivery** (acceptance criteria known up front): run the pipeline once, top to bottom, and never touch a loop. Reach for a loop only when the work calls for it.

---

## Cheat sheet

```
# set up (CLAUDE.md is living — keep adding to it; see steps 5–6)
npx degit MichalAnatolSkora/claudecodeSDD/templates/.claude .claude    # + start CLAUDE.md
/sdd-2-architecture-from-prd # 0  foundation: ARCHITECTURE.md + ADRs, when it earns it (or /sdd-2-architecture-from-code for existing code)

# per change — enter where you start, skip the rest
/sdd-1-prd-new "<idea>"      # 1  idea → docs/prd/…              (skip if small/solo)
/sdd-1-prd-from-poc          #     PoC already built? reverse a lean PRD out of it
/sdd-1-prd-review            #     audit the PRD, resolve gaps
/sdd-2-features-from-prd     # 2  PRD  → prioritized features    (skip if you know the feature)
/sdd-3-spec-new "<feature>"  # 3  → spec.md   (goal + acceptance criteria + out of scope)
/sdd-3-spec-review           #     audit, resolve open questions
/sdd-4-plan-from-spec        #     → plan.md  (the how)
/sdd-4-plan-validate         #     check vs ADRs + ARCHITECTURE.md
/sdd-5-tasks-from-plan       #     → tasks.md (ordered, verify per step)
/sdd-6-trio-check            #     gate: every AC has a task, no surprises
/sdd-7-implement             # 4  work tasks.md red→green, commit per task, break-the-code
# write an ADR if a real decision shows up; add it to CLAUDE.md
# merge: link the spec, update the stable layer if it changed, freeze the spec
# then loop: a shipped slice re-ranks the next — /sdd-2-features-from-prd to merge it in, then the next trio
```

---

*This page is the spine — the order you actually run things in. The [map is in the overview](spec-driven-development-guide.md#the-whole-flow); the depth is in the linked guides; the heart is [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md).*
