# Migrating a legacy repo to SDD

> How to add spec-driven development to a codebase you already have — without a two-week doc sprint, without faking the history of old decisions, and without stalling feature work. The agent drafts; you judge. That split is what makes it cheap.

---

## Table of Contents

1. [What migration is (and isn't)](#what-migration-is-and-isnt)
2. [Start with a 30-minute audit](#start-with-a-30-minute-audit)
3. [Week 1: the foundation](#week-1-the-foundation)
4. [Week 2 on: specs for new work only](#week-2-on-specs-for-new-work-only)
5. [Month 1 on: ADRs when something triggers them](#month-1-on-adrs-when-something-triggers-them)
6. [As needed: operational docs](#as-needed-operational-docs)
7. [Later: turn repeat prompts into commands](#later-turn-repeat-prompts-into-commands)
8. [More audit prompts](#more-audit-prompts)
9. [What goes wrong](#what-goes-wrong)
10. [What it looks like (worked examples)](#what-it-looks-like-worked-examples)
11. [Timeline and signals](#timeline-and-signals)
12. [Golden rules](#golden-rules)

---

## What migration is (and isn't)

You have a codebase. Maybe a `README`, maybe an old `ARCHITECTURE.md` nobody trusts. You want to use AI agents seriously. Where do you start?

**It is *not*:**

- Two weeks of full-time doc writing.
- Producing every file the method mentions.
- Writing specs for features you already shipped.
- Rewriting the docs you already have.

**It *is*:**

- Writing the 3–5 highest-value docs first, with the agent doing most of the typing.
- Using the spec → plan → tasks flow for *new* work from day one.
- Leaving old features undocumented until you touch them again.
- Adding the rest (ADRs, runbooks) only when something triggers it.

The agent drafts, you judge. That's what keeps it affordable.

---

## Start with a 30-minute audit

Before you write anything, have the agent survey the repo.

**Prompt:**

```text
Audit this repo and report:
1. Existing .md files — filename, rough age, one-line description.
2. The 5 most common code patterns in src/ (e.g. "all repositories follow X").
3. Up to 5 things that look like deliberate decisions (custom code where a
   library exists, unusual structure, workarounds with comments).
4. The domain — 5–10 business terms or abbreviations that show up in code/docs.
5. The test situation — frameworks, conventions, obvious gaps.

Output a markdown report. Don't propose changes yet — just survey.
```

The report tells you two things: what you already have (keep it — don't rewrite good docs), and what's *implicit* in the code but written nowhere. That second thing is the raw material for `CLAUDE.md`. Everything the audit doesn't surface, you don't need yet.

**When a doc contradicts the code, the code is the source of truth.** A legacy repo's docs lag its code by years; the code is what actually runs. When the audit surfaces a conflict, fix the doc or archive it — don't split the difference. Mark anything you can't verify against the code `[VERIFY]` rather than restating it as fact. And until the migration sweep is done, put one line in `CLAUDE.md`: *"Existing docs may be stale — when a doc and the code disagree, the code wins."* That single sentence stops the agent from faithfully implementing a doc that's been wrong since 2021.

---

## Week 1: the foundation

Week 1 produces 3 files (sometimes 4). Don't try for more.

### Day 1 — `CLAUDE.md`

The one file the agent reads every session. Draft it from the audit:

**Prompt:**

```text
From your audit, draft a CLAUDE.md. Sections:
- Project overview (1 paragraph)
- Stack (tech, versions, key libraries)
- Conventions (the patterns to follow — be specific)
- Do NOT (the anti-patterns — name them)
- File organization (where things live)
- Active decisions (placeholder; empty is fine on day 1)

Under 200 lines. Mark anything you're unsure of [VERIFY].
```

Then cut hard. The agent invents conventions that aren't real, misses the unwritten ones (team habits, decisions made in Slack), and pads with the obvious ("uses async/await"). Keep only what the agent would otherwise get wrong. Aim for ~150 lines, all specific to your repo. (Full how-to: [`claude-md-guide.md`](claude-md-guide.md).)

### Day 2–3 — `ARCHITECTURE.md`

The shipped command **`/architecture-from-code`** does this for you — it scans the source, drafts `ARCHITECTURE.md`, and additionally stubs a *reactive* ADR for each load-bearing decision already in force (the ORM, the scheduler, the service shape). The prompt below is essentially what it runs, if you'd rather drive it by hand:

**Prompt:**

```text
Read src/ thoroughly. Propose ARCHITECTURE.md with:
1. A high-level Mermaid diagram of the main components and their dependencies.
2. One paragraph per major component.
3. The key boundaries (what's allowed to call what).
4. Where cross-cutting concerns live (logging, errors, auth).

10,000-foot view, no implementation detail. Mark uncertainties [VERIFY].
```

Review it with someone who knows the system — the agent draws boundaries that aren't enforced and misses ones that are.

### Day 4 — `DOMAIN.md` (skip if the domain is generic)

**Prompt:**

```text
Read src/, README, and any docs. List every business term or abbreviation that
appears more than once. For each: a 1–2 sentence definition from how it's used,
and an example location. Mark uncertain ones [VERIFY DOMAIN EXPERT].
```

Have a domain expert check it — the agent gets definitions *almost* right, and almost-right is wrong here. Skip the file entirely if your domain is plain CRUD; a glossary that says *"User: a person who uses the system"* is noise.

### Day 5 — the first ADR

Pick the **one** decision the agent keeps fighting, or the one a new hire would re-litigate. Write it as `ADR-001`, `Accepted`, dated today, and say in the Context that it's written after the fact:

```markdown
## Context

This documents a decision originally made around 2022, written down today
because the agent keeps proposing [the rejected alternative]. Sources: team
memory and code archaeology — some detail is reconstructed.
```

Honest about being retroactive beats pretend-history. Add it to `CLAUDE.md`'s Active decisions. (Format and lifecycle: [`adr-guide.md`](adr-guide.md).)

---

## Week 2 on: specs for new work only

From week 2, every non-trivial change goes through the trio.

- **Don't** write specs for shipped features — reverse-engineered specs are fiction.
- **Don't** write specs for code you're not touching.
- **Do** write a spec for anything you'd explain in more than a paragraph: new features, real refactors.

After a month or two you'll have a `specs/` folder with 5–15 real entries, each linked from a PR. If you need to describe how an *existing* feature works (say, to onboard someone), that's `ARCHITECTURE.md` or a `docs/<feature>.md` — not a spec. **Specs describe what changed, not what exists.**

You don't have to hand-type the spec/plan/tasks prompts — the commands ship with this repo. Drop them in:

```bash
npx degit MichalAnatolSkora/claudecodeSDD/templates/.claude .claude
```

Then it's `/sdd-3-spec-new` → `/sdd-4-plan-from-spec` → `/sdd-5-tasks-from-plan` → `/sdd-6-trio-check` — the shipped files are phase-numbered; rename them if you'd rather drop the `sdd-N-` prefix. (The whole loop, step by step: [`flow-guide.md`](flow-guide.md). The trio itself: [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md).)

---

## Month 1 on: ADRs when something triggers them

Write an ADR when:

1. The agent suggests something that contradicts an unwritten decision → write it down, reference it from `CLAUDE.md`, correct the agent.
2. You're tempted to re-open a settled decision → write the ADR that closes the door (or `Supersedes` it).
3. A new hire asks *"why is this done this way?"* twice → a twice-asked question is an ADR waiting to happen.

Expect roughly **one a month** for the first six months. Don't write 20 ADRs in week 1 to "document the architecture" — most will be wrong, and all will rot. (Mechanics: [`adr-guide.md`](adr-guide.md).)

---

## As needed: operational docs

Each of these is born from a real event, not written on day 1:

| Trigger | Doc to create |
|---------|---------------|
| First production incident | `RUNBOOK.md` (an entry for that incident) |
| First *"what changed in version X?"* | `CHANGELOG.md` |
| First time onboarding takes > 1 hour | `ONBOARDING.md` + `.env.example` + `CONFIG.md` |
| First external contributor PR | `CONTRIBUTING.md` |
| Third *"how do we test X?"* | `TESTING.md` (see [`testing-guide.md`](testing-guide.md)) |
| First wrong code against an external API | `docs/integrations/<vendor>.md` |
| First incident worth not repeating | `docs/postmortems/<date>.md` |

Write any of these before the trigger and you get a doc that's wrong, generic, or both.

---

## Later: turn repeat prompts into commands

After a month of real use you'll notice yourself typing the same prompts. That's the signal to package them.

- **You don't have to build the trio commands** — this repo ships them ready-made (`/sdd-1-prd-new`, `/sdd-2-features-from-prd`, `/sdd-3-spec-new`, … `/sdd-6-trio-check`). Copy them with the one-liner above.
- **Your own repeated prompts** → a slash command. Anything you've typed 5+ times becomes a single `.claude/commands/<name>.md` (e.g. an `audit-docs` or `end-session` prompt).
- **Repeated read-heavy audits** (doc-staleness, ADR-vs-code checks) → a subagent, so they don't bloat your main session.
- **Invariants you want enforced** (block edits to accepted ADRs, check that shipped specs link a PR) → a `settings.json` hook.

Don't do this in week 1. The prompts that *look* recurring on day 1 are mostly wrong; the real ones surface after a few weeks of use. (Mechanics: [`working-with-agents-guide.md` § Claude Code Building Blocks](working-with-agents-guide.md#claude-code-building-blocks).)

---

## More audit prompts

The foundation prompts are in Week 1. A few more for digging:

**Find implicit decisions:**

```text
Find code that looks like a deliberate decision (custom code where a library
exists, unusual patterns, commented workarounds). For each: what was decided,
your best guess at why (cite comments/commits), and whether it's worth an ADR.
List candidates — don't write the ADRs. I'll pick.
```

**Find undocumented conventions:**

```text
Read 5 random files from each major directory in src/. Find patterns that repeat
but aren't in CLAUDE.md. For each: the convention (1 sentence), an example file,
and whether it's universal or partial. I'll decide what to add to CLAUDE.md.
```

**Check migration progress:**

```text
Compare this repo to a mature SDD one (CLAUDE.md, ARCHITECTURE.md, DOMAIN.md,
ADRs, specs/, runbook). For each: present? being updated (last commit)? any
stale content (dead commands, broken references)? Markdown report.
```

---

## What goes wrong

The common migration mistakes:

1. **The big-bang doc sprint.** Two weeks, 50 files, a wall of agent-confusing text — stale by week 3. Docs written away from the code change that motivated them drift immediately. *Fix:* foundation across week 1, then forward-only.
2. **Documenting decisions you weren't there for.** An `ADR-001` about *"why we chose Java in 2018"* with a made-up Context. *Fix:* document it as understood *today*, dated today, noting it's reconstructed.
3. **Specs for shipped code.** A *"spec for the 2022 auth system"* is fan fiction. *Fix:* describe existing features in `ARCHITECTURE.md` or `docs/<feature>.md`; specs are for what changes.
4. **`CLAUDE.md` as a wall of text.** 2,000 lines means the important rules get the same weight as filler. *Fix:* it's a pointer file — what matters and where to look. 100–250 lines; cut the generic.
5. **`CLAUDE.md` from `tree -L 3`.** *"src/ is for source code"* — the agent already knows. *Fix:* only write conventions the agent would otherwise get wrong.
6. **ADRs from `git log`.** The log has the *what*, not the *why* — and the why is the whole point. *Fix:* the agent drafts the structure; a human supplies the why.
7. **A runbook before any incident.** Untested *"restart the service"* steps become a 3 a.m. landmine. *Fix:* first incident → first entry, within 24h.
8. **Mirroring Confluence/Notion wholesale.** Most of it is discussion and status, not repo content. *Fix:* during the audit, link the pages that are still authoritative, distill the load-bearing ones into the repo, leave the rest.
9. **Archiving old docs on day 1.** The old doc is partly right; your fresh draft is partly wrong. *Fix:* keep both until the new one clearly wins — usually month 2+.
10. **Measuring by doc count.** *"30 markdown files!"* and the agent still drifts. *Fix:* measure by behavior — first-try suggestions matching conventions, corrections citing docs by name, faster onboarding.

---

## What it looks like (worked examples)

Same shape, different sizes.

**Solo Python web app** (5 years old, just a `README` + tests). Day 1: agent drafts `CLAUDE.md`; you cut 40% as obvious → ~120 lines. Day 2–3: `ARCHITECTURE.md` (Flask routes → services → repos → DB). Day 4: skip `DOMAIN.md` (generic CRUD). Day 5: `ADR-001 — Flask, not FastAPI` (the agent kept suggesting FastAPI). Week 2: first feature (rate limiting) gets a spec. Month 3: first prod incident → first runbook entry. End of Q1: 1 `CLAUDE.md`, 1 `ARCHITECTURE.md`, 2 ADRs, 6 specs, 1 runbook — all earned.

**.NET monorepo** (8 services, mid-sized team). A root `CLAUDE.md` for the shared rules (logging, errors, naming, DI) plus a small per-service `CLAUDE.md` each. Root `ARCHITECTURE.md` rewritten for the current state; the old one archived only after the new one stabilizes (month 2+). Confluence: link the authoritative pages, distill the degrading ones, leave the rest. ~3 weeks across all services in parallel. The trap: pressure to "unify everything" — resist; the root file holds only the genuinely shared rules.

**C# legacy** (200+ files, no docs) **and OSS projects** follow the same Week-1 shape. For OSS, the `CLAUDE.md` is written for *external contributors' agents* — scope, conventions, no-gos, and a pointer to `CONTRIBUTING.md` — and the trigger is the first low-quality AI-driven PR.

---

## Timeline and signals

**Rough timeline.** Week 1: foundation. Week 2: first spec, forward-only. Weeks 3–4: `CLAUDE.md` gets a few edits as you find gaps. Month 2: first reactive ADR. Month 3: maybe a first runbook entry. Month 6: mature — the agent matches conventions on the first try most of the time.

**Going well:** `CLAUDE.md` edited often early, then settling; specs get a date-slug within ~2 days of starting work; ADRs at ~1/month; the agent's first-try success visibly rising; people citing docs by name in review (*"per `CLAUDE.md` § Logging"*).

**Going badly:** `CLAUDE.md` untouched for 3 weeks (nobody reads it); specs written *after* the PR (that's documentation, not specification); ADRs piling up fast (fabricated, or the big-bang sprint); the agent still drifting; nobody references docs in review. If you see these, stop adding docs and audit what you have.

---

## Golden rules

1. **Foundation in a week, not a month.** `CLAUDE.md`, `ARCHITECTURE.md`, maybe `DOMAIN.md` and a first ADR — done by day 7.
2. **The agent drafts, you judge.** It reads source faster than you type, but it hallucinates — review heavily.
3. **Forward-only specs.** None for shipped features.
4. **Reactive ADRs.** Each has a trigger; no big-bang sprint.
5. **Operational docs follow reality.** First incident → first runbook entry. Not before.
6. **Trim before adding.** Every doc must pass: who reads it, when, what goes stale.
7. **One `CLAUDE.md` per repo.** Per-service files in a monorepo are fine; sharding one service's is not.
8. **Don't archive on day 1.** Keep old docs until the new ones clearly win.
9. **Honesty over completeness.** *"We're not sure when this was decided"* beats fabricated history.
10. **Migration ends when the agent stops drifting** — not when every doc exists. Measure behavior, not doc count.

---

*Companion to [`spec-driven-development-guide.md`](spec-driven-development-guide.md) (the steady-state method) and [`working-with-agents-guide.md`](working-with-agents-guide.md) (putting docs in front of the agent). Migration is the one-time trip from "no SDD" to "doing SDD"; after it, those guides take over.*
