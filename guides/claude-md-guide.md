# Writing a Good CLAUDE.md

> The single highest-leverage file in an SDD repo. This guide is the practical how-to: what goes in, what stays out, how big it should get, and how to keep it useful when your repo has 30+ other markdown files competing for attention.

---

## Table of Contents

1. [What CLAUDE.md is — and isn't](#what-claudemd-is--and-isnt)
2. [What goes in CLAUDE.md](#what-goes-in-claudemd)
3. [Writing principles](#writing-principles)
4. [Good vs bad: side-by-side examples](#good-vs-bad-side-by-side-examples)
5. [Sizing by project scale](#sizing-by-project-scale)
6. [The many-docs case](#the-many-docs-case)
7. [Anti-patterns](#anti-patterns)
8. [A starter template](#a-starter-template)
9. [Maintenance discipline](#maintenance-discipline)
10. [Golden rules](#golden-rules)

---

## What CLAUDE.md is — and isn't

`CLAUDE.md` is the file the agent loads automatically before doing anything in your repo. It is your *one guaranteed channel* to the agent. Everything else — `ARCHITECTURE.md`, `DOMAIN.md`, ADRs, specs — only gets read if `CLAUDE.md` (or your prompt) points at it.

That gives `CLAUDE.md` a peculiar dual role:

- It is the **conventions file** — the rules the agent must follow.
- It is the **discovery hub** — the map that tells the agent where to look for everything else.

If those two roles conflict, the discovery role wins. The convention text in `CLAUDE.md` should be the irreducible minimum the agent needs *right now*; everything else lives in linked detail files.

### What CLAUDE.md is NOT

- **It's not a README.** README is for humans landing on the repo. CLAUDE.md is for the agent that's about to write code. The audiences and goals are different.
- **It's not an architecture document.** Long structural prose belongs in `ARCHITECTURE.md`. CLAUDE.md may reference it; it should not duplicate it.
- **It's not a doc index by itself.** Pointing at docs is fine; *being only* a list of docs misses the convention role.
- **It's not a backup of the codebase.** *"`src/` is for source code, `tests/` is for tests"* contributes zero signal — the agent already knows that. CLAUDE.md is for *non-obvious* rules.
- **It's not a wall of text.** A 2,000-line CLAUDE.md is loaded into every prompt and dilutes attention. The right size is the smallest one that captures what the agent would otherwise get wrong.

---

## What goes in CLAUDE.md

A useful CLAUDE.md has these sections, roughly in this order:

### 1. Project overview (1 paragraph)

One paragraph: what the system does, who uses it, what success looks like. The agent uses this to choose idioms and avoid out-of-scope proposals.

> *"This is a B2B order export platform. It processes order data from internal databases, generates partner-specific files (XML or CSV), and delivers them via SFTP to ~12 banking and logistics partners. Used by operations team and partner integration engineers. Success = files delivered on time, with auditable history of every transmission."*

### 2. Stack (versions matter)

The tech the agent should treat as fixed. Versions matter — the agent's idioms for .NET 6 are different from .NET 8.

> - .NET 8, ASP.NET Core
> - Dapper (not EF Core — see ADR-007)
> - MS SQL Server 2022
> - Quartz.NET 3.x for scheduling
> - Serilog for logging, with `IBaseHandler<TSelf>` correlation contract
> - FluentValidation for input validation
> - NUnit + Testcontainers for integration tests

### 3. Conventions (specific, with examples)

The patterns the agent should follow. Each line should be specific enough that the agent can act on it without guessing.

> - All repositories follow the pattern in `src/Repositories/OrderRepository.cs`: SQL in `const string` at the top of the class, methods named after the intent (`GetOrdersByStatus`, not `Get`).
> - Handlers are pure (no I/O); I/O lives in repositories. See `src/Application/Handlers/_pattern.md`.
> - SQL keywords UPPERCASE, identifiers `snake_case` (`app.order_log`, `id`, `created_at`).
> - Logging: every method that touches the database calls `_logger.LogInformation` with the operation name and the order id (or batch id).

### 4. Do NOT (anti-patterns by name)

The mistakes the agent would otherwise make. This section is what makes a CLAUDE.md actually save you time.

> - Do NOT propose Entity Framework, DbContext, or LINQ-to-SQL (see ADR-007).
> - Do NOT add `MediatR` or CQRS. Handlers are called directly from controllers.
> - Do NOT use `Console.WriteLine` for logging — Serilog only.
> - Do NOT create new top-level folders without an ADR.
> - Do NOT write code in the `_archived/` folder; it's reference-only.

### 5. Active decisions (Accepted ADRs)

A one-line summary per active ADR. Update this when you add or supersede an ADR.

> - **ADR-001** — Dapper for data access (not EF Core)
> - **ADR-003** — Quartz for scheduling, configuration in `appsettings.json`
> - **ADR-007** — Repository-per-aggregate, SQL in constants
> - **ADR-014** — Per-environment Quartz config in database (supersedes ADR-003 from 2026-Q3)

Including this list serves two purposes: the agent gets the current decision set without reading every ADR file, and the next contributor sees what's settled vs unsettled.

### 6. Documentation map (essential if you have > 10 docs)

A table that tells the agent which doc to read for which type of task. This is the most underrated section.

> | Task | Read first |
> |------|------------|
> | Adding a new SQL query | `DOMAIN.md` (terminology), `src/Repositories/OrderRepository.cs` (pattern) |
> | New API endpoint | `ARCHITECTURE.md` § API, `docs/adr/ADR-005-rest-conventions.md` |
> | Background job | `ARCHITECTURE.md` § Scheduling, ADR-003 (or ADR-014 if 2026-Q3+) |
> | Production incident | `docs/runbooks/` (find by symptom) |
> | New external integration | `docs/integrations/_template.md`, `ARCHITECTURE.md` § Integrations |
> | Deployment / rollback | `OPERATIONS.md` § Release |

This converts CLAUDE.md from a passive instruction file into an active routing layer.

### 7. What to skip

Tell the agent what *not* to read. Especially valuable as the repo grows.

> - Do NOT read `docs/_archived/` unless I explicitly ask — it's historical.
> - Do NOT read `docs/research/` for implementation guidance — those are spike notes, not authoritative.
> - Do NOT load `docs/postmortems/` for routine work — only relevant when an incident is being investigated.

### 8. How to update this file (optional but underrated)

A note for future you (and future contributors and the agent itself):

> - Add a line to **Conventions** the third time you correct the agent on the same thing.
> - Update **Active decisions** when an ADR is added or superseded.
> - Trim any line older than 6 months that you can't justify in a sentence.
> - If this file passes ~300 lines, something is over-claimed — move details to a linked guide.

---

## Writing principles

A few rules that distinguish a CLAUDE.md that works from one that quietly becomes decoration.

### 1. Lead with what the agent would otherwise get wrong

CLAUDE.md content is *deltas from the agent's defaults*. If you use Flask, lead with that — because the agent would otherwise reach for FastAPI given the way modern Python projects look. If you use Dapper, say it — the agent would otherwise write EF.

If the agent would guess correctly without your line, the line is wasted.

### 2. Be specific, not generic

*"Use clean code"* contributes nothing. *"All public methods have XML doc comments; private methods don't unless they're non-obvious"* is actionable.

A useful test: can you derive a code review checklist from your CLAUDE.md? If not, it's too generic.

### 3. Reference files by path

*"Follow the existing repository pattern"* sends the agent searching. *"Follow `src/Repositories/OrderRepository.cs`"* sends it directly to the example.

File paths are the most token-efficient way to pin down what you mean.

### 4. Cap the length

100–300 lines is the sweet spot for most repos. Above ~500 lines, attention dilutes — the agent reads it but stops weighing each line equally.

If a section grows past 30 lines, that's a signal to extract it into a linked detail doc.

### 5. Update reactively, not preemptively

Don't try to anticipate every rule. Add a line when:

- You've corrected the agent on the same thing three times (the "third-time rule")
- The agent makes a mistake worth preventing
- A new ADR is accepted
- An old line is wrong and you noticed

Bulk rewrites tend to add more rot than they remove. Surgical updates compound.

### 6. Distinguish stable from active

Some content rarely changes (project overview, stack basics). Some changes regularly (ADR list, doc map as new docs land). Mixing them invites editing fatigue. Group the volatile content (Active decisions, Documentation map) so updating it is a quick scan, not a full re-read.

### 7. The "third time" rule

Three corrections on the same thing = a CLAUDE.md line. Fewer corrections = a one-time prompt. The threshold prevents bloat from one-off annoyances while still capturing real recurring issues.

---

## Good vs bad: side-by-side examples

The difference between a useful CLAUDE.md line and a useless one is usually about specificity. Examples:

### Conventions

**Bad:**
> Use the repository pattern.

**Good:**
> All repositories follow `src/Repositories/OrderRepository.cs`: SQL in `const string` at the top of the class, public methods named after intent (`GetOrdersByStatus`, not `Get`). No business logic inside repositories.

### Do NOT

**Bad:**
> Avoid bad practices.

**Good:**
> Do NOT use Entity Framework. We use Dapper (ADR-007). Don't propose `DbContext`, EF migrations, or LINQ-to-SQL — those are forbidden in this repo.

### Doc map

**Bad:**
> | Database stuff | See `docs/` folder |

**Good:**
> | Writing a new SQL query | `DOMAIN.md` for terminology, `src/Repositories/_pattern.md` for query style, `docs/schemas/app.sql` for current schema |

### Stack

**Bad:**
> Modern .NET stack.

**Good:**
> .NET 8 (not .NET Framework or .NET 6). ASP.NET Core minimal APIs (not MVC controllers). Dapper 2.x (not Dapper.Contrib for joins). MS SQL Server 2022.

The pattern across all of these: *specific names, version numbers, file paths, and explicit prohibitions* beat abstract intentions.

---

## Sizing by project scale

CLAUDE.md does **not** grow proportionally with the codebase. It plateaus. The job of CLAUDE.md is to point at *other* docs, not to absorb them.

| Project size | CLAUDE.md target | What it contains |
|--------------|------------------|------------------|
| Tiny (< 5 files of code) | Skip or 20–40 lines | Stack + 2–3 conventions. Maybe just live in README.md. |
| Small (5–30 files) | 80–150 lines | Stack + conventions + DO NOT. No doc map yet (not enough docs to map). |
| Mid-size (30–100 files) | 150–250 lines | All sections. Doc map essential. Active ADR list. |
| Large (100–500 files) | 200–300 lines | All sections, denser. Active ADR list is the most-edited section. Consider sub-CLAUDE.md per service in a monorepo. |
| Huge (500+ files, multi-team) | 200–300 lines at root + sub-CLAUDE.md per service | Root file is mostly an index. Per-service files carry the convention weight. |

Observation: even huge projects rarely need a CLAUDE.md > 300 lines at the root. If yours is heading there, the question is *what should move out*, not *what else can I squeeze in*.

---

## The many-docs case

This is where most CLAUDE.md files quietly stop being useful. A repo with 30+ markdown files needs `CLAUDE.md` to do more than carry conventions — it needs to **route attention**.

### The failure mode

Without a documentation map, the agent:

- Discovers docs by grep on the user's prompt — sometimes finding the wrong file (search noise)
- Loads files it doesn't need ("better safe than sorry")
- Misses the *authoritative* doc when an old / archived one is more prominent in search results
- Re-reads the same file across sessions because nothing tells it which is canonical

### Patterns that scale

**1. Document map as a first-class section.** Not a list of files; a *task → doc* mapping. The agent thinks *"I need to add a new endpoint"* and immediately knows to read `ARCHITECTURE.md § API`.

**2. Explicit "what to skip" rules.** With many docs, false positives multiply. Tell the agent which folders are historical, which are exploratory, which are authoritative.

**3. Convention precedence.** When two docs conflict, which wins? CLAUDE.md should state the rule:

> When `CLAUDE.md`, an ADR, and `ARCHITECTURE.md` disagree, follow this precedence: CLAUDE.md ➜ Accepted ADRs ➜ ARCHITECTURE.md. If they appear to disagree, flag it before acting.

**4. Sub-`CLAUDE.md` per directory (monorepo only).** Claude Code (and other tools) honor CLAUDE.md files inside subdirectories when the agent is working in that subtree. Per-service files keep each domain manageable:

```
/
├── CLAUDE.md                          # cross-service conventions
├── services/
│   ├── orders/
│   │   ├── CLAUDE.md                  # orders-specific stack, patterns
│   │   └── src/...
│   ├── payments/
│   │   ├── CLAUDE.md                  # payments-specific
│   │   └── src/...
```

Each per-service CLAUDE.md inherits the root file's spirit but layers in its own conventions. This breaks up the attention problem (the agent loads only what's relevant to its current task) without losing global rules.

**5. The "active list" pattern.** For ADRs in particular, the agent should never read every ADR — just the ones currently in force. CLAUDE.md's *Active decisions* section becomes the gate: only listed ADRs are authoritative; everything else in `docs/adr/` is historical context.

**6. Index files inside subfolders.** `docs/runbooks/README.md` lists every runbook with a one-line description. CLAUDE.md points at the README; the agent reads the README first; only then does it open the specific entry.

**7. Marking docs by lifecycle.** Add a `Status:` header to docs (`Status: Active`, `Status: Draft`, `Status: Archived`). CLAUDE.md instructs: *"Only read docs with `Status: Active`."* This solves the stale-content problem without manual pruning.

### The token-economy angle

The many-docs case is also the most expensive case. A doc map saves tokens directly:

- Without it: the agent runs grep, loads 5 candidate files, picks one. Cost: 10k tokens.
- With it: the agent reads the map, loads the named file. Cost: 1k tokens.

For repos that get worked on daily, a good doc map pays for the time to write it many times over in a single week.

---

## Anti-patterns

What kills a CLAUDE.md.

### 1. Wall of text

A 2,000-line CLAUDE.md is worse than no CLAUDE.md. The agent loads it but can't weight any of it. Trim ruthlessly; aim for 100–300 lines.

### 2. Generic best-practices

*"Write clean code,"* *"follow SOLID principles,"* *"use meaningful names"* — zero signal. The agent already does these by default. CLAUDE.md is for *deviations* from default, not affirmations of it.

### 3. Stale ADR list

The Active decisions section refers to ADR-002, which was superseded a year ago. Now the agent reads CLAUDE.md, follows ADR-002, and the human re-corrects the same drift every session. *The list is the discipline*; if it's not maintained, the file is actively misleading.

### 4. Stale stack info

CLAUDE.md says ".NET 6" but the codebase is on ".NET 8." Every prompt now starts with the agent writing slightly-outdated idioms.

### 5. Auto-generated content

*"Project structure: src/ contains source code, tests/ contains tests…"* generated from `tree -L 3`. The agent already knows this. Zero signal added; tokens wasted.

### 6. Hidden meta-rules

Convention `X` is implicit in the codebase but never mentioned in CLAUDE.md. The agent violates it every session. Either it deserves a line in CLAUDE.md, or it's not a real convention.

### 7. Conflicting authorities

CLAUDE.md says use Dapper. An old how-to in `docs/` says use EF. Neither references the other. The agent reads both and picks one — sometimes wrong, sometimes inconsistently across the same session.

### 8. Missing "Do NOT" section

Only positive guidance. The agent falls into known traps because the negative space isn't mapped.

### 9. Filing instead of summarizing

*"See `docs/X.md` for everything about Y."* A pure pointer with no summary. The agent has to load `docs/X.md` to get even the gist. CLAUDE.md should give the essence in a sentence and *then* point.

### 10. No update protocol

The file exists. Nobody owns it. Two years later, it documents a system that doesn't exist anymore. Add a "How to update this file" section so the discipline is itself documented.

---

## A starter template

Drop this into a fresh repo and fill in the brackets. Cut anything you don't need; don't add anything you can't justify in a sentence.

```markdown
# CLAUDE.md

> Conventions and meta-rules for working in this repository. The agent reads this
> before doing anything. Read it before editing code.

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

- [Pattern 1, with example file reference]
- [Pattern 2, with example file reference]
- [Naming convention with concrete examples]
- [Cross-cutting concern: logging, error handling, validation — and where it lives]

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

- [Folder or file that looks authoritative but is not — e.g. `docs/_archived/`]
- [Spike / research material not for implementation use]

## How to update this file

- Add a line to **Conventions** the third time you correct the agent on the same thing.
- Update **Active decisions** when an ADR is added or superseded.
- Trim any line older than 6 months you can't justify in one sentence.
- If this file passes ~300 lines, move detail into a linked doc.
```

---

## Maintenance discipline

A CLAUDE.md rots faster than any other doc in the repo, because it's the one the agent leans on most heavily. Stale content here causes the most drift.

### When to update

- **After an ADR is Accepted** — update Active decisions same day.
- **After an ADR is Superseded** — update Active decisions same day; remove the old ADR, add the new one.
- **After the agent makes the same mistake 3 times** — add a Conventions or Do NOT line.
- **After a stack upgrade** — update version numbers.
- **After a new detail guide is added to the repo** — add a row to the doc map.
- **Quarterly** — read the whole file. Trim anything you can't justify; verify every file path still exists.

### Ownership

CLAUDE.md without an owner is dead by month 6. Pick a maintainer (usually the engineer who set up the SDD workflow). Their job: review pull requests that change CLAUDE.md, run the quarterly trim, prompt the team when an ADR change should land in the Active list.

### The "agent-suggested" loop

A surprisingly good source of CLAUDE.md content is the agent itself. At the end of a session, ask:

```
Based on our work today, are there any conventions you noticed me correcting
you on that should be added to CLAUDE.md? List them — don't edit the file.
```

The agent often surfaces real candidates you forgot. You filter; you commit.

---

## Golden rules

1. **CLAUDE.md is for what the agent would otherwise get wrong** — not generic best practices, not the project's history.
2. **Specific beats generic, always.** Reference files by path; name patterns; cite ADRs by number.
3. **Cap at ~300 lines.** Above that, attention dilutes. The cure is trimming, not adding.
4. **Active decisions must be current.** A stale ADR list is the single most damaging form of rot.
5. **The doc map matters more the more docs you have.** It's the difference between a repo where the agent finds the right file and one where it grep-gambles.
6. **Mark what to skip.** Negative space (archived/, research/, draft/) is as important as positive guidance.
7. **Update reactively, not preemptively.** Third-time rule. Surgical edits compound; bulk rewrites add rot.
8. **One owner.** A file with no maintainer is dead by month 6.
9. **The agent can help maintain it.** Ask, in a session, what conventions kept coming up. Filter and commit.
10. **CLAUDE.md is the hub, not the warehouse.** When in doubt, summarize in CLAUDE.md and link to the detail file rather than duplicating it.

---

*This guide complements [`spec-driven-development-guide.md`](spec-driven-development-guide.md) (the methodology that produces the rest of the docs CLAUDE.md points at) and [`working-with-agents-guide.md`](working-with-agents-guide.md) (how the agent reads CLAUDE.md and what makes a file load well). The CLAUDE.md at the root of this repo is itself a worked example.*
