# Migrating a Legacy Repo to Spec-Driven Development

> How to retrofit SDD onto an existing codebase without a documentation sprint, without writing fiction about old decisions, and without losing two weeks of feature work.

---

## Table of Contents

1. [Framing: what migration actually looks like](#framing-what-migration-actually-looks-like)
2. [Before you start: the 30-minute audit](#before-you-start-the-30-minute-audit)
3. [Phase 1 — Foundation (Week 1)](#phase-1--foundation-week-1)
4. [Phase 2 — Forward-only specs (Week 2+)](#phase-2--forward-only-specs-week-2)
5. [Phase 3 — Reactive ADRs (Month 1+)](#phase-3--reactive-adrs-month-1)
6. [Phase 4 — Operational layer (as triggered)](#phase-4--operational-layer-as-triggered)
7. [Reusable agent prompts](#reusable-agent-prompts)
8. [Anti-patterns](#anti-patterns)
9. [Worked examples](#worked-examples)
10. [Timeline and signals of success](#timeline-and-signals-of-success)
11. [Golden rules for migration](#golden-rules-for-migration)

---

## Framing: what migration actually looks like

You have a codebase. It has `README.md`, maybe `CONTRIBUTING.md`, possibly an `ARCHITECTURE.md` from 2021 that nobody trusts. You want to start using AI coding agents seriously, and you've read this repo's main guide. Now what?

**What migration is NOT:**

- Two weeks of full-time documentation writing
- Producing every doc the methodology mentions
- Writing retroactive specs for shipped features
- A clean-slate rewrite of the docs you already have

**What migration IS:**

- Writing the 3–5 highest-leverage docs FIRST, with heavy agent assistance
- Adopting spec-driven workflow for *new* work from migration day forward
- Accepting that old features stay undocumented until they're touched again
- Building up reactive documentation (ADRs, runbooks) as triggers happen

The agent does most of the drafting work. You do most of the judging. That split is what makes migration affordable.

---

## Before you start: the 30-minute audit

Spend half an hour before writing anything. The agent can do this for you.

### Audit prompt

```
Audit this repo and report:

1. What .md files currently exist? For each, give:
   - filename, approximate age (last modified), one-line description.
2. What code conventions do you see in src/? Name the 5 most common patterns
   (e.g., "all repositories follow X pattern", "controllers use Y").
3. What looks like a decision someone made deliberately?
   (custom code where a library exists, unusual structure, deliberate workaround).
   List up to 5.
4. What's the domain? List 5–10 business terms or abbreviations that appear
   in the code/docs.
5. What's the test situation? Frameworks used, conventions visible, gaps obvious.

Output as a markdown report. Don't propose changes yet — just the survey.
```

Read the report. Two things will be clear:

- **What you already have** that should stay (don't rewrite existing good docs).
- **What's implicit** in the code but undocumented — this is the source material for `CLAUDE.md`.

The audit becomes your migration scope. Everything else is YAGNI.

---

## Phase 1 — Foundation (Week 1)

The first week produces 3 files (occasionally 4). Don't try to do more.

### Day 1: Draft `CLAUDE.md`

```
Based on your audit, draft a CLAUDE.md for this repo.

Include sections:
- Project overview (1 paragraph)
- Stack (tech, versions, key libraries)
- Conventions (the patterns to follow — be specific)
- Do NOT (the most common anti-patterns — call them out by name)
- File organization (where things live)
- Active decisions (placeholder for ADR list, can be empty on day 1)

Keep it under 200 lines. Mark anything you're uncertain about as [VERIFY].
```

Review the draft heavily. The agent will:

- Hallucinate conventions that aren't real (*"uses dependency injection"* when actually it just instantiates classes)
- Miss the implicit ones that aren't in code (Slack-era decisions, team-level habits)
- Over-include obvious things (*"uses async/await"*)

Edit ruthlessly. Cut anything generic. Keep only what the agent would otherwise violate.

**Target:** ~150 lines, all specific to YOUR repo.

### Day 2–3: Draft `ARCHITECTURE.md`

```
Read src/ thoroughly. Propose ARCHITECTURE.md with:

1. A high-level Mermaid diagram showing main components/modules and their dependencies
2. A 1-paragraph description per major component
3. Key boundaries (what is allowed to call what)
4. Cross-cutting concerns (logging, error handling, auth) — where they live

Don't include implementation details. This is the 10,000-foot view.
Mark anything uncertain as [VERIFY].
```

Heavy review again. The agent will draw boundaries that aren't enforced and miss boundaries that are. Walk through with a teammate who knows the system; correct.

**Target:** 300–500 lines, including diagrams.

### Day 4: Draft `DOMAIN.md` (skip if domain is generic)

```
Read src/, README.md, and any existing docs. List every business term,
abbreviation, or domain-specific noun that appears more than once.

For each, propose:
- A 1–2 sentence definition based on usage context
- An example of where it appears in code

Mark uncertain ones [VERIFY DOMAIN EXPERT].

Output as DOMAIN.md / GLOSSARY structure.
```

Have a domain expert (PM, founder, senior engineer) review. The agent will get definitions almost-right; you need someone who can spot *almost-right is wrong* in this domain.

**Skip this file if your domain is generic** — there's no value in a glossary that says *"User: a person who uses the system."*

### Day 5: First ADR

Pick the **single** convention you've already corrected the agent on multiple times, or the single decision most likely to be re-litigated by a new contributor.

Write it as `ADR-001`. Status: `Accepted`, dated today, with a note in the Context section that it documents a prior decision:

```markdown
## Context

This ADR documents a decision originally made circa 2022, written down 
today because the agent has repeatedly proposed [the rejected alternative].
Team recollection and code archaeology are the sources; some detail is 
necessarily reconstructed.
```

Honesty about retroactive documentation > pretending it was written at decision time.

Add `ADR-001` to `CLAUDE.md`'s "Active Decisions" section.

---

## Phase 2 — Forward-only specs (Week 2+)

Starting in week 2, every non-trivial change uses spec-driven workflow.

**Hard rules:**

- **Don't write specs for shipped features.** They're fan fiction — reverse-engineered from code, not derived from intent.
- **Don't write specs for code you're not actively touching.** YAGNI applies to docs too.
- **Do write specs for:** new features, non-trivial refactors, anything you'd explain to a teammate in more than a paragraph.

After 4–6 weeks you have a `specs/` folder with 5–15 entries, all real, all referenced by PRs.

**Edge case:** if you absolutely need a written description of how an *existing* feature works (e.g., to onboard a new team member), that's `ARCHITECTURE.md` content or a dedicated `docs/<feature>.md` — not a `specs/` entry. Specs describe *what changed*, not *what exists*.

---

## Phase 3 — Reactive ADRs (Month 1+)

ADRs accumulate when triggered by:

1. **The agent makes a bad suggestion contradicting an unwritten decision.** Write the ADR documenting the decision; reference it from `CLAUDE.md`; correct the agent.
2. **You hit a decision you're tempted to re-litigate.** Write the ADR closing the door (or open a new one with `Supersedes`).
3. **A new hire asks "why is this done this way?" twice.** The two-asks rule: a question asked twice is an ADR waiting to happen.

You will accumulate ADRs slowly. Target: ~1 ADR per month for the first 6 months. By month 6 you'll have 5–10 ADRs, each earning its keep.

**Anti-pattern:** writing 20 ADRs in week 1 to *"document the architecture."* Most of them are wrong, some are obvious, all of them rot.

---

## Phase 4 — Operational layer (as triggered)

| Trigger | Doc to create |
|---------|---------------|
| First production incident | `RUNBOOK.md` (entry for that incident) |
| First *"what changed in version X?"* user question | `CHANGELOG.md` |
| First time onboarding takes > 1 hour | `ONBOARDING.md` + `.env.example` + `CONFIG.md` |
| First external contributor PR | `CONTRIBUTING.md` |
| Third *"how do we test X?"* question | `TESTING.md` |
| First wrong-code generation against an external API | `docs/integrations/<vendor>.md` |
| First incident worth not repeating | `docs/postmortems/<date>.md` |

Each is born from a specific need. Pre-emptively writing these on day 1 produces docs that are wrong, generic, or both.

---

## Reusable agent prompts

Reusable prompts you'll use throughout the migration. Paste and adapt.

### Initial audit prompt

```
Audit this repo and report:
1. Existing .md files (filename, age, one-line description)
2. 5 most common code patterns in src/
3. Up to 5 deliberate-looking decisions visible in code
4. 5–10 business terms / abbreviations from the codebase
5. Test situation (frameworks, conventions, gaps)

Markdown report. No proposed changes yet.
```

### CLAUDE.md draft prompt

```
Draft a CLAUDE.md based on the audit. Sections:
- Project overview (1 paragraph)
- Stack
- Conventions (specific, not generic)
- Do NOT (anti-patterns by name)
- File organization
- Active decisions (placeholder)

Under 200 lines. [VERIFY] on anything uncertain.
```

### ARCHITECTURE.md draft prompt

```
Read src/ thoroughly. Propose ARCHITECTURE.md with:
- High-level Mermaid diagram
- 1-paragraph description per major component
- Key boundaries (what is allowed to call what)
- Where cross-cutting concerns live (logging, error handling, auth)

10,000-foot view, no implementation detail. [VERIFY] uncertainties.
```

### DOMAIN.md draft prompt

```
Read src/, README.md, existing docs. List every business term, abbreviation,
or domain-specific noun appearing more than once.

For each:
- 1–2 sentence definition based on usage
- Example file location where it appears

Mark uncertain entries [VERIFY DOMAIN EXPERT].
```

### Find-implicit-decisions prompt

```
Search the codebase for code that looks like a deliberate decision
(custom implementation where a library exists, unusual patterns,
workarounds with explanatory comments).

For each:
1. Describe what was decided
2. Best guess at why (cite comments, commit messages, related code)
3. Mark whether this is worth an ADR

Don't write the ADRs yet. List candidates. I'll pick.
```

### Find-undocumented-conventions prompt

```
Read 5 random files from each major directory in src/. Look for patterns
that appear repeatedly but aren't called out in CLAUDE.md.

For each, propose:
1. The convention (1 sentence)
2. An example file showing it
3. Whether it's universal or partial

I'll decide what to add to CLAUDE.md.
```

### Migration-progress check prompt

```
Compare the current state of this repo to a mature SDD repo
(CLAUDE.md, ARCHITECTURE.md, DOMAIN.md, ADRs, specs/, runbook).

For each piece:
1. Is it present?
2. Is it being updated (last commit date)?
3. Does it have stale content (commands that no longer work, dead references)?

Markdown report: status, last update, staleness signals.
```

---

## Anti-patterns

What goes wrong in migrations.

### 1. The big-bang documentation sprint

Two weeks blocked off, write 50 files, produce a wall of agent-confusing text. By week 3 the docs are stale and nobody trusts them.

**Why it fails:** docs written without active context (i.e., not paired with the feature work that produced them) drift the moment they leave the keyboard. SDD discipline assumes docs are written *next to* the code change that motivated them — the sprint approach severs that link.

**Fix:** spread foundation across week 1, then forward-only.

### 2. Documenting decisions verbatim from 5 years ago

Someone writes `ADR-001` about *"why we chose Java in 2018"* without having been there. The "Context" section is fabricated; the "Consequences" section is what hindsight thinks was important.

**Fix:** ADR documents the decision *as currently understood*, dated today, with a note in Context: *"originally decided ~2018; documented 2026 based on team recollection and code archaeology."* Honest about retroactivity, not pretend-historical.

### 3. Retrofitting specs for shipped code

A *"spec for the auth system from 2022"* by someone who wasn't there is fan fiction. It's not a real spec — it's reverse-engineered description.

**Fix:** if you need to explain how the auth system works *today*, that's `ARCHITECTURE.md` content (or a dedicated `docs/auth.md`). Not `specs/`. Specs describe *what changed*, not *what exists*.

### 4. `CLAUDE.md` as a wall of text

A 2,000-line `CLAUDE.md` is worse than a 150-line one. The agent loads both, but the longer one can't be weighted equally — important conventions in the middle get the same attention as filler.

**Fix:** `CLAUDE.md` is a pointer file. It says *what's important and where to look*. Details live in other docs, linked from `CLAUDE.md`. Target 100–250 lines; aggressively cut anything generic.

### 5. Generating `CLAUDE.md` from `tree -L 3`

A `CLAUDE.md` that says *"`src/` is for source code, `tests/` is for tests"* is useless. The agent already knows that.

**Fix:** `CLAUDE.md` content is *conventions the agent would otherwise violate*. If the agent would guess it correctly, don't include it.

### 6. Auto-generating ADRs from `git log`

The git log knows WHAT changed, not WHY. An auto-generated ADR says *"we switched to Postgres"* but not why; the why is the entire point of an ADR.

**Fix:** ADRs are hand-written by humans who know the why (or by humans who can interview the people who know). The agent can draft the *structure*; you provide the *substance*.

### 7. Writing the runbook before there's been an incident

A runbook entry for *"restart the service"* written in week 1, with commands that haven't been tested, becomes a 3 a.m. landmine.

**Fix:** runbook entries are born from incidents. First incident → first runbook entry, written within 24h while the recovery steps are fresh.

### 8. Migrating Confluence (or Notion, etc.) wholesale

Treating Confluence as the source and trying to mirror it into the repo. Confluence accumulates discussions, half-finished decisions, status reports — most of which aren't repo content.

**Fix:** treat Confluence as a *source* during audit. For each Confluence page that's still load-bearing: either link to it from `CLAUDE.md` (if it's authoritative) or distill the key bits into the repo and mark the Confluence page as superseded.

### 9. Archiving old docs on day 1

You delete (or move to `docs/_archived/`) the 2021 `ARCHITECTURE.md` immediately, replacing it with a draft the agent generated yesterday.

**Fix:** keep old docs accessible during the transition. Move to `docs/_archived/` only once the new docs are clearly superseding them in practice — usually month 2+. The old doc is at least partially correct; the new one is at least partially wrong.

### 10. Measuring migration by doc count

Team celebrates *"we have 30 markdown files now."* Agent still drifts. Nothing improved.

**Fix:** measure migration by agent behavior. Are first-try suggestions matching conventions more often? Are corrections referencing specific docs by name? Are new contributors landing PRs faster? Doc count is a vanity metric.

---

## Worked examples

Four scenarios, each with a realistic migration shape.

### Example 1: 5-year-old Python web app, solo developer

**Starting state:** `README.md`, `requirements.txt`, `src/`, `tests/`. No other docs.

**Migration timeline:**

- **Day 1**: Agent reads `src/`, drafts `CLAUDE.md`. Solo developer reviews, cuts 40% of the draft as "obvious things." Final `CLAUDE.md`: ~120 lines.
- **Day 2–3**: `ARCHITECTURE.md` with Mermaid diagram of Flask routes → services → repositories → DB. ~300 lines.
- **Day 4**: `DOMAIN.md` not needed — domain is generic CRUD on user accounts and content.
- **Day 5**: `ADR-001 — Flask, not FastAPI` — written because the agent kept suggesting FastAPI patterns. ~40 lines.
- **Week 2**: First new feature (rate limiting) uses spec-driven workflow. `specs/2026-05-rate-limiting/` with `spec.md` + `plan.md`.
- **Month 2**: `ADR-002 — Custom session auth, not OAuth library` — written after the agent suggested adding `python-social-auth`.
- **Month 3**: First deploy to production. `RUNBOOK.md` gets its first entry (database connection pool exhaustion).
- **End of Q1**: 1 `CLAUDE.md`, 1 `ARCHITECTURE.md`, 2 ADRs, 6 specs, 1 `RUNBOOK.md` with 2 entries. All earned.

### Example 2: .NET monorepo, 8 services, mid-sized team

**Starting state:** `README.md` per service, a 2021 `ARCHITECTURE.md` at root that's 60% outdated, scattered Confluence pages.

**Migration approach:**

- **Root `CLAUDE.md`**: cross-service conventions (logging, error handling, naming, DI container patterns, repository style).
- **Per-service `CLAUDE.md`**: service-specific stack/patterns (5–8 of them, one per service).
- **Root `ARCHITECTURE.md`**: rewritten with current state. Old version moved to `docs/_archived/2021-ARCHITECTURE.md` *only after* the new one stabilizes (month 2+).
- **Per-service `ARCHITECTURE.md`**: each service team owns its own; root file just shows service-level boundaries and their interactions.

**Confluence handling:**

- Don't migrate wholesale. For each Confluence page that's still load-bearing:
  - **Authoritative and current** → link to it from `CLAUDE.md` ("see Confluence: …")
  - **Authoritative but degrading** → distill the load-bearing bits into the repo; mark the Confluence page superseded
  - **Historical / discussion** → leave on Confluence; don't pull into repo

**Timeline:** ~3 weeks for foundation across all 8 services, in parallel — each service team takes 1 week with their portion.

**Cross-cutting trap:** monorepo migrations attract pressure to "unify everything." Resist. Different services may legitimately have different conventions; root `CLAUDE.md` captures only the genuinely shared rules.

### Example 3: C# .NET legacy, no docs

**Starting state:** Big .NET solution, 200+ `.cs` files, no `.md` files except a stub `README.md`.

**Migration approach:**

- **Day 1–2**: agent reads `src/`, drafts `CLAUDE.md` focused on .NET-specific conventions (DI container patterns, naming, namespace organization, repository pattern usage, async/await usage).
- **Day 3–4**: `ARCHITECTURE.md` with layer diagram (API → Application → Domain → Infrastructure). Important: have a senior engineer walk through the diagram — agent will misread layer boundaries that look right but aren't enforced.
- **Day 5**: `ADR-001 — Repository pattern with Dapper, not EF Core` — captured because team had this decision but nothing documented; agent kept suggesting EF.
- **Week 2+**: every new feature uses spec-driven workflow. Specs live in `specs/YYYY-MM-feature/`.
- **Month 2**: `ADR-002 — Serilog with IBaseHandler<TSelf> contract` — written after agent generated inconsistent logging.
- **Month 3**: First production runbook entry, after first incident (e.g., stuck background job).
- **Month 6**: ~3 ADRs, ~12 specs, 1 runbook with 4 entries. Mature state.

### Example 4: Open-source project receiving AI-augmented contributions

**Starting state:** Mature OSS project. Has `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `README.md`. No `CLAUDE.md`.

**Migration approach:**

- **`CLAUDE.md` written for external contributors using AI tools.** Different audience than internal migrations:
  - Project scope/goals (so the agent doesn't propose out-of-scope features)
  - Conventions (style, testing, commit format)
  - What NOT to do (license/scope no-gos, architectural taboos)
  - Pointer to `CONTRIBUTING.md` for the human workflow (issues, PRs, review)
- **Cross-link `CLAUDE.md` ↔ `CONTRIBUTING.md`.** `CONTRIBUTING.md` tells humans how to contribute; `CLAUDE.md` tells their agent how to behave during the contribution.
- **Boundary discipline:** don't expose internal architecture details in `CLAUDE.md` if maintainers want to retain flexibility. Keep `CLAUDE.md` to *what the contributor's agent needs*, not the full internal map.
- **`ARCHITECTURE.md`** at the depth maintainers are comfortable being publicly committed to. Less detail than an internal repo's equivalent.

**Migration trigger:** OSS project's first AI-driven low-quality PR. The `CLAUDE.md` exists to reduce these.

---

## Timeline and signals of success

### Realistic timeline for a single mid-sized project

- **Days 1–7 (Week 1)**: foundation written (`CLAUDE.md`, `ARCHITECTURE.md`, optionally `DOMAIN.md`, first ADR). Agent reading existing code, you reviewing heavily.
- **Week 2**: first new feature spec written. Forward-only workflow active.
- **Weeks 3–4**: refinements based on first real use. `CLAUDE.md` gets 1–3 edits per week as the team discovers what's missing.
- **Month 2**: first reactive ADR (because the agent drifted). First spec status: `shipped`.
- **Month 3**: possibly first runbook entry. Probably 5–10 specs in `specs/`. Probably 2–3 ADRs total.
- **Month 6**: mature state. Docs match codebase. Agent generates code that matches conventions on first try most of the time.

### Signals migration is going well

- `CLAUDE.md` is updated more than once a week early on, then stabilizes
- New specs have a date slug within ~2 days of feature work starting
- ADRs accumulate at ~1/month, not 5/week
- Agent's first-try success rate is visibly improving over weeks
- Team members start citing files by name in code review (*"per CLAUDE.md § Logging"*, *"per ADR-003"*)
- New hires reach productivity faster than before migration

### Signals migration is going badly

- `CLAUDE.md` hasn't been touched in 3 weeks → it's not the source of truth anyone reads
- Specs are being written days/weeks *after* the PR → they're documentation, not specification; the spec→code→docs ordering broke
- ADRs are accumulating fast → you're either documenting decisions never actually made, or doing the big-bang anti-pattern
- Agent is still drifting → docs are wrong size, wrong content, or both
- Nobody in code review references the docs → the docs aren't trusted

If you see these signals, stop adding docs and audit what's there.

---

## Golden rules for migration

1. **Foundation in a week, not a month.** `CLAUDE.md`, `ARCHITECTURE.md`, optionally `DOMAIN.md`, optionally first ADR. Done by day 7.
2. **The agent drafts, you judge.** For legacy code, the agent reading source is faster than you typing. But the agent hallucinates; review heavily.
3. **Forward-only for specs.** Don't write specs for shipped features. Reverse-engineered specs are fiction.
4. **Reactive ADRs.** Each ADR has a specific trigger (agent drifted, decision re-litigated, twice-asked question). No big-bang ADR sprint.
5. **Operational docs follow operational reality.** First incident → first runbook entry. Not before.
6. **Trim before adding.** Every new doc should pass the rule of thumb (who reads it, when, what goes stale).
7. **One `CLAUDE.md` per repo, not per file.** Sub-`CLAUDE.md` files in a monorepo with strong service boundaries — fine. Sharding `CLAUDE.md` for a single service — kills the auto-load benefit.
8. **Don't archive in week 1.** Keep old docs accessible during the transition. Move to `docs/_archived/` only once new docs clearly supersede them — usually month 2+.
9. **Honesty over completeness.** *"We're not sure when this decision was made"* is a valid context entry. Better than fabricated history.
10. **Migration ends when the agent stops drifting.** Not when every doc the methodology mentions exists. Success is measured in agent behavior, not in doc count.

---

*This guide complements [`spec-driven-development-guide.md`](spec-driven-development-guide.md) (the steady-state SDD methodology) and [`working-with-agents-guide.md`](working-with-agents-guide.md) (how to put docs in front of the agent). Migration is the one-time process of getting from "no SDD" to "doing SDD reliably." After migration, the other two guides take over.*
