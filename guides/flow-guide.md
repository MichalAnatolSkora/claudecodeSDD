# The whole flow, end to end

> The complete SDD loop as a runnable sequence — what you do, which command, and what it produces, at each step. The overview has the [map](spec-driven-development-guide.md#the-whole-flow); this is the step-by-step. Built for teams of **1–10**, so most steps are skippable: **you enter where your change starts.** Depth for each step lives in the linked guides — this page is just the spine.

---

## At a glance

```
 set up:  the commands + a starter CLAUDE.md   (then keep it updated)
     │
     ▼
    /sdd-prd · /sdd-check                  idea → PRD   (skip if small/solo)
     │
     ▼
 ┌─ /sdd-prd                               PRD → vertical slices (its Feature breakdown)
 │   │
 │   ▼
 │  /sdd-spec · /sdd-check                 the trio — gated, no code yet
 │   │
 │   ▼
 │  /sdd-build                             agent, red→green per task
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
| a working PoC, nothing written down yet | step 1 (PRD), `/sdd-prd <path to the PoC>` | the blank-page draft (it reads the PoC instead) |
| a rough idea, or an accepted PRD | step 2 (slice) | sometimes the PRD |
| a known single feature | step 3 (trio) | steps 1–2 |
| a one-line fix | step 3, short spec only | steps 1–2, plan, tasks |

---

## Step 0 — Set up the foundation

The docs the agent reads every session. Set them up on day one — then grow them as you go; `CLAUDE.md` especially is **living**, not write-once.

- **`CLAUDE.md`** at the repo root — conventions, stack, what NOT to do. The one file the agent always reads. Start it now, then **add a line every time you correct the agent or accept an ADR** — it grows with the project (steps 5–6 feed back into it). → [`claude-md-guide.md`](claude-md-guide.md)
- **`README.md`** — what the project is, how to run it.
- Copy the slash commands into your repo: `npx degit MichalAnatolSkora/claudecodeSDD/templates/.claude .claude` — four per-change commands (`/sdd-prd`, `/sdd-spec`, `/sdd-check`, `/sdd-build`) plus the once-per-project `/sdd-architecture`.
- Add `ARCHITECTURE.md`, `DOMAIN.md`, `TESTING.md` **when they earn it** — not before. When `ARCHITECTURE.md` does earn its place, bootstrap it in one pass with `/sdd-architecture`: on an existing codebase it reverse-engineers the structure; on a greenfield project it Q&As the foundational choices (hosting, datastore) from the PRD — either way stubbing an ADR per hard decision. Run it **once per project**; `/sdd-check` then checks every plan against the result. → [`adr-guide.md`](adr-guide.md)

That's the floor. Everything below is per change.

---

## Step 1 — Idea → PRD

- **Before, optionally:** informal user/market research feeds the PRD — see [overview § Before the PRD](spec-driven-development-guide.md#before-the-prd-research-and-discovery). Most 1–10 teams skip it (a paragraph from a few customer conversations is plenty).
- **Do:** `/sdd-prd "<1–3 sentences>"` — it sketches a lean PRD, then asks you the open questions and fills them in (you make the product calls). Then `/sdd-check` on the PRD — a read-only audit (specific users? measurable success criteria? ≥5 out-of-scope items?) before you slice.
- **Already have a working PoC?** Skip the blank page: `/sdd-prd <path to the PoC>` reads the prototype, asks you the *why* it can't infer (users, problem, success), and writes the lean PRD — never a spec. Then `/sdd-check` as above.
- **Get:** `docs/prd/YYYY-MM-slug.md` — product-level *what & why*, humans-only (the agent implements from specs, not the PRD).
- **Skip if:** solo, small, or the what/why already fits in your head — a one-paragraph issue is enough.

→ [`prd-guide.md`](prd-guide.md)

## Step 2 — PRD → features

- **Do:** `/sdd-prd` again on the accepted PRD — it slices it into **vertical, independently-shippable features**, prioritized, walking-skeleton first.
- **Get:** the PRD's `## Feature breakdown` table, saved on your OK. Each row becomes one feature → one trio. No status column: what's shipped is what's merged under `specs/`.
- **Loop:** the slice list is a first guess, not a contract. A shipped slice often re-ranks it — re-run `/sdd-prd` to merge new or dropped slices in. The Feature breakdown is the one part of the PRD that stays editable; the rest freezes.
- **Skip if:** you already know the single feature you're building.

→ [`prd-guide.md`](prd-guide.md) § "Slicing the PRD into features"

## Step 3 — Feature → trio

One `spec → plan → tasks` per feature, written **in order** — each locks down what the next needs. (The order is the discipline *within* a slice; across slices you loop — ship the thinnest, learn, slice the next.) `/sdd-spec` writes the **next missing piece** each time you run it and shows you the draft before saving:

1. **`/sdd-spec "<feature>"`** → `specs/YYYY-MM-slug/spec.md` — goal, **acceptance criteria** (the test contract), out of scope. You confirm the AC.
2. **`/sdd-check`** → audits the spec for gaps (read-only). Resolve the open questions.
3. **`/sdd-spec`** → `PLAN.md` — technical decisions (citing your ADRs), file structure, constraints (the *how*).
4. **`/sdd-spec`** → `TASKS.md` — ordered steps, each ending in `→ verify:`; a Verification section maps every AC.
5. **`/sdd-check`** → the gate: the plan against your ADRs + `ARCHITECTURE.md`, every AC has a task, no surprise files. Run it **before** any code.

**Compress to fit the change:** a small feature → the **one-file trio** (all three sections in one file); a bugfix → a short `SPEC.md` only.

→ [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md) — the core guide, with full worked examples.

## Step 4 — Implement + test

- **Do:** `/sdd-build` — the agent works `TASKS.md` task by task, **red→green** (write the failing test from the AC first, implement the minimum, run, green) and **commits at each green task** (a cheap rollback point). It ends with the break-the-code check below.
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
- `TASKS.md` post-merge: update the runbook if relevant, close the ticket. No status to flip — **git is the status**: merged means shipped.
- **The spec freezes.** If the feature changes later, write a *new* spec — never edit a merged one. (Frozen as the intent you had at merge, not as a description of today's behavior — when spec and code later disagree, the code wins.)
- **Then loop, don't stop.** Merging a slice usually teaches you something about the next — let it re-rank the backlog before you start the next trio (Step 2). The flow loops back here; it doesn't end at freeze.

---

## When work loops — three rules

The pipeline above is one pass; real work loops. No extra commands and no modes — three rules, each re-entering the pipeline at the right point:

1. **Can't write the acceptance criteria yet?** (a new UX, an unfamiliar integration, an algorithm you have to feel out) **Spike first** on a throwaway branch, then write the spec from what it taught — `/sdd-spec`, and on through the trio. The spike never becomes the code. → [`spec-plan-tasks-guide.md` § When you can't write the acceptance criteria yet](spec-plan-tasks-guide.md#when-you-cant-write-the-acceptance-criteria-yet)
2. **The code shows an AC was wrong?** **Fix the spec before merge** — unmerged means not frozen. Edit it in place, a dated line in the trio's Notes says why, update plan and tasks, re-run `/sdd-check`, back to `/sdd-build`. → [`spec-plan-tasks-guide.md` § When the code shows the spec was wrong](spec-plan-tasks-guide.md#when-the-code-shows-the-spec-was-wrong)
3. **A shipped slice changed the backlog?** **Re-slice**: `/sdd-prd` merges new, dropped, or re-ordered slices into the PRD's Feature breakdown, then pick the next slice → `/sdd-spec`. → [`prd-guide.md` § Slicing](prd-guide.md#slicing-the-prd-into-features)

Most features need none of them: acceptance criteria known up front, the pipeline once, top to bottom. Reach for a rule only when the work calls for it.

---

## Cheat sheet

```
# set up (CLAUDE.md is living — keep adding to it; see steps 5–6)
npx degit MichalAnatolSkora/claudecodeSDD/templates/.claude .claude    # + start CLAUDE.md
/sdd-architecture            # 0  foundation, once per project: ARCHITECTURE.md + ADRs, when it earns it

# per change — enter where you start, skip the rest
/sdd-prd "<idea>"            # 1  idea → docs/prd/…              (skip if small/solo; or /sdd-prd <PoC path>)
/sdd-check                   #     audit the PRD, resolve gaps
/sdd-prd                     # 2  PRD → its Feature breakdown    (skip if you know the feature)
/sdd-spec "<feature>"        # 3  → spec.md   (goal + acceptance criteria + out of scope)
/sdd-check                   #     audit, resolve open questions
/sdd-spec                    #     → plan.md  (the how)
/sdd-spec                    #     → tasks.md (ordered, verify per step)
/sdd-check                   #     gate: plan vs ADRs, every AC has a task, no surprises
/sdd-build                   # 4  work tasks.md red→green, commit per task, break-the-code
# write an ADR if a real decision shows up; add it to CLAUDE.md
# merge: link the spec, update the stable layer if it changed — the spec is now frozen (git is the status)
# then loop: a shipped slice re-ranks the next — /sdd-prd to re-slice, then the next trio
```

---

*This page is the spine — the order you actually run things in. The [map is in the overview](spec-driven-development-guide.md#the-whole-flow); the depth is in the linked guides; the heart is [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md).*
