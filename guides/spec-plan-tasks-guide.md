# The Feature Trio: spec → plan → tasks

> **This is the core loop of spec-driven development — the one guide to read if you read only one.** Three short markdown files, written in order: `spec.md` (what & why) → `plan.md` (how) → `tasks.md` (in what order). Each one locks down what the next needs. That's the whole method. No toolkit, no scaffolder, no wall of generated files — the discipline is the *order*, not the tooling. Everything else in this repo exists to support this loop.

---

## Table of Contents

*Sorted as **what → why → how → examples**.*

**What**
1. [What this guide is](#what-this-guide-is)
2. [The trio as a flow](#the-trio-as-a-flow)
3. [What sections each file needs](#what-sections-each-file-needs)

**Why / when**
4. [When to skip parts of the trio](#when-to-skip-parts-of-the-trio)

**How**
5. [How the three documents reference each other](#how-the-three-documents-reference-each-other)
6. [AI-assisted authoring: prompts per artifact](#ai-assisted-authoring-prompts-per-artifact)
7. [Iteration patterns — sharpening a draft](#iteration-patterns--sharpening-a-draft)
8. [Cross-artifact consistency checks](#cross-artifact-consistency-checks)
9. [Cross-trio anti-patterns](#cross-trio-anti-patterns)
10. [Slash commands worth having](#slash-commands-worth-having)

**Examples**
11. [Worked example 1 — Rate limiting on the orders endpoint](#worked-example-1--rate-limiting-on-the-orders-endpoint)
12. [Worked example 3 — the whole trio in one file](#worked-example-3--the-whole-trio-in-one-file)

13. [Golden rules for trio authoring](#golden-rules-for-trio-authoring)

---

## What this guide is

This is the definitive, hands-on treatment of the trio — the centerpiece of the whole repo. The [overview guide](spec-driven-development-guide.md) sketches the principles as part of the bigger map (`CLAUDE.md`, ADRs, PRDs, the documentation layers); *this* is where you actually learn to write the three documents that drive every change.

What you get here:

- **Two complete worked examples** — all three filled-in documents *for the same feature*, with the cross-references explicit, ready to copy as a starting point
- **AI-assisted authoring prompts** — copy-pasteable prompts for drafting each artifact, reviewing it, refining it, and validating it against the others
- **Iteration patterns** — how a rough draft becomes a tight spec, then a clear plan, then an executable task list
- **Cross-artifact consistency checks** — concrete rules that keep the three documents honest (every acceptance criterion has a task; every "out of scope" survives into the plan; every open question resolves into a decision or an ADR)

The method is deliberately small. You don't install anything or generate a dozen scaffolded files — you write three markdown documents in order, and you keep them honest. The rest of this guide is how to do that well.

---

## The trio as a flow

The three documents are a sequence, not a set. Each one *locks down* something the next one needs:

```
[vague idea]
   ↓  (the spec locks down: WHY, WHAT, and HOW WE'LL KNOW IT WORKS)
spec.md
   ↓  (the plan locks down: HOW we'll build it, in which files)
plan.md
   ↓  (tasks locks down: in WHAT ORDER, with VERIFICATION per step)
tasks.md
   ↓
[code]
```

*(Where the **feature** itself comes from — slicing a PRD into features — is one step upstream of this guide: see [`prd-guide.md`](prd-guide.md) § "Slicing the PRD into features".)*

Each step narrows the option space. Once `spec.md` is accepted, the team has agreed on *what success means*. Once `plan.md` is accepted, the team has agreed on *the shape of the solution*. Once `tasks.md` is set, the engineer (and the agent) just executes.

**The order matters because it forces decisions before code.** Writing tasks before plan forces premature ordering assumptions. Writing plan before spec forces premature architecture decisions. Writing spec after code is fiction.

For tiny changes, the trio compresses (see [When to skip parts of the trio](#when-to-skip-parts-of-the-trio)). For most non-trivial work, all three earn their place.

And when code *won't* come cleanly out of a finished trio? That's almost always a gap in the spec or plan — not a cue to abandon them and vibe code. See [`working-with-agents-guide.md` § "When the Agent Can't Build from the Spec"](working-with-agents-guide.md#when-the-agent-cant-build-from-the-spec).

---

## What sections each file needs

Each section in the trio earns its place — none are decoration. Below: the **bare minimum** each file must have (and *why* the agent or a reviewer breaks without it), then the **nice-to-haves** worth adding as a change grows. Start minimal; add a section the moment its absence costs you something.

### `spec.md` — the *what & why*

**Bare minimum**

| Section | Why it's required |
|---------|-------------------|
| **Goal** | *What problem, for whom*, in one paragraph. Without it the agent optimises for the wrong thing and a reviewer can't tell done from not-done. |
| **Acceptance criteria** | The testable definition of done — these become your tests. A spec without them can't be verified, only argued about. |
| **Out of scope** | The highest-leverage section: *"what we're NOT doing"* stops the agent (and scope creep) from wandering, better than any positive instruction. |

**Nice to have** (add as the change grows)

| Section | Why it helps |
|---------|--------------|
| **In scope** | Makes the boundary explicit when the Goal alone is ambiguous. |
| **Impact on existing code** | Lists the files/contracts touched — catches surprises before code and tells the agent where to look. |
| **Open questions** | Surfaces what's undecided so the agent doesn't silently guess (and you don't find the guess in the diff). |
| **References** | ADRs, prior specs, integration docs — the lineage that keeps decisions consistent. |

### `plan.md` — the *how*

**Bare minimum**

| Section | Why it's required |
|---------|-------------------|
| **Technical decisions** | The stack/pattern choices, each tied to an ADR. Without it the agent picks defaults that may fight your architecture. |
| **File structure** | Concrete paths to create/modify — what stops the agent inventing a layout, and what `tasks.md` and the consistency check key off. |
| **Constraints** | The explicit *"do NOT"* list — the known failure modes the agent would otherwise walk into. |

**Nice to have**

| Section | Why it helps |
|---------|--------------|
| **Data model** | Needed *only* when persistent state is involved — DDL/entities the agent must get exactly right. |
| **Open questions** | For *how*-level uncertainties left after the spec's are resolved. |
| **References** | The spec, the relevant ADRs, the existing pattern file to mirror. |

### `tasks.md` — the *in what order*

**Bare minimum**

| Section | Why it's required |
|---------|-------------------|
| **Implementation (in execution order)** | The ordered steps, each ending in ` → verify:`. The order *is* the point, and a step with no verification can't tell you (or the agent) when it's done. |
| **Verification (against acceptance criteria)** | Maps every acceptance criterion to the task(s) that prove it — the gate that says "ready to merge." |

**Nice to have**

| Section | Why it helps |
|---------|--------------|
| **Setup** | Branch, migration, env — the prerequisites before step 1. |
| **Post-merge** | The follow-ups that always get forgotten: flip the spec status, update the runbook, close the ticket. |
| **Notes (append as you work)** | The running log of gotchas and mid-build decisions — the messy reality the worked examples show. Cheap insurance against *"why did we do that?"* |

The bare-minimum rows are the sections you'll see filled in across the [worked examples](#worked-example-1--rate-limiting-on-the-orders-endpoint) below; the nice-to-haves are what a multi-module feature accumulates and a bug-fix never needs.

---

## When to skip parts of the trio

The trio is the default for any real change. The main lever isn't *dropping* spec/plan/tasks — it's how many **files** you spread them across; only for the very smallest changes do you drop documents entirely. Match the weight to the size:

| Change shape | How to write it |
|--------------|-----------------|
| One-line doc / config tweak | A PR description. No spec. |
| Bug fix (one file, one commit) | A short `spec.md` — goal + acceptance criteria. Plan and tasks are implicit. |
| Small feature (1–3 files, ~half a day) | The one-file trio — spec / plan / tasks as three sections in a single file ([Worked example 3](#worked-example-3--the-whole-trio-in-one-file)). |
| Non-trivial feature (multiple modules, multi-day) | The full three-file trio ([Worked example 1](#worked-example-1--rate-limiting-on-the-orders-endpoint)). |
| Refactor (no behavior change, larger surface) | Short spec + full plan + tasks — the *how* and *order* matter more than the *what*. |
| Spike / research | A spec heavy on Open Questions; plan and tasks wait until the spike resolves. |

The rule: **if you'd struggle to explain the change in a one-paragraph PR description, write a spec.** If the execution order would change had you been interrupted for a week, write the tasks. Reach for three *separate* files only when the plan or tasks get long, or when more than one person edits them at once — otherwise the one-file trio keeps everything in view.

When in doubt, write the shorter form. You can always promote a one-file trio into three files later (nothing is rewritten, only relocated); you can't recover the time spent ceremony-ing a tiny change.

---

## How the three documents reference each other

The trio is most useful when each document *visibly* refers to the others — you'll see it throughout [the worked examples](#worked-example-1--rate-limiting-on-the-orders-endpoint) below. The pattern, made explicit (snippets are from Worked example 1):

**From `plan.md` back to `spec.md`:**
- *"per spec § Open Q1 resolution"* — the plan explicitly cites where the spec decided something
- File structure matches the *"Impact on existing code"* section of `spec.md` — same files listed, no surprises
- Constraints in `plan.md` (*"Do NOT add Redis"*) restate constraints from `spec.md` § Out of scope

**From `tasks.md` back to `spec.md`:**
- The "Verification" section explicitly maps each AC to a task — *"AC2 → covered by AC2 test (step 9)"*
- Post-merge actions reference docs the spec mentioned (*"runbook"*, *"CLAUDE.md"*)

**From `tasks.md` back to `plan.md`:**
- Tasks follow the file structure in `plan.md` — no task touches files outside the listed paths
- Each task uses the patterns the plan settled on (Options binding, middleware pattern from `AuthMiddleware`)

**The benefit of these explicit cross-refs:**

- The agent reading the trio can trace any decision back to its source
- A reviewer can verify completeness — does every AC have a task? does every plan decision trace to the spec?
- Future-you reads `tasks.md`, sees *"AC4 already enforced by AuthMiddleware"*, and remembers *why* without grepping the codebase

A trio without cross-references is three documents that happen to share a folder. With cross-references, it's a coherent decision chain.

---

## AI-assisted authoring: prompts per artifact

The agent does most of the typing; you do the judging. Six reusable prompts, one per common authoring step.

### 1. Draft `spec.md` from a one-paragraph idea

Use when you have a rough idea and want to turn it into a structured spec.

**Prompt:**

```text
We want a new feature: [one-paragraph description of the idea].

Draft specs/YYYY-MM-feature-slug/spec.md using templates/spec.md as the structure.

Fill in: Goal (1-3 sentences), In scope, Out of scope (deliberately, not now),
Acceptance criteria as testable checkboxes, Impact on existing code (specific
file paths after scanning src/), Open questions (don't fill them in yourself —
list what's genuinely undecided), References (related ADRs by number, related
specs by slug, integrations if relevant).

For each acceptance criterion, phrase it so it could become a test name.

Mark any [VERIFY] uncertainty — for example, file paths you guessed.

Show me the draft before saving.
```

The agent produces a starting point in 30 seconds; you spend 5 minutes editing instead of 30 minutes drafting from scratch.

### 2. Review a draft `spec.md` for missing pieces

Use after you've drafted a spec (yours or the agent's) and want a sanity check.

**Prompt:**

```text
Read specs/YYYY-MM-feature-slug/spec.md.

Audit it against this checklist:
1. Is the Goal phrased as a sentence about a problem, not a feature?
2. Does "Out of scope" exclude at least three things a reader might assume?
3. Are acceptance criteria specific enough to write tests from?
4. Does "Impact on existing code" list real file paths (verify against src/)?
5. Are Open Questions genuine uncertainties, or things the author should
   have decided already?
6. Are references current (verify ADR numbers exist in docs/adr/)?
7. Does anything in scope contradict an existing ADR?

Return a numbered list of concerns. Don't modify the file — list the gaps.
```

This catches the "spec looked complete but missed three obvious AC's" failure that wastes a full review cycle.

### 3. Draft `plan.md` from `spec.md`

Once the spec is reviewed and Open Questions are resolved.

**Prompt:**

```text
Read specs/YYYY-MM-feature-slug/spec.md.
Also read CLAUDE.md, ARCHITECTURE.md, and docs/adr/ (active list).

Draft plan.md in the same folder using templates/plan.md as the structure.

Translate the spec into:
- Technical decisions (must align with active ADRs — cite them)
- Data model (if persistent state involved)
- File structure (concrete paths under src/)
- Constraints (carry forward "Out of scope" and any plan-level "do nots")

Open Questions in plan.md should only exist for HOW questions that
remain after the spec's Open Questions are resolved.

Show me the draft. Mark anything you're uncertain about as [VERIFY].
```

### 4. Validate `plan.md` against existing ADRs and `ARCHITECTURE.md`

Run this *before* implementation starts. Catches plan/architecture conflicts early.

**Prompt:**

```text
Read specs/YYYY-MM-feature-slug/plan.md.
Read all Accepted ADRs in docs/adr/.
Read ARCHITECTURE.md.

For each "Technical decision" in the plan:
- Does an existing ADR speak to this? If yes, does the plan match the ADR
  or contradict it? Quote the relevant ADR text.
- Does ARCHITECTURE.md establish a boundary the plan crosses?

For each "File structure" entry:
- Does the path conform to the existing layout in ARCHITECTURE.md?

Return a markdown table: plan decision, ADR/section it relates to, status
(consistent / contradicts / not addressed). Do not modify the plan — list
findings.
```

A plan that quietly contradicts ADR-007 will produce code reviewers hate. Catch it before code.

### 5. Draft `tasks.md` from `plan.md`

After plan review, before implementation.

**Prompt:**

```text
Read specs/YYYY-MM-feature-slug/spec.md and plan.md.

Draft tasks.md in the same folder using templates/tasks.md as the structure.

For each acceptance criterion in spec.md, identify which tasks (one or
more) would prove it. The Verification section should map every AC.

Order tasks so that earlier tasks let you write a failing test (red),
then the implementing tasks make it pass (green). Group related tasks
under headings if the count exceeds ~8.

Each task should end with " → verify: [how to confirm]". A task without
a verification step is incomplete.

Show me the draft. Mark anything you're guessing as [VERIFY].
```

### 6. Trio consistency check

Run before implementation as a final gate.

**Prompt:**

```text
Read all three of: specs/YYYY-MM-feature-slug/spec.md, plan.md, tasks.md.

Verify:
1. Every acceptance criterion in spec.md has at least one task in tasks.md
   (and at least one entry in the Verification section).
2. Every file mentioned in plan.md § File structure has at least one task
   that creates or modifies it.
3. Every "Out of scope" item in spec.md is respected — no task touches it.
4. Every Open Question in spec.md is marked [x] (resolved) or moved to
   an ADR; none are still [ ].
5. plan.md technical decisions don't contradict any Accepted ADR.
6. tasks.md uses only file paths listed in plan.md's File structure.

Return a markdown table: check, status (pass / fail), evidence (cite
specific lines from the trio).

Don't modify any files. Surface only — I'll decide which gaps to fix
before starting implementation.
```

This is your "ready to ship to implementation" gate. Run it before issuing a `/start-implementation` prompt.

---

## Iteration patterns — sharpening a draft

A first-draft trio is almost never the right one. Three loops are common.

### Loop 1: Sharpening the spec

After the initial spec draft, the spec usually has:
- Vague acceptance criteria (*"the endpoint works correctly"*)
- Under-specified Out of Scope
- Open questions that should have been decisions

**Prompt:**

```text
Read specs/YYYY-MM-feature-slug/spec.md.

For each acceptance criterion, sharpen it: if I tried to write a test
from this line, could I? If not, rewrite it so I could. Quote the
existing line, then propose the sharpened replacement.

Also: scan In scope. For each item, propose one thing that might be
assumed but should explicitly land in Out of scope. List candidates,
don't edit.
```

Run this twice if needed. By the second pass, the spec is usually shippable.

### Loop 2: Plan refinement after spec review

If the spec changes (open questions resolved, scope sharpened), the plan needs to follow. **Prompt:**

```text
spec.md changed since plan.md was drafted. Diff between them is roughly:
[paste the spec change].

Re-read both. Identify which Technical decisions, File structure entries,
or Constraints in plan.md need to update to match the new spec. Don't
modify plan.md — list the diffs I should apply.
```

### Loop 3: Tasks granularity review

Before implementation. **Prompt:**

```text
Read specs/YYYY-MM-feature-slug/tasks.md.

For each task in "Implementation":
- Estimate roughly how long it would take to complete and verify (in
  minutes, assuming familiarity with the codebase)
- If > 30 minutes, it's likely too big — propose how to split
- If < 5 minutes, propose combining with a neighboring task
- If the verification step is vague, propose a more specific one

Return a table: task #, estimate, suggestion. Don't modify the file.
```

After these three loops, the trio is usually ready to drive implementation without correction mid-flight.

---

## Cross-artifact consistency checks

The consistency checks I'd add as either subagent calls or `/trio-check` slash command:

1. **AC → task coverage.** Every acceptance criterion in `spec.md` has at least one task that produces evidence for it.
2. **Out-of-scope respect.** Every item in `spec.md` § Out of scope appears nowhere in `plan.md` or `tasks.md`.
3. **Open question resolution.** Every `[ ]` in `spec.md` § Open Questions is either marked `[x]` with resolution noted, OR moved to a new ADR (cited in References).
4. **Plan ↔ ADR consistency.** Every technical decision in `plan.md` is either an existing Accepted ADR, a new ADR, or trivially in-line with existing conventions (no contradictions).
5. **File-path consistency.** Every file modified by a task in `tasks.md` is listed in `plan.md` § File structure; no surprise files.
6. **Cross-document language.** Same vocabulary used across all three (the `spec.md` says *"rate limit breach"*; the `tasks.md` shouldn't suddenly call it *"quota exceeded"*).
7. **Reference integrity.** Every cited ADR number exists in `docs/adr/`; every cited spec slug exists in `specs/`; every cited file path exists in the repo.

A trio that passes these seven checks is ready for implementation. A trio that fails on (1) or (4) usually loops back through review.

For mechanizing these checks — as a configured subagent (`trio-auditor`), a slash command (`/trio-check`), a pre-commit hook, or a CI step — see [`quality-gates-guide.md`](quality-gates-guide.md) § "Pattern C — Configured subagent" and the worked-example setup.

---

## Cross-trio anti-patterns

The five most damaging mistakes specific to writing the trio together.

### 1. Plan written before spec is finalized

Engineer drafts spec, doesn't wait for review, starts writing plan. Plan locks in architectural choices that the spec review then invalidates. Result: plan thrown out and rewritten.

**Fix:** Spec must be in *Reviewed* state (Open Questions resolved, ACs sharpened) before plan starts. Imposing a 24-hour SLA on spec review usually fixes this.

### 2. Tasks written before plan

The agent (or impatient engineer) jumps from spec to tasks, skipping plan. Tasks assume implementation order without knowing decisions — frequently produces tasks like *"add validation"* with no clear pattern to follow.

**Fix:** Tasks reference patterns from the plan (*"follow the middleware pattern from `AuthMiddleware`"*). If the plan isn't written, tasks can't make those references.

### 3. Spec retro-fitted from completed code

Engineer codes the feature first, then writes spec to "document what got built." The spec is no longer a spec — it's reverse-engineered description. Worse: it claims to be the spec, fooling future readers.

**Fix:** If you've already written code and want to document it, file under `docs/<feature>.md`, not `specs/`. Specs ship before code (see [`legacy-to-sdd-migration-guide.md`](legacy-to-sdd-migration-guide.md) for the discipline applied to whole codebases).

### 4. Trio written all at once by AI without human verification

The agent generates all three documents in one prompt. They're plausible-sounding, internally consistent, and partly wrong. The Context section invents constraints; the Alternatives Rejected lists generic options; the tasks reference files that don't exist.

**Fix:** Author the trio iteratively. Spec first; review. Plan from reviewed spec; review. Tasks from reviewed plan; verify. Each step is human-gated even if the agent does the typing.

### 5. The trio that doesn't audit

Three documents in `specs/YYYY-MM-feature-slug/`. Each looks complete in isolation. But:
- An AC in `spec.md` isn't covered by any task
- A "decision" in `plan.md` contradicts ADR-007
- A task in `tasks.md` touches a file that's not in `plan.md` § File structure

The implementation proceeds anyway. Code reviewer catches the gap. Now you're patching specs after the fact.

**Fix:** Run [the trio consistency check](#cross-artifact-consistency-checks) before the first commit. Better — make it a slash command (`/trio-check`) or a hook that runs on PR open.

---

## Slash commands worth having

A repo doing SDD seriously usually has these in `.claude/commands/`:

- **`/prd-new <idea>`** — *(furthest upstream)* turns a 1–3 sentence idea into a lean PRD draft, then fills the gaps by asking you the open questions. See [`prd-guide.md`](prd-guide.md) § "AI-assisted PRD authoring".
- **`/features-from-prd`** — *(upstream of the trio)* slices an accepted PRD into a prioritized, vertically-sliced feature list; each row becomes a spec. See [`prd-guide.md`](prd-guide.md) § "Slicing the PRD into features".
- **`/spec-new <feature-description>`** — drafts `spec.md` from a one-paragraph description ([prompt 1](#1-draft-specmd-from-a-one-paragraph-idea) above)
- **`/spec-review <path>`** — runs the audit checklist (prompt 2)
- **`/plan-from-spec`** — drafts `plan.md` from the active spec (prompt 3)
- **`/plan-validate`** — checks `plan.md` against ADRs and ARCHITECTURE.md (prompt 4)
- **`/tasks-from-plan`** — drafts `tasks.md` from scratch (prompt 5)
- **`/tasks-add <what>`** — appends/inserts task(s) into an existing `tasks.md` (or a one-file trio's Tasks section), in order, each with a verify step
- **`/trio-check`** — final consistency audit (prompt 6)
- **`/implement`** — *(downstream of the trio)* works `tasks.md` task-by-task red→green, commits each green task, then runs the break-the-code check. See [`flow-guide.md`](flow-guide.md) (Step 4) and [`testing-guide.md`](testing-guide.md).

Worked-example placement of these files:

```
.claude/
└── commands/
    ├── prd-new.md           # /prd-new
    ├── features-from-prd.md # /features-from-prd
    ├── spec-new.md          # /spec-new
    ├── spec-review.md       # /spec-review
    ├── plan-from-spec.md    # /plan-from-spec
    ├── plan-validate.md     # /plan-validate
    ├── tasks-from-plan.md   # /tasks-from-plan
    ├── tasks-add.md         # /tasks-add
    ├── trio-check.md        # /trio-check
    └── implement.md         # /implement
```

**Ready-made copies of all ten commands live in [`templates/.claude/`](../templates/.claude/)** — copy that folder into your project's `.claude/` and adjust the paths inside to match your layout.

See [`working-with-agents-guide.md` § Claude Code Building Blocks](working-with-agents-guide.md#claude-code-building-blocks) for the mechanics of writing slash commands.

---

## Worked example 1 — Rate limiting on the orders endpoint

The scenario: partner integrations occasionally enter retry storms, and the orders endpoint gets pummeled. We need per-partner rate limiting with proper 429 responses and observability.

Below: all three documents, fully filled in, for this single feature. Read them as one continuous piece — notice how each references the previous ones.

### `specs/2026-05-orders-rate-limit/spec.md`

```markdown
# Rate limiting for the orders API

## Goal

Add per-partner rate limiting to `POST /api/orders` to prevent partner integrations
from accidentally flooding the system during retry storms. The limit must be
configurable per partner and observable in production.

## In scope

- Rate limiting on `POST /api/orders` only
- Per-partner limits, configurable in `appsettings.json`
- 429 response with `Retry-After` header on limit breach
- Metrics for limit breaches (Prometheus counter via existing telemetry)

## Out of scope (deliberately, not now)

- Rate limiting on GET endpoints (no abuse vector identified)
- Distributed rate limiting (single-instance is fine for current load)
- IP-based limits (per-partner is the abuse vector we care about)
- Sliding-window algorithms (fixed window is enough)
- Customer-facing UI for limit configuration

## Acceptance criteria

- [ ] AC1: `POST /api/orders` with valid `X-Partner-Id` returns 201 on the first N requests within window W
- [ ] AC2: Same partner exceeding N requests in window W gets 429 with `Retry-After` header containing seconds-until-reset
- [ ] AC3: Different partners have independent counters (one partner's burst does not affect another)
- [ ] AC4: Missing `X-Partner-Id` header → 400 Bad Request (existing behavior preserved)
- [ ] AC5: Configuration loaded from `appsettings.json` § `RateLimits:Orders`
- [ ] AC6: Prometheus counter `orders_rate_limit_breaches_total{partner_id}` increments on each 429
- [ ] AC7: Integration test covers AC1–AC4 against Testcontainers Postgres + in-memory rate-limit store

## Impact on existing code

- `src/Api/Controllers/OrdersController.cs` — wraps `POST` with rate-limit middleware
- `src/Api/Middleware/` — new `RateLimitMiddleware.cs`
- `src/Api/Configuration/` — new `RateLimitOptions.cs` bound to `appsettings.json`
- `appsettings.json` — new `RateLimits:Orders` section
- No changes to `src/Domain/` or `src/Infrastructure/` (rate limiting is an API concern)
- No changes to existing controllers (middleware scoped to `OrdersController` only)

## Open questions

- [x] In-memory store sufficient, or do we need Redis? → **Resolved**: single-instance fine for current load; defer Redis to spec `2026-Q3-distributed-rate-limit`
- [x] Burst allowance (token bucket) vs strict fixed window? → **Resolved**: fixed window for v1
- [x] What to log on breach beyond Prometheus? → **Resolved**: Serilog Information level with `partner_id`, no body
- [x] What's the default limit, and can partners negotiate higher? → **Resolved**: default 100 req / 60s; per-partner overrides in `appsettings.json` keyed by `partner_id`; no runtime changes in v1 (config reload deferred)

## References

- ADR-001: Repository per aggregate (informs service layer placement)
- ADR-007: Dapper for data access (no DB change here, but consistent with stack)
- `docs/runbooks/orders-api-degraded.md` (will get a new diagnostic step)
- `docs/integrations/sftp-acme-bank.md` (partner contract referenced for retry behavior)
- Upstream cause: spec `2026-03-partner-retry-backoff`
```

### `specs/2026-05-orders-rate-limit/plan.md`

```markdown
# Plan: Rate limiting for the orders API

## Technical decisions

- Stack: .NET 8 ASP.NET Core middleware (per ADR-001)
- Store: in-memory `MemoryCache`-backed counter (single-instance, per spec § Open Q1 resolution)
- Algorithm: fixed window, 60-second buckets per `partner_id` (per spec § Open Q2 resolution)
- Config binding: Options pattern; `RateLimitOptions` bound to `RateLimits:Orders` section
- Metrics: use existing `IMetricsCollector` interface (avoid adding a new dependency)
- Logging: Serilog Information on each 429 (per spec § Open Q3 resolution)

## Data model

No persistent storage. In-memory counter structure:

```
ConcurrentDictionary<string partnerId, RateLimitBucket>
  RateLimitBucket: { Count, WindowStart, ResetAt }
```

Eviction: bucket expires when `ResetAt < UtcNow`; cleaned via `MemoryCache` TTL.

## File structure

```
src/Api/
├── Configuration/
│   └── RateLimitOptions.cs           # new — Options pattern binding
├── Middleware/
│   └── RateLimitMiddleware.cs        # new — the actual middleware
├── Services/
│   └── RateLimitService.cs           # new — counter logic, testable in isolation
├── Controllers/
│   └── OrdersController.cs           # modified — wire middleware
└── Startup.cs                         # modified — register middleware + options

tests/Api.Integration/
└── RateLimitTests.cs                 # new — covers AC1–AC4
```

## Constraints

- Do NOT add Redis dependency (single-instance per spec)
- Do NOT use ASP.NET Core 7's built-in rate limiter (its API is hostile to per-partner keys; we keep custom)
- Do NOT block on GETs (only `POST /api/orders`)
- `Retry-After` MUST be integer seconds, not an HTTP-date — a partner client can't parse the date form (see `docs/integrations/sftp-acme-bank.md`)
- Counter mutations must be atomic per `partner_id` — naive read-then-increment races under load (added after load test; see `tasks.md` notes)
- Maintain consistency with existing middleware patterns in `src/Api/Middleware/AuthMiddleware.cs`

## Open questions

None — all resolved during spec review (see `spec.md` § Open questions).

## References

- `spec.md` (this folder)
- ADR-001 — Repository per aggregate
- `src/Api/Middleware/AuthMiddleware.cs` — middleware pattern to mirror
```

### `specs/2026-05-orders-rate-limit/tasks.md`

```markdown
# Tasks: Rate limiting for the orders API

> Order matters. Each task has its own verification. Check off as you go.

## Setup

- [ ] Create branch `feature/2026-05-orders-rate-limit`
- [ ] Open `spec.md` + `plan.md` alongside the task list

## Implementation (in execution order)

- [ ] 1. Write the failing integration test for AC1 (happy path: 201 under limit) → verify: test fails with "endpoint exists but no rate-limit logic" output
- [ ] 2. Add `RateLimitOptions.cs` with bindings for `MaxRequests` and `WindowSeconds` per partner → verify: `dotnet build` succeeds; options bind correctly in a unit test
- [ ] 3. Add `appsettings.json` § `RateLimits:Orders` with example values → verify: app starts; options injected via DI
- [ ] 4. Implement `RateLimitService.IsAllowed(partnerId)` with fixed-window logic + unit tests → verify: unit tests cover under-limit / at-limit / just-over-limit / different-partners cases
- [ ] 5. Implement `RateLimitMiddleware` that calls the service and returns 429 + `Retry-After` on breach → verify: integration test for AC1 now passes
- [ ] 6. Wire middleware into pipeline scoped to `OrdersController.Post` only → verify: `GET /api/orders` still works; `POST` is gated
- [ ] 7. Add Prometheus counter `orders_rate_limit_breaches_total{partner_id}` → verify: counter increments observable in `/metrics` endpoint after a forced breach
- [ ] 8. Add Serilog Information log on each 429 with `partner_id` field → verify: log line appears in test output
- [ ] 9. Add remaining integration tests (AC2, AC3, AC4) → verify: full test suite green

## Verification (against acceptance criteria)

- [ ] AC1: First N requests within window W return 201 — covered by test in step 1
- [ ] AC2: N+1 request gets 429 + `Retry-After` — covered by AC2 test (step 9)
- [ ] AC3: Independent counters per partner — covered by AC3 test (step 9)
- [ ] AC4: Missing `X-Partner-Id` → 400 — covered by AC4 test (step 9)
- [ ] AC5: Config from appsettings.json — covered by step 3 + binding test
- [ ] AC6: Prometheus counter increments — covered by step 7
- [ ] AC7: Integration test suite green — covered by all tests above

## Post-merge

- [ ] Append `STATUS: shipped (PR #N, YYYY-MM-DD)` to `spec.md`
- [ ] Update `docs/runbooks/orders-api-degraded.md` with a new "check rate-limit breaches" diagnostic step
- [ ] Add a line to `CLAUDE.md` § Conventions: *"Per-endpoint middleware → `src/Api/Middleware/` with options bound from `appsettings.json`"*
- [ ] Close ticket ORDERS-1234

## Notes (append as you work)

- [2026-05-15]: AC4 (missing `X-Partner-Id` → 400) is already enforced by existing `AuthMiddleware`. Step 9's AC4 test is just a regression guard.
- [2026-05-16]: `MemoryCache` TTL had an eviction-timing edge case in tests — switched to explicit cleanup in service. Documented in `plan.md` Technical decisions.
- [2026-05-16]: Load test (500 rps, single partner) exposed a race — two threads read-then-incremented the counter and both passed the limit. Switched to a per-`partner_id` lock in `RateLimitService`; added the atomicity constraint to `plan.md`.
- [2026-05-17]: The AC2 window-boundary test was flaky — it depended on wall-clock and failed when the 60s window rolled mid-test. Injected an `IClock` and froze time in the test, instead of a `Thread.Sleep` hack.
```

---

## Worked example 3 — the whole trio in one file

Some changes are too big for a bare bugfix spec — they have real *how* and *order* decisions — but too small to deserve three separate files and a folder. Keep all three concerns; just collapse them into one file with three sections.

Here's a small feature — pagination on a list endpoint — written as a single `specs/2026-06-orders-pagination.md`:

```markdown
# Add pagination to `GET /api/orders`

> Small feature: spec + plan + tasks as three sections in one file.
> Split into a `specs/2026-06-orders-pagination/` folder with separate files only if it grows.

## Spec — what & why

**Goal.** `GET /api/orders` returns every order in one response; for large partners
that means multi-megabyte payloads and slow queries. Add offset-based pagination with
sane defaults.

**In scope**
- `?page` and `?pageSize` query params on `GET /api/orders`
- Default `pageSize=50`; reject `pageSize > 200` with a 400
- Response envelope gains `page`, `pageSize`, `totalCount`

**Out of scope**
- Cursor / keyset pagination (offset is fine at current scale)
- Pagination on any other endpoint
- Sort controls (separate change)

**Acceptance criteria**
- [ ] AC1: no params → first 50 orders + envelope (`page=1`, `pageSize=50`, `totalCount=N`)
- [ ] AC2: `?page=2&pageSize=20` → orders 21–40
- [ ] AC3: `?pageSize=500` → 400, message "pageSize exceeds max of 200"
- [ ] AC4: `?page=999` past the end → 200 with an empty list (not an error)

## Plan — how

- Bind params via a `PaginationQuery` record with `[Range]` validation (max 200)
- Page in SQL, never in memory: `ORDER BY created_at DESC, id DESC OFFSET @skip ROWS FETCH NEXT @take ROWS ONLY` (Dapper) — the `id` tiebreaker keeps paging deterministic (see Notes)
- Get `totalCount` from a second statement in the same Dapper multi-query (one round trip)
- Wrap the existing `OrderDto[]` in a `PagedResult<OrderDto>` envelope
- Touch only: new `PaginationQuery.cs` + `PagedResult.cs`; modify `OrdersController.Get` and `OrderRepository.List`

## Tasks — in what order

- [ ] 1. Failing integration test for AC1 → verify: fails (no envelope yet)
- [ ] 2. Add `PaginationQuery` + `PagedResult` records → verify: build green, range validation unit-tested
- [ ] 3. `OrderRepository.List(skip, take)` with OFFSET/FETCH + count → verify: repo test returns the right slice and total
- [ ] 4. Wire `OrdersController.Get` to bind the query and return `PagedResult` → verify: AC1 passes
- [ ] 5. Tests for AC2–AC4 → verify: suite green

## Notes

- Load test caught it: `ORDER BY created_at DESC` alone isn't deterministic (`created_at` isn't unique), so a boundary row jumped between pages 1 and 2. Added the `id` tiebreaker — folded into the Plan line above.
- Deep pages (`?page=900`) are slow: OFFSET scans then discards rows. Fine now (partners page shallow); when it bites, switch to the keyset pagination already parked in *Out of scope*.
```

**Why one file works here:**

- The three sections still appear **in order** — you read top-to-bottom and the discipline holds. The order is the method; the file count isn't.
- A small feature's plan and tasks are short. Three nearly-empty files cost more attention — a folder, cross-references, open tabs — than three headings in one.
- Cross-references collapse to *"see the section above"* — no need to cite `plan.md` by name when it's fifteen lines up.
- It promotes cleanly: if the change grows, move the three sections into `spec.md` / `plan.md` / `tasks.md` in a folder of the same name. Nothing is rewritten, only relocated.

Reach for the one-file trio when a change has genuine *how* and *order* decisions but separate files would just be padding. Reach for the full three-file trio when the plan or tasks get long, or when more than one person edits them at once.

---

## Golden rules for trio authoring

1. **Order is the discipline.** Spec → plan → tasks → code. Skipping or reversing produces worse output for the same total effort.
2. **Each document references the previous one explicitly.** Without cross-refs, you have three documents that share a folder, not a coherent decision chain.
3. **The agent drafts; you judge.** Especially for Context, Alternatives Rejected, and Acceptance Criteria — the agent's plausible-sounding fills are most likely to be wrong here.
4. **Human gates between artifacts.** Spec → review → plan → review → tasks → verify. No skipping reviews because *"the agent said it was fine."*
5. **A consistency check before the first commit.** Run `/trio-check` (or the equivalent prompt) before you start implementing; catch contradictions when they're cheap to fix.
6. **Compress for small changes.** A bugfix doesn't need three documents. A short spec is enough. The trio is for changes worth the ceremony.
7. **Out of scope is the most important section in spec.md.** It does more to prevent drift than any positive guidance.
8. **Every AC traces to a task. Every task traces to an AC.** No orphans.
9. **Open questions resolve, they don't linger.** A `Proposed` ADR > 1 month old, or an Open Question still `[ ]` two weeks into a spec, is rot.
10. **The trio freezes after merge.** Don't tidy retrospectively. The messy reality of `tasks.md` notes is more valuable than a clean rewrite.

---

*This guide complements [`spec-driven-development-guide.md`](spec-driven-development-guide.md) § "Writing a Good spec.md", "Writing a Good PLAN.md", and "Writing a Good tasks.md" (the principles), [`working-with-agents-guide.md`](working-with-agents-guide.md) § "Working on Specs, ADRs, and Refactors" (the prompts for after-creation), and [`sdd-in-teams-guide.md`](sdd-in-teams-guide.md) § "Spec lifecycle" (who owns what). Together they form the complete picture: principles in main SDD, practice here, post-spec implementation in working-with-agents, ownership in teams.*
