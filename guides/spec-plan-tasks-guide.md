# Writing the Feature Trio: spec → plan → tasks (with AI help)

> The companion guide to the principle-level treatment in `spec-driven-development-guide.md`. Here you'll find two full worked examples (a complete feature trio you can copy as a starting point), the AI-assisted authoring prompts for each artifact, the consistency checks that keep the three documents honest, and the anti-patterns specific to writing all three together.

---

## Table of Contents

1. [What this guide adds](#what-this-guide-adds)
2. [The trio as a flow](#the-trio-as-a-flow)
3. [Worked example 1 — Rate limiting on the orders endpoint](#worked-example-1--rate-limiting-on-the-orders-endpoint)
4. [How the three documents reference each other](#how-the-three-documents-reference-each-other)
5. [AI-assisted authoring: prompts per artifact](#ai-assisted-authoring-prompts-per-artifact)
6. [Iteration patterns — sharpening a draft](#iteration-patterns--sharpening-a-draft)
7. [Cross-artifact consistency checks](#cross-artifact-consistency-checks)
8. [Worked example 2 — a small change (bugfix shape)](#worked-example-2--a-small-change-bugfix-shape)
9. [When to skip parts of the trio](#when-to-skip-parts-of-the-trio)
10. [Cross-trio anti-patterns](#cross-trio-anti-patterns)
11. [Slash commands and skills worth having](#slash-commands-and-skills-worth-having)
12. [Golden rules for trio authoring](#golden-rules-for-trio-authoring)

---

## What this guide adds

The main SDD guide has three sections — *Writing a Good spec.md*, *Writing a Good PLAN.md*, *Writing a Good tasks.md* — covering principles, templates, and brief tips. Each in isolation.

This guide adds what doesn't fit there:

- **Two complete worked examples** showing what all three filled-in documents look like *for the same feature*, with the cross-references explicit
- **AI-assisted authoring prompts** — copy-pasteable prompts for drafting each artifact, reviewing it, refining it, and validating it against the others
- **Iteration patterns** — how a rough draft becomes a tight spec, then a clear plan, then an executable task list
- **Cross-artifact consistency checks** — concrete rules to verify the three documents agree (every acceptance criterion has a task; every "out of scope" survives in the plan; every open question resolves into a decision or an ADR)

Read the main SDD's three sections first for principles; come here when you need to actually write the trio for a real feature.

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

Each step narrows the option space. Once `spec.md` is accepted, the team has agreed on *what success means*. Once `plan.md` is accepted, the team has agreed on *the shape of the solution*. Once `tasks.md` is set, the engineer (and the agent) just executes.

**The order matters because it forces decisions before code.** Writing tasks before plan forces premature ordering assumptions. Writing plan before spec forces premature architecture decisions. Writing spec after code is fiction.

For tiny changes, the trio compresses (see [When to skip parts of the trio](#when-to-skip-parts-of-the-trio)). For most non-trivial work, all three earn their place.

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
```

---

## How the three documents reference each other

The trio is most useful when each document *visibly* refers to the others. The example above shows it; here are the references made explicit:

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

## Worked example 2 — a small change (bugfix shape)

Not every change needs a full trio. Below: a bugfix where the trio compresses into one short document.

### `specs/2026-05-orders-422-typo-fix/spec.md`

```markdown
# Fix: `POST /api/orders` returns 422 instead of 400 on malformed JSON

## Goal

Fix a small response-code regression: the orders endpoint started returning
422 (Unprocessable Entity) instead of 400 (Bad Request) on malformed JSON
bodies. Partner integrations log 400 as transient and 422 as terminal; this
breaks the retry contract.

## In scope

- `POST /api/orders` returns 400 on malformed JSON
- Existing 422 behavior on valid-JSON-but-invalid-fields preserved

## Out of scope

- Other endpoints (verified unaffected; they use the same middleware which is fine)
- Changing the 422-on-validation-error behavior

## Acceptance criteria

- [ ] AC1: `POST /api/orders` with `{"foo":` (malformed JSON) → 400 Bad Request
- [ ] AC2: `POST /api/orders` with `{}` (valid JSON, missing required fields) → 422 (unchanged)
- [ ] AC3: Integration test added for AC1 (regression guard)

## Impact

- One change in `src/Api/Middleware/JsonErrorMiddleware.cs` — a config flag flip
- New test in `tests/Api.Integration/OrdersControllerTests.cs`

## References

- Bug report: ORDERS-1257
- Spec `2026-04-validation-response-codes` (the change that introduced the regression — superseded)
```

That's the whole spec. No plan.md, no tasks.md.

**Why this compression works for a bugfix:**

- The change is one file, one line, one test. *Plan* is implicit ("flip the flag").
- The execution order is trivial: write the test, flip the flag, verify. *Tasks* would be three checkboxes that don't deserve their own file.
- The acceptance criteria are testable directly; no orchestration of multiple steps.

For changes like this, a single `spec.md` is enough. The trio is for non-trivial features; bugfixes get the appropriate compression.

---

## When to skip parts of the trio

| Change shape | spec.md | plan.md | tasks.md |
|--------------|---------|---------|----------|
| Bugfix, single file, single commit | ✅ short | ❌ skip | ❌ skip |
| Small feature (1–3 files, ~half day work) | ✅ full | optional | ❌ skip |
| Non-trivial feature (multiple modules, multi-day) | ✅ full | ✅ full | ✅ full |
| Refactor (no behavior change, larger code surface) | ✅ short | ✅ full | ✅ full |
| Spike / research | ✅ heavy on Open Questions | ❌ skip until the spike resolves | ❌ skip |
| One-line CLAUDE.md or doc update | ❌ skip (PR description suffices) | ❌ | ❌ |

The rule: **if you'd struggle to explain the change in a one-paragraph PR description, write the spec.** If the implementation order would change if you were interrupted for a week, write the tasks. The plan emerges in between.

When in doubt, write the shorter form. You can always escalate later if the change grows; you can't recover the time spent ceremony-ing a tiny change.

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

## Slash commands and skills worth having

A repo doing SDD seriously usually has these in `.claude/commands/` and `.claude/skills/`:

**Slash commands** (single-shot, user-invoked):

- **`/spec-new <feature-description>`** — drafts `spec.md` from a one-paragraph description (the prompt from [section 5.1](#1-draft-specmd-from-a-one-paragraph-idea))
- **`/spec-review <path>`** — runs the audit checklist (prompt 5.2)
- **`/plan-from-spec`** — drafts `plan.md` from the active spec (prompt 5.3)
- **`/plan-validate`** — checks `plan.md` against ADRs and ARCHITECTURE.md (prompt 5.4)
- **`/tasks-from-plan`** — drafts `tasks.md` (prompt 5.5)
- **`/trio-check`** — final consistency audit (prompt 5.6)

**Skills** (multi-step, auto-invoked):

- **`trio-author`** — sequenced workflow that drafts spec, prompts you to review, then drafts plan, prompts review, then tasks. Good for the standard feature flow.
- **`trio-consistency`** — same as `/trio-check` but invokable from prompts that aren't aware they're trio-related (the agent can auto-call this when it detects spec/plan/tasks edits).

Worked-example placement of these files:

```
.claude/
├── commands/
│   ├── spec-new.md          # /spec-new
│   ├── spec-review.md       # /spec-review
│   ├── plan-from-spec.md    # /plan-from-spec
│   ├── plan-validate.md     # /plan-validate
│   ├── tasks-from-plan.md   # /tasks-from-plan
│   └── trio-check.md        # /trio-check
└── skills/
    ├── trio-author/
    │   └── SKILL.md
    └── trio-consistency/
        └── SKILL.md
```

See [`working-with-agents-guide.md` § Claude Code Building Blocks](working-with-agents-guide.md#claude-code-building-blocks) for the mechanics of writing skills and slash commands.

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
