# Quality Gates: Enforcing and Evaluating SDD with Hooks, Subagents, and CI

> Three categories of checks (mechanical, LLM-evaluator, human) and five implementation patterns (pre-commit hooks, Claude Code hooks, configured subagents, slash commands, CI). The honest version of *"how do I make sure my team actually does SDD?"* — including where automation backfires.

---

## Table of Contents

1. [What this guide adds](#what-this-guide-adds)
2. [For most 1–10 teams: the lightweight version](#for-most-110-teams-the-lightweight-version)
3. [Enforcement vs evaluation — same tooling, different purposes](#enforcement-vs-evaluation--same-tooling-different-purposes)
4. [Three categories of checks: mechanical, LLM, human](#three-categories-of-checks-mechanical-llm-human)
5. [What belongs in each category](#what-belongs-in-each-category)
6. [Pattern A — Pre-commit / Git hooks (mechanical)](#pattern-a--pre-commit--git-hooks-mechanical)
7. [Pattern B — Claude Code hooks (mechanical, in-session)](#pattern-b--claude-code-hooks-mechanical-in-session)
8. [Pattern C — Configured subagent (LLM evaluator)](#pattern-c--configured-subagent-llm-evaluator)
9. [Pattern D — Slash commands (LLM evaluator, user-invoked)](#pattern-d--slash-commands-llm-evaluator-user-invoked)
10. [Pattern E — CI/CD checks (mechanical + LLM, team-wide)](#pattern-e--cicd-checks-mechanical--llm-team-wide)
11. [Worked example — complete setup for one repo](#worked-example--complete-setup-for-one-repo)
12. [What to mechanize vs leave human](#what-to-mechanize-vs-leave-human)
13. [Anti-patterns](#anti-patterns)
14. [Golden rules](#golden-rules)

---

## What this guide adds

The main SDD guide and the artifact-specific guides describe *what* each artifact should look like. They give principles, templates, and worked examples. What they don't give:

- **How to verify** that an actual `SPEC.md` in a real repo meets the bar
- **How to block** edits that would damage history (Accepted ADR bodies, PII in research)
- **How to scale** SDD discipline so it doesn't depend on every contributor remembering every rule

This guide fills that gap. It covers:

- **Five implementation patterns** for quality gates (pre-commit, Claude Code hooks, subagents, slash commands, CI)
- **Three categories of checks** with explicit guidance on which goes where
- **What to mechanize vs leave human** — the most underrated topic
- **Worked example** of a full setup for one repo
- **Anti-patterns** specific to over-automating SDD

What it doesn't cover:

- *"How to lint markdown formatting"* — that's tooling-specific and well-covered elsewhere
- *"How to set up GitHub Actions from scratch"* — out of scope; assume the reader has CI
- *"Every possible regex check"* — selectively useful, exhaustive lists go stale

For the building blocks themselves (skills, slash commands, subagents, hooks — what they are, how they work in Claude Code), see [`working-with-agents-guide.md` § "Claude Code Building Blocks"](working-with-agents-guide.md#claude-code-building-blocks). This guide assumes you've read that section and focuses on *applying* those blocks to SDD discipline.

---

## For most 1–10 teams: the lightweight version

**You almost certainly don't need the five patterns below.** For a solo dev or a 2–3 person team, the whole of "making SDD stick" is three things you already have:

- **Good docs the agent actually reads** — a fresh `CLAUDE.md`, a spec before each change. The gate that matters most, and it needs no automation.
- **One human on the PR** — a second pair of eyes reading the diff against the spec's acceptance criteria. Judgment, not tooling (see [`sdd-in-teams-guide.md`](sdd-in-teams-guide.md)).
- **The break-the-code check** — already built into `/sdd-7-implement` ([`testing-guide.md`](testing-guide.md)). It catches confidently-wrong green suites for free.

Add exactly one mechanical gate when you feel a specific pain: a `pii-scan` pre-commit hook the first time someone nearly commits a customer name ([Pattern A](#pattern-a--pre-commit--git-hooks-mechanical)), or a `/sdd-6-trio-check` slash command the third time a trio ships with an uncovered AC ([Pattern D](#pattern-d--slash-commands-llm-evaluator-user-invoked)). Nothing else, until it earns its place.

**Everything below — the five patterns, the CI matrix, the full worked setup — is "when you outgrow ~10":** multiple contributors who don't share a memory, a real CI pipeline, ADR bodies worth protecting mechanically. Read it then. *(The hook/CI snippets are illustrative — Claude Code's hook interface and exit-code semantics vary by version; adapt the specifics to your harness rather than pasting verbatim.)*

---

## Enforcement vs evaluation — same tooling, different purposes

Two related but distinct activities:

- **Enforcement** = *blocking* bad actions. *"Don't let anyone edit the body of an Accepted ADR."*
- **Evaluation** = *scoring* existing artifacts. *"Does this spec have testable acceptance criteria?"*

They share infrastructure (the same hooks, subagents, and CI runners can do either) but the design choice matters:

- An enforcement check that wrongly fires becomes a workaround magnet (commits with `--no-verify`, edited-around hooks, frustration).
- An evaluation check that wrongly fires becomes noise. People stop reading the reports.

**Asymmetric cost as the rule of thumb:**

| If false positive is… | Use this for | Example |
|----------------------|--------------|---------|
| Cheap (one extra review) | Enforcement (block) | Reject edits to ADRs with `Status: Accepted` body |
| Expensive (blocks legitimate work) | Evaluation (warn, don't block) | "This spec's ACs look untestable — please review" |
| Catastrophic if false negative | Hard enforce, accept some false positives | PII regex on `docs/research/` commits |

The cost asymmetry decides whether a check blocks or warns. Get this wrong and your team will route around your discipline.

---

## Three categories of checks: mechanical, LLM, human

Different checks demand different tools. The mistake most teams make: trying to automate something that needs judgment.

### Mechanical (regex, structure, format)

The check is fully specifiable: this pattern must match, this section must exist, this format must hold.

Examples:
- Does the file have an `## Acceptance criteria` section?
- Are ACs formatted as `- [ ]` checkboxes?
- Does the file path match `specs/YYYY-MM-*/spec.md`?
- Does the commit contain PII patterns (email, phone, SSN-like numbers)?
- Is the ADR's `## Status` line `Accepted` while its body was just edited?

Implementation: pre-commit hooks, Git hooks, Claude Code PreToolUse/PostToolUse hooks, CI lint steps.

**Strength:** deterministic, fast, no token cost.
**Weakness:** can't judge quality; only presence/format/pattern.

### LLM-evaluator (semantic, structural-with-judgment)

The check has rules but requires understanding meaning. *"Is this AC testable?"* is rule-based ("could you write a test from it?") but needs language understanding to evaluate.

Examples:
- Are the acceptance criteria phrased as something you could write a test for?
- Does the spec leak implementation detail (specific class names from `src/`)?
- Do the tasks in `TASKS.md` actually cover every AC in `SPEC.md`?
- Does this plan contradict any active ADR?
- Is the "Out of scope" section trivial (only listing obvious exclusions) or substantive?

Implementation: configured subagent (`.claude/agents/trio-auditor.md`), slash command (`/spec-check`), or CI step that invokes the agent. (Note on names: the shipped command files in `templates/.claude/commands/` are namespaced and phase-numbered — `sdd-6-trio-check.md` → `/sdd-6-trio-check`, `sdd-3-spec-review.md` → `/sdd-3-spec-review`. Example gate names like `/spec-check` and `/trio-auditor` below are illustrative ones you author yourself, not shipped commands.)

**Strength:** can evaluate semantic content, catches what regex can't.
**Weakness:** probabilistic (occasional false positives and negatives), takes tokens, slower than mechanical.

### Human (judgment, context, intent)

The check fundamentally requires human knowledge.

Examples:
- Is this Acceptance Criterion *complete* — does it capture what we actually need?
- Does this Out-of-scope section include the things a reader might *surprise-assume* are in scope?
- Is this PRD properly scoped to an era boundary, or is it incremental feature work?
- Are the Alternatives Rejected in this ADR real options, or straw men?
- Does this spec serve the actual user need, or is it solving an imaginary problem?

Implementation: code review, PR reviewer checklist, architecture meeting, retrospective.

**Strength:** captures intent, context, completeness; irreplaceable for non-obvious quality.
**Weakness:** doesn't scale; depends on reviewer attention; gets shortcut under pressure.

---

## What belongs in each category

For each SDD artifact, a working split. This is a menu of what *could* be automated, not a list of requirements: a solo dev skips all of it, a 2–5 person team might lift one or two checks into a `/sdd-6-trio-check`-style command, and only toward 10 does wiring any of this into CI start to pay for itself.

### SPEC.md

**Mechanical:**
- Path matches `specs/YYYY-MM-*/spec.md`
- Required sections present (Goal, In scope, Out of scope, Acceptance criteria, Impact on existing code, Open questions, References)
- ACs are formatted as `- [ ]` checkboxes
- "Out of scope" has ≥ 3 items
- File length under ~150 lines

**LLM evaluator:**
- Each AC could plausibly be turned into a test (heuristic: contains a specific behavior, not a vibe)
- Spec doesn't reference class names that aren't in `src/` (didn't invent file paths)
- Open Questions read as genuine uncertainties, not as decisions in disguise
- Problem statement is specific (named user + behavior), not generic
- Spec doesn't include implementation detail that belongs in `PLAN.md`

**Human:**
- Do the ACs *fully cover* the intent? (Are any missing?)
- Is the Out-of-scope list complete enough to prevent drift?
- Does the spec match what the PRD actually called for?

### PLAN.md

**Mechanical:**
- References `SPEC.md` in References section
- File structure section non-empty
- Constraints section non-empty (even if "none")

**LLM evaluator:**
- Technical decisions don't contradict any Accepted ADR (verify by reading `docs/adr/`)
- File structure aligns with `ARCHITECTURE.md` boundaries
- Open Questions are either resolved from the spec, or genuinely new HOW questions

**Human:**
- Are technical decisions *good* decisions for this context?
- Is the file structure right, or does it just look right?

### TASKS.md

**Mechanical:**
- Each task line contains `→ verify:`
- Tasks formatted as `- [ ]` checkboxes
- "Verification" section exists and references ACs

**LLM evaluator:**
- Every AC in `SPEC.md` is covered by at least one task
- Every file in `PLAN.md` § File structure is touched by at least one task
- Task granularity reasonable (no 30-second tasks, no 4-hour tasks)
- Verification phrasing is actionable

**Human:**
- Is the *order* sensible (dependencies respected)?
- Are tasks small enough to checkpoint but not so small they're noise?

### Cross-artifact (the trio together)

**Mechanical:**
- All three files exist in the same `specs/YYYY-MM-*/` folder

**LLM evaluator:**
- Spec doesn't reference files that the plan doesn't cover (spotting file references in prose takes judgment, not regex)
- Every Out-of-scope item in spec is respected by plan (no task touches it)
- Every Open Question in spec is marked `[x]` (resolved) or moved to ADR
- Plan-cited ADRs exist and are Accepted

**Human:**
- Is the trio internally consistent in *intent* (not just in mechanics)?

---

## Pattern A — Pre-commit / Git hooks (mechanical)

The earliest gate: catch issues before code enters the repo. Pre-commit framework (`pre-commit.com`) or hand-written Git hooks (`.git/hooks/pre-commit`).

**Use for:**
- Blocking PII patterns in `docs/research/`
- Blocking edits to bodies of ADRs with `Status: Accepted`
- Checking spec structural format on commit

**Example: pre-commit config for SDD basics**

`.pre-commit-config.yaml`:

```yaml
repos:
  - repo: local
    hooks:
      - id: pii-scan-research
        name: PII scan on docs/research/
        entry: scripts/pii-scan.sh
        language: script
        files: ^docs/research/.*\.md$
        pass_filenames: true

      - id: adr-accepted-body-edit
        name: Block edits to body of Accepted ADRs
        entry: scripts/adr-frozen-check.sh
        language: script
        files: ^docs/adr/ADR-.*\.md$
        pass_filenames: true

      - id: spec-required-sections
        name: Spec has required sections
        entry: scripts/spec-format-check.sh
        language: script
        files: ^specs/.+/spec\.md$
        pass_filenames: true
```

**Example: `scripts/pii-scan.sh`**

```bash
#!/usr/bin/env bash
# Scan staged research files for PII patterns. Block commit if found.
set -e
for file in "$@"; do
  if grep -qE '\b[A-Z][a-z]+ [A-Z][a-z]+\b|@[a-zA-Z0-9.-]+\.[a-z]{2,}|\b[0-9]{3}-[0-9]{3}-[0-9]{4}\b' "$file"; then
    echo "ERROR: $file may contain PII (names, emails, phone numbers)." >&2
    echo "Anonymize before committing. See docs/research/ guidance." >&2
    exit 1
  fi
done
```

(Heuristic — will have false positives. For research, this is acceptable: better to over-flag than to leak.)

**Example: `scripts/adr-frozen-check.sh`**

```bash
#!/usr/bin/env bash
# If an ADR has Status: Accepted, only its Status header (and title) may change.
# Pre-commit: checks staged changes. CI: pass --ci to diff against origin/main instead.
set -e
[ "${SDD_BYPASS_ADR:-0}" = "1" ] && exit 0   # documented escape hatch
diff_range=(--cached)
if [ "${1:-}" = "--ci" ]; then diff_range=(origin/main...HEAD); shift; fi
files=("$@"); [ ${#files[@]} -eq 0 ] && files=(docs/adr/ADR-*.md)
for file in "${files[@]}"; do
  [ -f "$file" ] && grep -q '^Status: Accepted' "$file" || continue
  # Changed content lines only: drop diff metadata (+++/---), headings,
  # and the explicitly allowed Status-header lines.
  body_changes=$(git diff "${diff_range[@]}" -- "$file" \
    | grep -E '^[+-]' \
    | grep -vE '^(\+\+\+|---) ' \
    | grep -vE '^[+-](#|Status:|\*\*Status|Superseded by|Deprecated)' || true)
  if [ -n "$body_changes" ]; then
    echo "ERROR: $file has Status: Accepted. Only the Status header may change." >&2
    echo "To revise, create a new ADR with Supersedes: ADR-NNN." >&2
    echo "(Legitimate exception? Run with SDD_BYPASS_ADR=1.)" >&2
    exit 1
  fi
done
```

**Example: `scripts/spec-format-check.sh`**

```bash
#!/usr/bin/env bash
# Minimal structural check on a spec.md: required sections + AC checkboxes.
status=0
for file in "$@"; do
  for section in '## Goal' '## Acceptance criteria' '## Out of scope'; do
    grep -q "^$section" "$file" || { echo "ERROR: $file missing \"$section\" section." >&2; status=1; }
  done
  grep -q -- '- \[ \]' "$file" \
    || { echo "ERROR: $file has no '- [ ]' acceptance-criteria checkboxes." >&2; status=1; }
done
exit $status
```

**Trade-offs:**

- Pros: catches issues earliest; works for anyone using the repo (not just Claude Code users); team-wide once committed
- Cons: requires `pre-commit install` per contributor; can be bypassed with `--no-verify`; runs only on `git commit` (not on agent edits in real time)

---

## Pattern B — Claude Code hooks (mechanical, in-session)

Hooks in `.claude/settings.json` fire on Claude Code events: `PreToolUse`, `PostToolUse`, `Stop`, `UserPromptSubmit`. Useful when you want the agent (or the user) to see the failure *immediately*, not at commit time.

Two mechanics to know. Hook commands receive a JSON payload on **stdin** (`tool_name`, `tool_input.file_path`, `tool_input.command`, …) — read it with `jq`. And exit codes carry meaning: **0** allows, **2** is the blocking error whose stderr is fed back to the agent (on `PreToolUse` it blocks the tool call), and any other non-zero exit is non-blocking — shown to the user only, *not* to the agent.

**Use for:**
- Same checks as Pattern A but with faster feedback during a session
- Injecting context (active ADR list before every prompt)
- End-of-session reminders

**Example: block agent from editing Accepted ADR bodies**

`.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "[ \"${SDD_BYPASS_ADR:-0}\" = \"1\" ] && exit 0; f=$(jq -r '.tool_input.file_path // empty'); if [[ \"$f\" == *docs/adr/* ]] && grep -q '^Status: Accepted' \"$f\" 2>/dev/null; then echo 'Blocked: ADR is Accepted. Only header may change. Create a Supersedes ADR instead.' >&2; exit 2; fi"
          }
        ]
      }
    ]
  }
}
```

The `SDD_BYPASS_ADR=1` check at the front is the documented escape hatch (golden rule 4): set it for one session when you legitimately need to touch an Accepted ADR body — visible, deliberate, no hook-disabling.

**Example: warn (don't block) when a spec is edited without checking format**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "f=$(jq -r '.tool_input.file_path // empty'); case \"$f\" in *specs/*/spec.md) ./scripts/spec-format-check.sh \"$f\" >&2 || exit 2 ;; esac"
          }
        ]
      }
    ]
  }
}
```

(Note the `exit 2`: on `PostToolUse` the edit has already happened, so nothing is undone — but exit 2 is the only exit code that feeds the stderr message back to the agent, which can then decide whether to fix. A plain non-zero exit would show the message to the user only, and the agent would never see it.)

**Example: inject active ADR list before each prompt**

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Active ADRs (from docs/adr/, Status: Accepted only):' && grep -l '^Status: Accepted' docs/adr/*.md 2>/dev/null | sed 's|docs/adr/||'"
          }
        ]
      }
    ]
  }
}
```

The agent sees this context appended to every prompt, reducing the chance it'll forget about an ADR mid-task.

**Trade-offs:**

- Pros: immediate feedback inside Claude Code; can both block and warn; integrates with the agent's reasoning
- Cons: only fires when Claude Code is the editor (manual `vim` bypasses); requires committing `.claude/settings.json` for team-wide use; over-talkative hooks fatigue

---

## Pattern C — Configured subagent (LLM evaluator)

For checks that need language understanding: testability of ACs, implementation leakage, cross-artifact consistency. Define a subagent in `.claude/agents/<name>.md` and invoke it on demand (or auto-trigger via hook).

**Use for:**
- Cross-artifact consistency checks (every AC has a task; out-of-scope respected)
- Spec quality audit (ACs testable; problem specific; no leakage)
- Trio audit before implementation
- Periodic ADR consistency review

**Example: `.claude/agents/trio-auditor.md`**

```yaml
---
name: trio-auditor
description: Audit a spec/plan/tasks trio in a specs/ folder for cross-artifact
  consistency. Use before implementation starts on a non-trivial feature, or
  when verifying that a draft trio is implementation-ready.
tools: Read, Grep, Glob
---

The user will give you a specs/YYYY-MM-feature-slug/ folder path. Read
spec.md, plan.md, and tasks.md in that folder. Also read CLAUDE.md and
all files matching docs/adr/ADR-*.md with Status: Accepted.

Run these checks and return a markdown table (Check, Status, Evidence):

1. Every acceptance criterion in spec.md has at least one task in tasks.md
   that produces evidence for it.
2. Every file mentioned in plan.md § File structure has at least one task
   that creates or modifies it.
3. Every "Out of scope" item in spec.md is respected — no task touches it.
4. Every Open Question in spec.md is marked [x] (resolved) or moved to
   an ADR (cite the ADR number).
5. plan.md's "Technical decisions" don't contradict any Accepted ADR.
   Quote the relevant ADR text for any conflict.
6. tasks.md uses only file paths listed in plan.md § File structure.
7. spec.md acceptance criteria are testable (could each be turned into a
   test name?). Flag any that are vague ("works correctly", "users happy").
8. spec.md doesn't leak implementation detail (no specific class names from
   src/, no framework names that belong in plan.md).

For each FAIL, cite the specific line in the trio that triggered it.

Don't modify any files. Return only the report.
```

Invoke from the main session:

**Prompt:**

```text
Spawn the trio-auditor subagent for specs/2026-05-orders-rate-limit/.
```

The subagent reads, analyzes, and returns a structured report. The main session's context stays clean. Expect run-to-run variance: verdicts on judgment calls (e.g. "no task touches an out-of-scope item") can differ between runs of the same audit — treat them as advisory annotations, never as required checks.

**Trade-offs:**

- Pros: handles semantic checks; cleanly separated context; reusable across the team once committed
- Cons: probabilistic (occasional misses); costs tokens; requires the agent to actually invoke it (see [`working-with-agents-guide.md` § "When the agent skips spawning a subagent"](working-with-agents-guide.md#when-the-agent-skips-spawning-a-subagent))

---

## Pattern D — Slash commands (LLM evaluator, user-invoked)

Lighter than a subagent: user explicitly types `/sdd-6-trio-check` to run the audit. Useful when you want quick, repeatable evaluation without setting up a full subagent.

**Use for:**
- One-off audits when the user wants them
- Checks that don't justify subagent overhead (under ~5 tool calls)
- Quick spec/plan review during authoring

**Example: `.claude/commands/spec-check.md`**

```markdown
Read the spec.md file referenced by $ARGUMENTS (path to a spec folder).

Run this audit checklist and return findings as a numbered list:

1. Required sections present (Goal, In scope, Out of scope, Acceptance criteria,
   Impact on existing code, Open questions, References)?
2. Out of scope has ≥ 3 items?
3. Each AC formatted as `- [ ]` checkbox?
4. Each AC phrased so it could be turned into a test name? (Specific behavior,
   not vague outcome.)
5. Problem statement names a specific user/role and a specific behavior?
6. No implementation detail leaked (class names from src/, framework names
   belonging in plan.md)?
7. Open Questions are genuine uncertainties (not decisions in disguise)?

For each FAIL, quote the offending line. For PASS, just "OK".

Don't modify the file. Surface only.
```

Then the user runs:

```
/spec-check specs/2026-05-orders-rate-limit/
```

**Trade-offs:**

- Pros: lighter than subagent; user-explicit invocation; easy to iterate
- Cons: runs in main session (uses its context); requires user to remember to invoke

---

## Pattern E — CI/CD checks (mechanical + LLM, team-wide)

For team-wide invariants that should fire on every PR regardless of who created it (human, agent, automated). Pre-commit only catches local commits; CI catches PRs and pushes from any source.

**Use for:**
- Required status checks on PRs (mechanical structure validation)
- Automated trio audit when specs/ changes in a PR
- PII scan as a required check on docs/research/

**Example: GitHub Actions workflow for SDD basics**

`.github/workflows/sdd-check.yml`:

```yaml
name: SDD discipline checks

on:
  pull_request:
    paths:
      - 'specs/**'
      - 'docs/adr/**'
      - 'docs/research/**'
      - 'docs/prd/**'

jobs:
  mechanical-checks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0   # full history — the diffs against origin/main below need it

      - name: Spec format check
        if: contains(github.event.pull_request.labels.*.name, 'feature')
        run: ./scripts/spec-format-check.sh specs/*/spec.md

      - name: PII scan on research
        run: |
          for f in $(git diff --name-only origin/main...HEAD -- docs/research/); do
            if [ -f "$f" ]; then ./scripts/pii-scan.sh "$f"; fi
          done

      - name: ADR Accepted body unchanged
        run: ./scripts/adr-frozen-check.sh --ci

  trio-audit:
    runs-on: ubuntu-latest
    continue-on-error: true   # advisory: LLM evaluators advise, they don't block (golden rule 8)
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Run trio-auditor headless
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          # Find the changed spec folder (the on.pull_request.paths filter already
          # scoped this workflow to relevant PRs)
          SPEC_DIR=$(git diff --name-only origin/main...HEAD -- specs/ | xargs -I{} dirname {} | sort -u | head -1)
          if [ -n "$SPEC_DIR" ]; then
            claude -p "Run the trio consistency audit from .claude/agents/trio-auditor.md on $SPEC_DIR. Output the findings table in markdown." > trio-audit.md
            cat trio-audit.md >> $GITHUB_STEP_SUMMARY
          fi
```

(`claude -p` is headless mode — the same trio-auditor subagent definition you'd invoke locally, driven by a prompt from CI. It needs `ANTHROPIC_API_KEY` in the repo's secrets.)

**Trade-offs:**

- Pros: team-wide enforcement; catches PRs from any source; runs in clean environment
- Cons: slower feedback (PR-level, not commit-level); needs API credentials in CI; can become a bottleneck if too strict

---

## Worked example — complete setup for one repo

Putting all five patterns together for a single mid-size SDD repo. This is **opinionated**, not exhaustive; adjust for your context.

### `.claude/settings.json` (Claude Code hooks)

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "[ \"${SDD_BYPASS_ADR:-0}\" = \"1\" ] && exit 0; f=$(jq -r '.tool_input.file_path // empty'); if [[ \"$f\" == *docs/adr/* ]] && grep -q '^Status: Accepted' \"$f\" 2>/dev/null; then echo 'Blocked: ADR Accepted — header only. Use Supersedes.' >&2; exit 2; fi"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "f=$(jq -r '.tool_input.file_path // empty'); case \"$f\" in *specs/*/spec.md) ./scripts/spec-format-check.sh \"$f\" >&2 || exit 2 ;; esac"
          }
        ]
      }
    ],
    "UserPromptSubmit": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Active ADRs:' && grep -l '^Status: Accepted' docs/adr/*.md 2>/dev/null | sed 's|docs/adr/||'"
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "if git status --porcelain specs/ docs/adr/ | grep -q .; then echo 'Reminder: uncommitted changes in specs/ or docs/adr/.' >&2; fi"
          }
        ]
      }
    ]
  }
}
```

(Hook payloads arrive as JSON on stdin — hence the `jq`. The ADR block exits 2 so the agent sees why it was stopped, and `SDD_BYPASS_ADR=1` is its documented escape hatch, same as in Pattern B.)

### `.claude/agents/trio-auditor.md` (LLM evaluator)

(See the worked example earlier in Pattern C.)

### `.claude/commands/spec-check.md` and `.claude/commands/sdd-6-trio-check.md`

(See the slash command example in Pattern D.)

### `.pre-commit-config.yaml` (mechanical, pre-commit framework)

(See Pattern A.)

### `.github/workflows/sdd-check.yml` (CI)

(See Pattern E.)

### Summary of what fires when

| When | Layer | What runs |
|------|-------|-----------|
| Agent edits an ADR | Claude Code PreToolUse hook | Block if Status: Accepted |
| Agent edits a spec | Claude Code PostToolUse hook | Warn if format issues |
| User submits a prompt | Claude Code UserPromptSubmit hook | Inject active ADR list |
| Session ends | Claude Code Stop hook | Remind about uncommitted specs |
| User commits | Pre-commit hooks | PII scan, ADR body check, spec format |
| PR opened or updated | GitHub Actions | Mechanical checks + trio-auditor LLM run |
| User explicitly asks | `/spec-check <path>`, `/sdd-6-trio-check` | On-demand evaluation |
| Implementation kickoff | `trio-auditor` subagent (manual or via `/sdd-6-trio-check`) | Full pre-implementation gate |

Each layer catches what the others miss. None alone is sufficient; together they're robust without being suffocating.

---

## What to mechanize vs leave human

The hardest design choice in this whole topic. The honest split:

### Mechanize when

- **The check is fully specifiable** — regex-able, structure-checkable
- **False positives are cheap** — minor annoyance, no work blocked
- **False negatives are expensive** — PII leak, lost history, broken invariant
- **The check fires frequently enough** to justify setup cost (otherwise it's premature automation)

### LLM-evaluate when

- **The check needs language understanding** but follows clear rules
- **False positives are tolerable** (a warning, not a block)
- **Token cost is justified** by check frequency × consequence

### Leave human when

- **Judgment matters** — quality, completeness, intent
- **Context is essential** — *"this is feature work, not new era"*
- **False positives would degrade the work itself** (over-strict review)
- **The check is too rare to justify automation**

### Frequency vs cost matrix

|  | Cheap to mechanize | Expensive to mechanize |
|---|---|---|
| **High-frequency check** | **Mechanize aggressively** (every commit needs this) | **Build a subagent / slash command** |
| **Low-frequency check** | Mechanize if trivial, otherwise human | **Just keep it human** — automation cost > savings |

The most common mistake: mechanizing infrequent checks because *"we can"*. Each automation has ongoing cost (false positives, maintenance, alert fatigue). Pay it only where the math works.

---

## Anti-patterns

### 1. Over-strict pre-commit hooks

Every commit fights the hooks. Team members start using `--no-verify` routinely. The discipline becomes worse than no discipline because the appearance of enforcement is now misleading.

**Fix:** restrict mechanical enforcement to checks where false positives are genuinely rare. Move judgment-needing checks to LLM evaluator (warn, not block) or human review.

### 2. No escape hatch

A hook blocks all edits to `docs/adr/`. Now you can't fix a typo in an Accepted ADR (which is technically allowed under the rule — only body changes are blocked). You end up disabling the hook entirely.

**Fix:** Every blocking hook should have a documented escape mechanism. Environment variable (`SDD_BYPASS_ADR=1`), commit message tag (`[skip-adr-check]`), explicit user gesture. The escape must be visible (logged, tracked) but available.

### 3. Mechanical checks of subjective things

A regex tries to detect *"is this AC testable?"*. False positive rate is huge because language can be tricky. Team's trust in the system erodes.

**Fix:** subjective quality is for the LLM evaluator (probabilistic but better than regex on language) or human review. Don't ask a regex to do a human's job.

### 4. Alert fatigue

Hooks print warnings on every commit. After a week, nobody reads them. The most important warning gets lost in noise.

**Fix:** warnings only for things humans should act on. Use stdout for "I did the thing"; reserve stderr (visible to the agent and user) for genuine issues.

### 5. Hook scope creep

A spec-format-check hook starts at 5 lines. Six months later it's 200 lines, checking arbitrary properties of unrelated things. Nobody understands what it does anymore.

**Fix:** one hook per cohesive purpose. When a hook grows past ~30 lines, split it.

### 6. Hooks that need network access

A pre-commit hook calls an external API. Now `git commit` requires network. Air-gapped or offline developers can't commit.

**Fix:** mechanical checks should be local-only. Network-requiring checks belong in CI, not pre-commit.

### 7. Hooks that take more than a few hundred milliseconds

Slow hooks add to every commit/edit. Team finds workarounds.

**Fix:** measure hook timing. Anything > 500ms is too slow for `PreToolUse` or pre-commit. Move slow work to PostToolUse, CI, or background subagents.

### 8. The subagent that no one invokes

A `trio-auditor` exists in `.claude/agents/`. Nobody triggers it. Audits don't happen.

**Fix:** wire it into a slash command (`/sdd-6-trio-check`) and a CI step. Don't rely on humans remembering to invoke it.

### 9. CI checks that block PRs on flaky LLM evaluations

A required CI check uses an LLM evaluator. Occasionally the evaluator is wrong. PRs are blocked despite being correct. Team becomes hostile to the discipline.

**Fix:** LLM evaluators in CI should advise, not block. Use them as PR comments / annotations, not as required status checks. Keep required checks to mechanical-only.

### 10. Automation without human counterpart

Every check is automated. Code review becomes a rubber stamp because "the bots checked it." Subtle quality issues slip through because no human looked deeply.

**Fix:** automation is *layered with* review, not a *replacement for* it. The reviewer's job evolves (focus on judgment items) but doesn't disappear.

---

## Golden rules

1. **Three categories, three tools.** Mechanical → regex/lint/hook. LLM → subagent / slash command. Judgment → human review. Don't blur the lines.

2. **Block only when false positives are cheap.** Asymmetric cost rules: PII leak (block hard), spec-with-vague-ACs (warn, don't block).

3. **Warnings beat blocks for subjective things.** A blocked PR over an arguable point is worse than no check.

4. **Every blocking gate needs an escape hatch.** Document it. If it's secret, it'll be misused.

5. **Mechanize only when the check fires often enough to justify the setup cost.** Premature automation is technical debt.

6. **One hook per purpose.** Split when growing past ~30 lines.

7. **The subagent that nobody invokes does nothing.** Wire it into a slash command, a hook, or CI.

8. **LLM evaluators advise; they don't block.** Required status checks should be mechanical only.

9. **Automation is layered with review, not a replacement.** Reviewers focus on judgment when machines handle mechanics.

10. **The goal is durable discipline, not maximum enforcement.** A team that thinks about SDD outperforms a team that's mechanically forced into compliance.

---

*This guide complements [`working-with-agents-guide.md` § "Claude Code Building Blocks"](working-with-agents-guide.md#claude-code-building-blocks) (the building blocks themselves), [`spec-plan-tasks-guide.md` § "Cross-artifact consistency checks"](spec-plan-tasks-guide.md#cross-artifact-consistency-checks) (the trio audit conceptually), [`adr-guide.md`](adr-guide.md) (the ADR immutability invariant). Together they describe what to enforce and how; this guide is the *how to make it stick* across patterns.*
