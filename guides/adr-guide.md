# Writing Good Architecture Decision Records (ADRs)

> The full how-to for the third leg of SDD's documentation stool. Code shows *what* you did; ADRs document *why*. This guide covers the format, the lifecycle, the worked examples, the anti-patterns, and how to keep ADRs from rotting alongside your codebase.

---

## Table of Contents

1. [Framing — what an ADR is and isn't](#framing--what-an-adr-is-and-isnt)
2. [When to write an ADR (and when not)](#when-to-write-an-adr-and-when-not)
3. [The Nygard format, section by section](#the-nygard-format-section-by-section)
4. [Alternative formats](#alternative-formats)
5. [Status lifecycle in depth](#status-lifecycle-in-depth)
6. [The Supersedes pattern in depth](#the-supersedes-pattern-in-depth)
7. [Worked examples](#worked-examples)
8. [Numbering, naming, file organization](#numbering-naming-file-organization)
9. [Cross-referencing ADRs](#cross-referencing-adrs)
10. [The ADR review process](#the-adr-review-process)
11. [Agent-assisted drafting](#agent-assisted-drafting)
12. [Anti-patterns](#anti-patterns)
13. [Edge cases](#edge-cases)
14. [Tooling](#tooling)
15. [Maintenance discipline](#maintenance-discipline)
16. [Golden rules](#golden-rules)

---

## Framing — what an ADR is and isn't

An **Architecture Decision Record** is a short, dated, immutable document capturing:

- A decision the team made
- The context that made it necessary
- The alternatives considered
- The consequences accepted

Michael Nygard introduced the format in [2011](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions). The motivation: *"code shows what you did, but not why."* A year later you're staring at unusual code; without an ADR, the why lives only in someone's head — and that person might have left.

### What an ADR is NOT

- **Not a spec.** A spec describes a feature being built. An ADR describes a *decision* about how the system is shaped. Specs freeze after merge; ADRs are immutable from the day they're Accepted.
- **Not a design document.** Long structural explanations belong in `ARCHITECTURE.md`. An ADR is 1–2 pages, focused on *one* decision.
- **Not a post-mortem.** Post-mortems describe what went wrong; ADRs describe what was chosen. Different artifact, different lifecycle.
- **Not living documentation.** Once Accepted, you don't edit an ADR. You write a new one that supersedes it.

The most common mistake is over-scoping — turning the ADR into a mini-architecture document. Keep each one to a single decision.

---

## When to write an ADR (and when not)

### Write an ADR when a decision

- **Is hard to reverse.** ORM choice, message broker, authorization approach, primary database, deployment platform.
- **Has consequences beyond a single module.** Logging convention across the system. Error-handling protocol. Versioning strategy.
- **Was made against the obvious choice.** *"Why Dapper when the .NET standard is EF?"*, *"Why custom auth when OAuth libraries exist?"* These are the decisions that get re-litigated most.
- **Keeps resurfacing in discussions.** If you've explained the same choice three times in PR review, write the ADR. It ends the cycle permanently.
- **Trades off something important.** *"Faster build at the cost of test isolation."* The trade-off is the artifact's value.

### Do NOT write an ADR for

- **Variable naming, formatting, code style** — that's `CLAUDE.md` or `.editorconfig`.
- **Decisions inside a single feature** — that's `spec.md`.
- **Things obvious from the stack** — *"we use async/await"* — wasted ceremony.
- **Reversible conventions** — if changing your mind would take an afternoon, it's a convention, not an ADR.

**The practical test:** *"If a year from now someone asks 'why the hell did we do it this way?' — that's an ADR."*

If the answer would be *"because we always did"* or *"someone decided in a Slack thread"*, you needed an ADR you don't have.

---

## The Nygard format, section by section

The canonical sections, in order. Each has a specific job; cutting one breaks the artifact.

### `# ADR-NNN: <Short title>`

The title is a **noun phrase**, not a question or a verb. *"Dapper as the Data Access Layer"*, not *"Should we use EF or Dapper?"* and not *"Use Dapper instead of EF."* The decision is the headline; the deliberation lives below.

`NNN` is sequential from 001, zero-padded for sort order. Don't skip numbers; don't reuse.

### `## Status`

One of: `Proposed`, `Accepted`, `Deprecated`, `Superseded by ADR-NNN`. Include the date. For `Superseded`, the link to the replacement is mandatory.

```
## Status
Accepted — 2026-05-20
```

### `## Context`

Why this decision was necessary. The *forces* that pushed the choice — technical, business, social. 3–8 sentences usually. The future reader uses this section to verify whether your forces still apply.

What belongs here:
- The system constraint that made the decision necessary
- Constraints from other (already-decided) ADRs that bound the option space
- Quantitative considerations (e.g., *"10k+ records per batch"*)
- Team / org constraints (e.g., *"team knows T-SQL, not LINQ"*)

What does NOT belong: the decision itself. Save that for `Decision`.

### `## Decision`

What you chose. Should be **short — 3 to 5 sentences**. Not a tutorial; not an explanation of how the chosen tool works. Just the choice, plus enough constraint that the agent (and the future engineer) can act on it.

```
## Decision
We use Dapper as the sole data access layer. Repositories are
hand-written, one class per aggregate. SQL lives in constants in
the repo class, not in `.sql` files.
```

If you find yourself writing more than ~5 sentences here, you're explaining; explanation lives in `Context` (why) or `Consequences` (what happens).

### `## Consequences`

What follows from the decision. Both positive *and* negative — an ADR with only positives is propaganda, not documentation.

Split into **Positive / Negative / Neutral**:

- **Positive** — the benefits you got
- **Negative** — what you gave up; what becomes harder
- **Neutral** — side effects that aren't clearly good or bad; constraints they introduce on later decisions

The Negative section is the one future readers value most. *"We chose X; here's what X cost us"* tells them whether to keep paying that cost.

### `## Alternatives Rejected`

The options you considered and chose not to take. One short bullet per alternative: name + one-sentence reason.

This is the single most important section for future readers. Without it, the ADR reads as *"we did X because we did X."* With it, the reader can verify whether the rejected alternatives still deserve rejection — and, if not, write the supersede.

### `## References` (optional but useful)

Spike PRs, comparison documents, meeting notes, links to specs that prompted the decision. Anything that would let a future reader trace the evidence.

---

## Alternative formats

### MADR (Markdown ADR)

A trimmed template — just Status / Context / Decision / Consequences. For smaller decisions where Alternatives Rejected and References would feel like ceremony. Closer to how most teams actually work day-to-day.

Format: same shape, fewer required sections. Document the trim in `CLAUDE.md` so the team is consistent.

### Y-statements

A one-paragraph compressed form:

> *"In the context of `<scenario>`, facing `<concern>`, we decided for `<option>` and against `<rejected option>`, to achieve `<benefit>`, accepting `<downside>`."*

Y-statements work for very low-stakes decisions, or as a way to draft an ADR fast and expand it later. Bad for any decision worth real archaeology — too compressed to survive.

### Custom hybrid

Many mature teams write their own variant: standard sections plus a `Migration` section for decisions that supersede prior ones, plus a `Validation` section for how they'll know the decision was right. Document the variant in `CLAUDE.md` or a `docs/adr/_template.md`.

The format matters less than the *consistency*. Pick one; use it everywhere.

---

## Status lifecycle in depth

Four standard statuses. Real ADRs move through them in a few common patterns.

| From | To | Trigger |
|------|----|---------|
| (none) | `Proposed` | Someone drafts the ADR for review |
| `Proposed` | `Accepted` | Team agrees; status updated, date set |
| `Proposed` | (deleted/abandoned) | Discussion concludes the proposal isn't a real decision |
| `Accepted` | `Superseded by ADR-NNN` | A new ADR replaces this one |
| `Accepted` | `Deprecated` | We no longer recommend this, but haven't replaced it |

### `Proposed`

The draft state. The decision hasn't been ratified; the ADR is up for review. Anyone can comment / propose changes to the *content* of the ADR (Context, Decision, Alternatives) while it's `Proposed`. Once it moves to `Accepted`, the content freezes.

### `Accepted`

The decision is in force. From here on, only the **Status header** may change — never the body. If facts change, write a new ADR that supersedes.

### `Deprecated`

The decision is no longer recommended, but there's no replacement yet. Used when:
- You know an approach has problems but you haven't decided what's next
- You're winding down an old pattern but new ADRs haven't crystallized

`Deprecated` is the rarest state. Most decisions move directly from `Accepted` to `Superseded` (because writing the supersede is part of how you make the change).

### `Superseded by ADR-NNN`

A newer ADR replaced this one. The body of the original stays unchanged — only the header updates. The newer ADR's Context section explains *what changed*.

---

## The Supersedes pattern in depth

**The key rule:** you never edit the body of an Accepted ADR. You change only the status header, and you write a new ADR with the new decision.

This is what keeps the history reconstructible. Someone reading the repo in 2030 can trace the decision lineage by following Supersedes links.

### Single supersede

The basic case: ADR-014 replaces ADR-002.

```markdown
# ADR-014: Quartz Configuration from Database
## Status
Accepted — 2026-05-20
Supersedes: ADR-002
```

And in ADR-002:

```markdown
# ADR-002: Quartz with Configuration in appsettings.json
## Status
Superseded by ADR-014 — 2026-05-20
## Context
[unchanged — history must stay]
...
```

Two header changes total; no body edits.

### Supersede chains

When ADR-014 itself gets superseded later by ADR-031:

```
ADR-002 (2025-03)  →  ADR-014 (2026-05)  →  ADR-031 (2027-Q2)
   Superseded         Superseded            Accepted
   by ADR-014         by ADR-031
```

Each step has its own ADR; each Status header points at the immediate replacement. Don't try to "skip" — ADR-014 stays in the chain even after ADR-031 obsoletes it.

### Partial supersedes

Sometimes a new ADR replaces *some* of an old one but not all. Example: ADR-007 said *"Dapper for data access, repositories hand-written, no Dapper.Contrib."* ADR-019 says *"OK to use Dapper.Contrib for trivial CRUD."*

Two acceptable patterns:

1. **Supersedes ADR-007 partially.** Status: `Supersedes ADR-007 (Dapper.Contrib clause only)`. Body of ADR-019 explains scope. ADR-007 stays largely in force; only the Dapper.Contrib bullet is overridden.
2. **Extends ADR-007.** Status: `Extends ADR-007`. Body of ADR-019 adds the new permissive rule without contradicting the core decision.

Either works; pick the one that reads more naturally. Document the convention in `CLAUDE.md` if you use both.

---

## Worked examples

### Example 1: A fresh Accepted ADR

```markdown
# ADR-007: Dapper as the Data Access Layer

## Status
Accepted — 2026-01-15

## Context
The system processes orders with HDR/DTL files, where critical needs include:
- batch query performance (10k+ records per order)
- full control over SQL (`legacy_import` schema with many joins)
- debugging simplicity — team knows T-SQL better than LINQ

EF Core would be the natural choice for a new .NET 8 project, but:
- EF migrations don't align with our existing DDL-diff process
- LINQ-to-SQL produces unpredictable plans on complex queries
- our repositories would write raw SQL via `FromSqlRaw` anyway

## Decision
We use Dapper as the sole data access layer. Repositories are hand-written,
one class per aggregate (`OrderImportRepository`, `OrderLogRepository`).
SQL lives in `const string` at the top of the repo class, not in `.sql` files.

## Consequences

**Positive:**
- Full control over generated SQL
- Easier code review (you see exactly what hits the database)
- No hidden N+1
- Smaller dependency graph

**Negative:**
- Manual DTO ↔ record mapping (boilerplate)
- No change tracking — updates require explicit SQL
- Integration tests matter more than with EF (less "free" type validation)

**Neutral:**
- DDL migrations handled by separate tooling (DbUp or custom scripts)
- `Dapper.Contrib` only for simple CRUD, never for joins

## Alternatives Rejected

- **EF Core 8** — see Context
- **NHibernate** — abandoned by community; younger devs don't know it
- **Linq2Db** — interesting, but smaller community and less familiar to the team

## References

- Spike: PR #87 (EF vs Dapper comparison on `order_log` query)
- Discussion: architecture meeting notes 2026-01-10
```

A few things to notice:

- Decision is **3 sentences**. No tutorial on how Dapper works.
- Context explains *why this team specifically* — generic forces would be a red flag.
- Consequences includes negatives by name. *"Manual DTO mapping is boilerplate"* — that's honest.
- Alternatives Rejected names each option; the future reader can verify they still deserve rejection.

### Example 2: A Superseded ADR pair

**The original (ADR-002) after supersede — header changed only:**

```markdown
# ADR-002: Quartz.NET with Configuration in appsettings.json

## Status
Superseded by ADR-014 — 2026-05-20

## Context
[unchanged — original 2025-03-10 content stays intact]
We need a scheduling system for batch jobs. Quartz.NET is the .NET standard;
config in appsettings.json is the natural fit because...

## Decision
[unchanged — the original decision text]

## Consequences
[unchanged]

## Alternatives Rejected
[unchanged]
```

Notice: only the Status header changed. The body is exactly as written in 2025-03-10. Someone reading the repo in 2030 sees both the original decision and the fact that it was later replaced.

**The replacement (ADR-014):**

```markdown
# ADR-014: Quartz Configuration from Database Instead of appsettings

## Status
Accepted — 2026-05-20
Supersedes: ADR-002

## Context
ADR-002 (2025-03-10) established Quartz configuration in appsettings.json.
Since then new requirements emerged:
- Different schedules per environment (DEV / TEST / PROD)
- Ability to change cron expressions without a deployment
- Audit trail of schedule changes for compliance

The original rationale in ADR-002 doesn't address these — config flat files
can't be changed at runtime, and they aren't auditable.

## Decision
Quartz job definitions live in `app.quartz_jobs` (DB table). The application
reads from the table at startup and on a 60-second poll. Schema migrations
for the table follow our standard DDL-diff process.

## Consequences

**Positive:**
- Per-environment schedules without separate config files
- Cron changes without deployment
- Audit trail via the table's `updated_at` and `updated_by` columns

**Negative:**
- One more table to operate (backups, migrations)
- 60-second polling adds startup-time latency on schedule changes
- Local dev requires a populated table (handled via seed script)

**Neutral:**
- ADR-002 stays as historical context; new jobs go to the DB; existing
  appsettings-based jobs migrate in spec `2026-Q3-quartz-db-migration`

## Alternatives Rejected
- **Keep appsettings.json** — see Context; doesn't meet new requirements
- **Per-environment config files** — solves env split, not runtime change
- **Feature flag service** — overkill for cron strings; we don't have one

## Migration
- Existing jobs stay on appsettings until end of Q3 2026
- New jobs go to the DB table immediately
- Full migration tracked by spec `specs/2026-Q3-quartz-db-migration/`

## References
- Spike: PR #142 (DB-backed config prototype)
- Original: ADR-002
```

Notice the `Migration` section — useful when the supersede has phased rollout. Treat it as optional; include when relevant.

### Example 3: A Proposed ADR under discussion

A draft that hasn't been accepted yet. Visible to the team for comments; subject to change.

```markdown
# ADR-022: Switch from Synchronous SFTP to Event-Driven File Delivery

## Status
Proposed — 2026-08-12

## Context
[Drafted by Alice. Subject to revision until Accepted.]

Current setup (per ADR-005): the scheduler polls partner SFTPs every 5
minutes; partner-side late files cause our SLA to slip by up to 5 minutes.
Adding more frequent polling increases load on partner systems and is
explicitly forbidden by some partner contracts.

An event-driven model — partner pushes notifications to our endpoint when
a file is ready — would close the gap.

## Decision
[DRAFT] We add a webhook endpoint `/api/v1/file-ready/{partnerId}` that
partners call to signal a file is available. The scheduler retains the
polling fallback for partners without webhook support.

## Consequences

**Positive:**
- SLA gap closed for partners on webhook
- Reduced load on partner SFTPs (we poll less)

**Negative:**
- New public endpoint = new attack surface; needs HMAC auth per partner
- Existing partner contracts need amendment for webhook support
- Some partners can't host outbound calls (firewall) — they stay on polling

**Open questions (resolve before Accept):**
- How do we sign webhook calls? HMAC vs mTLS?
- What's the retry policy on partner side? Do we expose idempotency keys?
- Does the polling fallback's interval change?

## Alternatives Rejected
- **More frequent polling** — partner contracts forbid; load
- **Partner pushes via existing SFTP "trigger file"** — half the partners don't support it
- **Long polling** — operationally complex; webhooks are simpler

## References
- Slack thread #ops-platform 2026-08-08 (kicking off discussion)
```

The `Open questions` block is the tell that this is `Proposed` — once those are answered, they collapse into the Decision and Consequences, the `Open questions` block disappears, and Status moves to Accepted.

### Example 4: A Deprecated ADR with no replacement

Rare but useful when you know an approach has problems but haven't decided what's next.

```markdown
# ADR-011: Synchronous Webhook Retries with Exponential Backoff

## Status
Deprecated — 2026-09-04

## Context
[Original 2025-08 content — unchanged]
...

## Decision
[Original — unchanged]

## Consequences
[Original — unchanged]

## Deprecation note (2026-09-04)
This approach causes head-of-line blocking when one slow partner saturates
the worker pool. We've seen this twice in production. A queue-based async
model would be better, but we haven't committed to a specific queue
technology yet (Redis Streams? RabbitMQ? Postgres LISTEN/NOTIFY?).

For now: do NOT add new partners to this pattern. Existing partners stay
until a replacement ADR is written. Tracking in `specs/2026-Q4-webhook-async-discovery`.
```

Notice: the body still isn't edited. A *new section* (`Deprecation note`) is appended at the bottom, with its own date. The original Decision and Consequences stay intact.

---

## Numbering, naming, file organization

### Numbering

- Sequential from `001`, zero-padded to 3 digits (`ADR-007`, not `ADR-7`).
- **No gaps.** If a `Proposed` ADR is abandoned, leave the number assigned; just don't write the ADR. Renumbering breaks every external reference.
- **No sub-numbers.** No `ADR-002.1`, no `ADR-002-v2`. Decision changes → new top-level number with `Supersedes`.

### Filenames

`ADR-NNN-<short-slug>.md` — slug is kebab-case, ideally 2–4 words.

Good: `ADR-007-dapper-data-layer.md`, `ADR-014-quartz-db-config.md`
Bad: `ADR-007-the-decision-about-using-dapper-instead-of-entity-framework-core.md`
Worse: `ADR-007.md` (no slug — hard to navigate)

### Directory

Always `docs/adr/` (or `adrs/` — pick one and stick with it). The folder also holds:

- `_template.md` — your team's canonical ADR template (Nygard, MADR, or custom)
- `README.md` (optional but recommended) — index of ADRs by status: Accepted, Proposed, Deprecated, Superseded

Don't shard ADRs into subfolders by topic. Sequential numbering + an index gives better discoverability than a topical hierarchy that requires guessing where things live.

---

## Cross-referencing ADRs

ADRs are most valuable when they're *findable* from where the decision shows up.

### From code

A short comment with the ADR number, at the top of the affected file or method:

```csharp
// ADR-007: Dapper-only data access. No EF here.
public sealed class OrderRepository
{
    ...
}
```

The agent (and humans) can grep for `ADR-` to find every reference. Avoid prose explanations of *why* — that's what the ADR is for; just link.

### From PRs

In the PR description: *"Implements `specs/2026-05-x/`. Follows ADR-007 (Dapper)."* The PR has a permanent trail back to the decision.

### From other ADRs

When ADR-019 builds on ADR-007 without superseding, reference it in Context: *"Per ADR-007, data access goes through hand-written repositories. This ADR extends that…"*

### From `CLAUDE.md`

A short "Active decisions" list, by number with one-line summaries:

```markdown
## Active decisions (Accepted ADRs)
- ADR-001 — Repository per aggregate
- ADR-007 — Dapper for data access (not EF)
- ADR-014 — Quartz config in DB (supersedes ADR-003)
```

Do **not** list Superseded or Deprecated ADRs here — the agent should not treat them as authoritative.

---

## The ADR review process

An ADR moves from `Proposed` to `Accepted` through some form of review. Two patterns work well; pick one.

### Pattern A: PR-based review

The ADR is committed as `Proposed` in a PR. Reviewers comment inline; the author iterates. When the team agrees, a separate small PR flips Status to `Accepted` and adds the date.

Pros: standard tooling, async, leaves a trail.
Cons: can drag on if no one drives the merge.

### Pattern B: Meeting-driven

The ADR is drafted in a doc, discussed at the next architecture meeting, then committed as `Accepted` after the meeting.

Pros: faster decisions; everyone hears the same context.
Cons: no async record of the discussion; remote-unfriendly.

Many teams combine: PR-based for routine ADRs, meeting-driven for ADRs touching multiple teams.

### What reviewers check

A working ADR review checklist:

- [ ] **Context** explains *why* this decision is necessary, not just what
- [ ] **Decision** is short (3–5 sentences)
- [ ] **Alternatives Rejected** lists realistic alternatives with one-sentence reasons
- [ ] **Consequences** includes Negatives by name
- [ ] No references to `Status: Superseded` ADRs as current authority
- [ ] No overlap with another active ADR — if there is, supersede it explicitly
- [ ] If retroactive, the Context says so honestly

If the ADR fails any of those, comment and ask the author to revise before Accept.

---

## Agent-assisted drafting

The agent can do most of the typing; you do the judging. Three reusable prompts.

### Draft a new ADR from a decision summary

**Prompt:**

```text
I just decided to use [X] instead of [Y] for [reason].

Draft an ADR following docs/adr/_template.md.
Use the next available number (scan docs/adr/ for the highest).
Status: Proposed, today's date.

Fill in Context, Decision, Consequences (positive/negative/neutral),
Alternatives Rejected. Leave References empty — I'll add them.

Show me the draft before saving. Anything you're unsure about,
mark [VERIFY].
```

### Draft a Supersedes ADR

**Prompt:**

```text
ADR-NNN currently says [old approach]. We're changing to [new approach]
because [reason].

Draft a new ADR that supersedes ADR-NNN. Use the next available number.
Status: Accepted, today's date.

Include a Migration section if relevant (when does the new ADR take effect;
what happens to code following the old one).

Don't edit ADR-NNN yet. I'll change its Status header after this draft is final.
```

### Find implicit decisions worth an ADR

**Prompt:**

```text
Scan src/ for code that looks like a deliberate decision (custom code
where a standard library exists, workarounds with explanatory comments,
unusual patterns).

For each candidate:
1. Describe what was decided (one sentence)
2. Best guess at why (cite comments / commit messages)
3. Mark whether this is worth a new ADR

Return a list. Don't draft the ADRs — I'll pick which to formalize.
```

(Also useful as a configured subagent — see [`working-with-agents-guide.md` § Subagents](working-with-agents-guide.md#subagents).)

---

## Anti-patterns

What goes wrong, and how to fix it.

### 1. ADR as post-mortem documentation

You write the ADR six months after the decision *"because we should document it."* The Context section is reconstructed; the Alternatives Rejected list is best-guess; the Consequences omit things that turned out badly.

**Fix:** write the ADR *at the moment of decision*. If you must write retroactively (during migration to SDD, for example), mark it honestly: *"This ADR documents a decision made ~2022; written 2026 from team recollection and code archaeology."*

### 2. ADR as a manual

The Decision section runs 30 lines, explaining how the chosen tool works.

**Fix:** Decision = 3–5 sentences. *Choice + immediate constraints*. Explanation goes elsewhere (Context for why, Consequences for what follows, external docs for tutorials).

### 3. Missing "Alternatives Rejected"

The ADR reads as *"we did X because we did X."*

**Fix:** every ADR has at least 2 alternatives listed. If only one alternative existed, it wasn't really a decision worth recording.

### 4. Editing old ADRs

*(To mechanically prevent this — block edits to bodies of ADRs with `Status: Accepted` via a pre-commit hook or Claude Code `PreToolUse` hook — see [`quality-gates-guide.md`](quality-gates-guide.md) § "Pattern A" and "Pattern B".)*



You notice ADR-002 has a typo, or its Context is now inaccurate, and you edit the body.

**Fix:** body of `Accepted` ADRs is immutable. Fix typos in `Proposed` ADRs only. For factual updates, write a Supersedes. The point of the rule isn't pedantry — it's that the history must be reconstructible.

### 5. Numbering with gaps or sub-versions

`ADR-002.1`, `ADR-007-v2`, ADR-005 skipped because *"that one wasn't ready yet."*

**Fix:** strictly sequential; gaps allowed only for ADRs that were Proposed and never reached Accepted (and even then, keep the number assigned — don't reuse).

### 6. ADR for reversible things

*"We use camelCase for JSON keys."* That's a convention. ADRs are for things you'd burn engineering weeks reversing.

**Fix:** apply the "year from now" test. If reversing the decision in a year would take a sprint, ADR. If it would take an afternoon, `CLAUDE.md` or `.editorconfig`.

### 7. ADR sprawl

After 6 months you have 40 ADRs. Half are obsolete; half overlap. Nobody references any of them.

**Fix:** quarterly audit — re-read the Active list; supersede the stale; trim the never-needed-an-ADR ones (they're conventions). Aim for ~10–20 active ADRs in a mid-size system; fewer in smaller systems.

### 8. Status `Superseded` without the link

Status: `Superseded`. No ADR number, no date.

**Fix:** *always* `Superseded by ADR-NNN — YYYY-MM-DD`. Without the link, the chain breaks.

### 9. Conflicting active ADRs

ADR-007 says use Dapper. ADR-019 (also `Accepted`) says use EF Core for new modules. Neither references the other.

**Fix:** one of them must supersede the other (fully or partially). Two `Accepted` ADRs on the same topic = a bug in the discipline.

### 10. ADR written by the agent unsupervised

The agent generated a perfectly-formatted ADR. The Context section is plausible-sounding but invented; the Alternatives are generic.

**Fix:** agent drafts, human verifies. Especially: verify Context (do these forces actually exist?) and Alternatives (were these real options?). ADRs are decisions you own.

---

## Edge cases

### Retroactive ADRs (during SDD migration)

Sometimes you need to document a decision that was made years ago. The ADR is honest about this:

```markdown
## Context
This ADR documents a decision originally made circa 2022, written down
in 2026 because the AI agent has repeatedly proposed [the rejected
alternative]. Team recollection and code archaeology are the sources;
some detail is necessarily reconstructed.
```

Treat retroactive ADRs as exceptions, not the default. See [`legacy-to-sdd-migration-guide.md` § Phase 1 Day 5](legacy-to-sdd-migration-guide.md#day-5-first-adr).

### Sub-ADRs / ADR groups

When a single big decision has multiple related sub-decisions (e.g., adopting a microservices architecture might involve 6 connected sub-ADRs), two patterns:

1. **Group of independent ADRs.** Write each as a top-level ADR; cross-reference each other in Context. Don't fake hierarchy.
2. **One big ADR with sub-sections.** Acceptable for tightly-coupled sub-decisions. Keep it under ~3 pages or split.

Avoid: numbered sub-ADRs (`ADR-002.1`). They break sort order and cross-references.

### Format migrations

You've been writing Nygard-format ADRs and want to switch to MADR (or vice versa).

Don't rewrite the old ones. New ADRs use the new format; old stay as written. Note the change in your `_template.md` and `CLAUDE.md`. The cross-format inconsistency is mild; rewriting the past is worse.

### ADRs in monorepos

Per-service ADRs OR a global ADR folder. Pick by ownership:

- **Per-service** (`services/orders/docs/adr/`) — service team owns local decisions.
- **Global** (`docs/adr/`) — when most decisions affect multiple services.

Hybrid is common: global for cross-cutting (logging, auth, observability), per-service for the rest. Document which is which in root `CLAUDE.md`.

---

## Tooling

In order of how much overhead they add:

- **None.** A `docs/adr/` folder plus discipline. Works fine up to ~30 ADRs. Most teams never need more.
- **[adr-tools](https://github.com/npryce/adr-tools)** (Node/Bash) — generates numbered skeletons: `adr new "Database choice"`. Minimal; recommended once your team forgets numbers manually.
- **[log4brains](https://github.com/thomvaill/log4brains)** — generates a static site from your ADR markdown. Nice for browsing; overkill until 30+ ADRs.
- **MADR template** — just the template, no tooling. Good middle ground.
- **Custom scripts** — most large teams end up with a 20-line shell script: `new-adr.sh "title"` finds the highest number, copies the template, opens the editor.

**Add tooling reactively.** If you're in `docs/adr/` and you forget the next number — that's the trigger to add `adr-tools` or a script. Before then, the friction of installation outweighs the friction of typing the number.

---

## Maintenance discipline

ADRs rot if you don't audit them. A quarterly ritual that takes ~30 minutes for most teams:

1. **List all ADRs by status.** Most easily by running `grep -l '^## Status' docs/adr/*.md` plus a `grep '^Accepted'` pipeline.
2. **For each `Accepted` ADR**, ask: *do the forces in Context still hold?* If not, candidate for supersede.
3. **For each `Proposed` ADR older than 1 month**, ask: *is this stalled or about to land?* Stalled `Proposed` ADRs rot the catalog.
4. **For each `Deprecated` ADR**, ask: *is there a replacement now?* If yes, write the supersede.
5. **Update CLAUDE.md's Active list** to match.

After the quarterly audit, the Active list in CLAUDE.md should reflect reality. If it doesn't, the ADRs aren't earning their place.

### Ownership

ADRs need a maintainer. Usually the engineer who runs the architecture-review cadence; in smaller teams, the most senior engineer. Their job is *not* to write every ADR — it's to make sure the Active list stays accurate and stale ADRs get superseded promptly.

---

## Golden rules

1. **An ADR is a decision, not a design.** One decision per ADR; keep the Decision section short.
2. **Immutable from `Accepted`.** Body of an Accepted ADR is never edited. Supersede instead.
3. **Alternatives Rejected is the most valuable section.** Without it, future readers can't tell whether the decision still deserves to stand.
4. **Honest about retroactivity.** A retroactive ADR says so in Context. Pretending it was written contemporaneously is fabrication.
5. **Numbering is sacred.** Sequential, zero-padded, no gaps, no sub-versions.
6. **Cross-reference everywhere.** ADRs are most valuable when findable from code, PRs, other docs.
7. **The agent drafts; you verify.** Especially Context and Alternatives — those are the parts the agent invents most plausibly.
8. **Active list = source of truth.** `CLAUDE.md` lists only `Accepted` ADRs. Superseded and Deprecated ADRs stay in `docs/adr/` for history but never as current authority.
9. **Quarterly audit or it rots.** 30 minutes a quarter beats a wholesale cleanup once a year.
10. **A short ADR written today beats a perfect one written never.** Your first five ADRs will be imperfect. By the tenth you'll have developed a style that fits how you work.

---

*This guide complements [`spec-driven-development-guide.md`](spec-driven-development-guide.md) (the layered SDD model — PRD, specs, ADRs, conventions), [`claude-md-guide.md`](claude-md-guide.md) (how to maintain the Active ADR list in `CLAUDE.md`), and [`legacy-to-sdd-migration-guide.md`](legacy-to-sdd-migration-guide.md) (how to introduce ADRs into a codebase that's never had them).*
