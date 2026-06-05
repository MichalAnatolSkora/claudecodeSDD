# The whole flow, end to end

> The complete SDD loop as a runnable sequence — what you do, which command, and what it produces, at each step. The overview has the [map](spec-driven-development-guide.md#the-whole-flow); this is the step-by-step. Built for teams of **1–10**, so most steps are skippable: **you enter where your change starts.** Depth for each step lives in the linked guides — this page is just the spine.

---

## At a glance

```
once:  write CLAUDE.md  +  copy the commands
                     │
per change ─▶ /prd-new ─▶ /features-from-prd ─▶ /spec-new … /trio-check ─▶ implement + test ─▶ ADR? ─▶ merge
             (idea→PRD)   (PRD→features)        (the trio)                 (agent, red→green)  (record) (freeze)
```

**Enter where your change starts** — you rarely run the whole thing:

| You have… | Start at | Skip |
|-----------|----------|------|
| a new product / a real change of direction | step 1 (PRD) | nothing |
| a rough idea, or an accepted PRD | step 2 (slice) | research, sometimes the PRD |
| a known single feature | step 3 (trio) | steps 1–2 |
| a one-line fix | step 3, short spec only | steps 1–2, plan, tasks |

---

## Step 0 — Set up once

The docs the agent reads every session. Do this on day one; grow the rest reactively.

- **`CLAUDE.md`** at the repo root — conventions, stack, what NOT to do. The one file the agent always reads. → [`claude-md-guide.md`](claude-md-guide.md)
- **`README.md`** — what the project is, how to run it.
- Copy the slash commands into your repo: `npx degit MichalAnatolSkora/claudecodeSDD/templates/.claude .claude`
- Add `ARCHITECTURE.md`, `DOMAIN.md`, `TESTING.md` **when they earn it** — not before.

That's the floor. Everything below is per change.

---

## Step 1 — Idea → PRD

- **Do:** `/prd-new "<1–3 sentences>"` — it sketches a lean PRD, then asks you the open questions and fills them in (you make the product calls).
- **Get:** `docs/prd/YYYY-MM-slug.md` — product-level *what & why*, humans-only (the agent reads specs, not the PRD).
- **Skip if:** solo, small, or the what/why already fits in your head — a one-paragraph issue is enough.

→ [`prd-guide.md`](prd-guide.md)

## Step 2 — PRD → features

- **Do:** `/features-from-prd` — slices the PRD into **vertical, independently-shippable features**, prioritized, walking-skeleton first.
- **Get:** a short prioritized list (in the PRD, an issue tracker, or `docs/roadmap.md`). Each row becomes one feature → one trio.
- **Skip if:** you already know the single feature you're building.

→ [`prd-guide.md`](prd-guide.md) § "Slicing the PRD into features"

## Step 3 — Feature → trio

One `spec → plan → tasks` per feature, written **in order** — each locks down what the next needs.

1. **`/spec-new "<feature>"`** → `specs/YYYY-MM-slug/spec.md` — goal, **acceptance criteria** (the test contract), out of scope.
2. **`/spec-review`** → audits it for gaps (read-only). Resolve the open questions.
3. **`/plan-from-spec`** → `plan.md` — technical decisions, file structure, constraints (the *how*).
4. **`/plan-validate`** → checks the plan against your ADRs + `ARCHITECTURE.md` (read-only).
5. **`/tasks-from-plan`** → `tasks.md` — ordered steps, each ending in `→ verify:`; a Verification section maps every AC.
6. **`/trio-check`** → the gate: every AC has a task, no surprise files, no contradicted ADR. Run it **before** any code.

**Compress to fit the change:** a small feature → the **one-file trio** (all three sections in one file); a bugfix → a short `spec.md` only.

→ [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md) — the core guide, with full worked examples.

## Step 4 — Implement + test

- **Do:** the agent works `tasks.md` task by task, **red→green** — write the failing test (from the AC) first, implement the minimum, run, green. **Commit at each green task** — a cheap rollback point.
- **Don't trust green:** break the code on purpose; the test must go red. Review the agent's tests, especially the assertions.
- **Stuck?** If the agent can't build from a finished trio, that's a *spec gap*, not a cue to vibe-code — shrink the task, feed the missing context, or resolve an open question.

→ [`working-with-agents-guide.md`](working-with-agents-guide.md) · [`testing-guide.md`](testing-guide.md)

## Step 5 — Decide (ADR), when one shows up

- **Do:** hit a real, hard-to-reverse decision (ORM, auth, a message broker, a new pattern)? Write an **ADR** — context, decision, alternatives, consequences — and add it to the active-ADR list in `CLAUDE.md`.
- **Skip if:** the change makes no decision worth not relitigating later.

→ [`adr-guide.md`](adr-guide.md)

## Step 6 — Merge + freeze

- The PR description links the spec: `Implements: specs/YYYY-MM-slug/`.
- Update the stable layer **only if** a convention or boundary changed — a line in `CLAUDE.md`, an edit to `ARCHITECTURE.md`.
- `tasks.md` post-merge: append `STATUS: shipped (PR #N, date)` to the spec, update the runbook if relevant, close the ticket.
- **The spec freezes.** If the feature changes later, write a *new* spec — never edit a shipped one.

---

## Cheat sheet

```
# once
npx degit MichalAnatolSkora/claudecodeSDD/templates/.claude .claude    # + write CLAUDE.md

# per change — enter where you start, skip the rest
/prd-new "<idea>"        # 1  idea → docs/prd/…              (skip if small/solo)
/features-from-prd       # 2  PRD  → prioritized features    (skip if you know the feature)
/spec-new "<feature>"    # 3  → spec.md   (goal + acceptance criteria + out of scope)
/spec-review             #     audit, resolve open questions
/plan-from-spec          #     → plan.md  (the how)
/plan-validate           #     check vs ADRs + ARCHITECTURE.md
/tasks-from-plan         #     → tasks.md (ordered, verify per step)
/trio-check              #     gate: every AC has a task, no surprises
# implement: red→green, commit per green task, break-the-code check
# write an ADR if a real decision shows up; add it to CLAUDE.md
# merge: link the spec, update the stable layer if it changed, freeze the spec
```

---

*This page is the spine — the order you actually run things in. The [map is in the overview](spec-driven-development-guide.md#the-whole-flow); the depth is in the linked guides; the heart is [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md).*
