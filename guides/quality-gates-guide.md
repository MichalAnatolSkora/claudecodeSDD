# Quality Gates: Enforcing and Evaluating SDD

> How to make the discipline stick without depending on everyone remembering every rule — and where automation backfires. For a team of 1–10 the honest answer is mostly *"good docs plus one human on the PR"*; this guide is the lightweight version of everything beyond that, and the repo's **one home for Claude Code hooks**.

---

## Table of Contents

1. [For most 1–10 teams: the lightweight version](#for-most-110-teams-the-lightweight-version)
2. [Three kinds of checks](#three-kinds-of-checks)
3. [What to mechanize vs leave human](#what-to-mechanize-vs-leave-human)
4. [Where each check runs](#where-each-check-runs)
5. [Pre-commit hooks (mechanical, team-wide)](#pre-commit-hooks-mechanical-team-wide)
6. [Claude Code hooks (in-session)](#claude-code-hooks-in-session)
7. [When you outgrow ~10](#when-you-outgrow-10)
8. [Golden rules](#golden-rules)

---

## For most 1–10 teams: the lightweight version

**You almost certainly don't need much automation.** For a solo dev or a 2–3 person team, the whole of "making SDD stick" is three things you already have:

- **Good docs the agent actually reads** — a fresh `CLAUDE.md`, a spec before each change. The gate that matters most, and it needs no automation.
- **One human on the PR** — a second pair of eyes reading the diff against the spec's acceptance criteria. Judgment, not tooling (see [`sdd-in-teams-guide.md`](sdd-in-teams-guide.md)).
- **The break-the-code check** — already built into `/sdd-7-implement` ([`testing-guide.md`](testing-guide.md)). It catches confidently-wrong green suites for free.

Add exactly one gate when you feel a specific pain — and only that one:

- the first time someone nearly commits a customer name → the `pii-scan` [pre-commit hook](#pre-commit-hooks-mechanical-team-wide);
- the third time a trio ships with an uncovered acceptance criterion → make [`/sdd-6-trio-check`](../templates/.claude/commands/sdd-6-trio-check.md) a habit before `/sdd-7-implement` (it already ships);
- the first time the agent edits the body of an Accepted ADR → the ADR lock [Claude Code hook](#claude-code-hooks-in-session).

Nothing else until it earns its place. The rest of this guide is how to pick and build those gates without the automation backfiring.

---

## Three kinds of checks

Different checks need different tools. The mistake most teams make is automating something that needs judgment — or leaving something mechanical to vigilance.

| Kind | What it can check | Runs as | Strength | Weakness |
|------|-------------------|---------|----------|----------|
| **Mechanical** (regex, structure, format) | A required section exists; ACs are `- [ ]` checkboxes; PII patterns; the body of an Accepted ADR changed | Pre-commit hook, Claude Code hook, CI script | Deterministic, fast, no token cost | Presence and pattern only — it can't judge quality |
| **LLM evaluator** (semantic, rules plus judgment) | Are the ACs testable? Does implementation detail leak into the spec? Does every AC have a task? Does the plan contradict an ADR? | The shipped read-only commands — `/sdd-3-spec-review`, `/sdd-4-plan-validate`, `/sdd-6-trio-check` | Reads meaning; catches what regex can't | Probabilistic — verdicts on judgment calls vary between runs of the same audit; costs tokens |
| **Human** (judgment, context, intent) | Are the ACs *complete*? Is Out of scope enough to prevent drift? Are an ADR's rejected alternatives real or straw men? Does the spec serve the actual user need? | PR review | Captures intent and context | Doesn't scale; gets shortcut under pressure |

**Block or warn? The asymmetric-cost rule decides.** An enforcement check that wrongly fires becomes a workaround magnet (commits with `--no-verify`, hooks edited around); an evaluation that wrongly fires becomes noise nobody reads.

| If a false positive is… | Then | Example |
|-------------------------|------|---------|
| Cheap (one extra look) | Block | An edit to the body of an Accepted ADR |
| Expensive (blocks legitimate work) | Warn, don't block | *"These ACs look untestable — please review"* |
| Worth it, because a false negative is catastrophic | Block hard, accept some noise | PII in committed research notes |

---

## What to mechanize vs leave human

The hardest design choice in this whole topic. The honest split:

- **Mechanize when** the check is fully specifiable (regex-able, structure-checkable), false positives are cheap, false negatives are expensive (a PII leak, lost history, a broken invariant), and it fires often enough to justify the setup.
- **LLM-evaluate when** the check needs language understanding but follows clear rules, a warning (not a block) is acceptable, and the token cost is justified by frequency × consequence.
- **Leave it human when** judgment, completeness, or intent is the point; when context is essential (*"this is feature work, not a new era"*); when a false positive would degrade the work itself; or when the check is too rare to justify automating.

|  | Cheap to mechanize | Expensive to mechanize |
|---|---|---|
| **High-frequency check** | **Mechanize** (every commit needs it) | **A read-only command** (the shipped audits) |
| **Low-frequency check** | Mechanize if trivial, otherwise human | **Just keep it human** — automation cost > savings |

The most common mistake: mechanizing infrequent checks because *"we can."* Each automation carries ongoing cost — false positives, maintenance, alert fatigue. Pay it only where the math works.

---

## Where each check runs

Pick the earliest place that actually sees the mistake.

| Where | Fires when | Good for | Cost |
|-------|------------|----------|------|
| **Pre-commit hook** | `git commit`, from any editor | Mechanical checks, for every contributor — human or agent | Needs `pre-commit install` per contributor; bypassable with `--no-verify` |
| **Claude Code hook** | The agent's own tool calls, in-session | The same mechanical checks, with feedback the agent can act on immediately | Only fires when the agent is the editor — a human in vim walks past it |
| **Slash command** | When you run it | LLM evaluation — `/sdd-3-spec-review`, `/sdd-4-plan-validate`, `/sdd-6-trio-check` | Someone has to run it (`/sdd-7-implement` refuses a trio that clearly skipped `/sdd-6-trio-check`); uses the main session's context |
| **CI** | Every PR, from any source | Team-wide invariants | Slower feedback; LLM steps need API credentials — see [When you outgrow ~10](#when-you-outgrow-10) |

---

## Pre-commit hooks (mechanical, team-wide)

The earliest gate: catch issues before they enter the repo, whoever made them. Use the pre-commit framework (`pre-commit.com`) or hand-written Git hooks (`.git/hooks/pre-commit`).

**`.pre-commit-config.yaml`:**

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

**`scripts/pii-scan.sh`:**

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

**`scripts/adr-frozen-check.sh`:**

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

**`scripts/spec-format-check.sh`:**

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

**Trade-offs:** catches issues earliest and works for everyone, Claude Code user or not — but it runs only on `git commit` (not on the agent's edits in real time), needs `pre-commit install` per contributor, and can be bypassed with `--no-verify`.

---

## Claude Code hooks (in-session)

Hooks are automation the **harness** runs on the agent's events — not the agent. They live in `.claude/settings.json` (project — commit it, so the whole team gets them) or `~/.claude/settings.json` (just you). A hook can **block** an action, **feed context** back to the agent, or **run a side effect**. This section is the repo's single reference for them; other guides link here.

*(Claude Code's hook interface evolves — check event names and exit-code semantics against your version's docs before wiring these up.)*

### Mechanics

| Event | Fires when | Typical SDD use |
|-------|------------|-----------------|
| `PreToolUse` | Before a tool call | Block an action — an edit to an Accepted ADR, a destructive command |
| `PostToolUse` | After a tool call completes | Check the file just written and tell the agent; run side effects |
| `UserPromptSubmit` | You submit a prompt | Inject context — the active ADR list |
| `Stop` | The main agent finishes responding — **every turn**, not session end | Per-turn checks: failing tests, uncommitted work |
| `SubagentStop` | A subagent finishes | Validate its report |
| `SessionEnd` | The session terminates | Reminders and logging — output is *not* fed to the agent |
| `Notification` | The agent emits a notification | Forward or log it |

- **Input** arrives as JSON on **stdin** — `session_id`, `tool_name`, `tool_input` (with `file_path`, `command`, …). Read it with `jq`: `f=$(jq -r '.tool_input.file_path // empty')`. `$CLAUDE_PROJECT_DIR` holds the absolute project root, handy for referencing scripts (`"$CLAUDE_PROJECT_DIR/scripts/check.sh"`).
- **Exit codes carry meaning.** `0` allows. `2` is the blocking error: its stderr is fed back to the agent (on `PreToolUse` the tool call is blocked; on `PostToolUse` the tool already ran, but the agent sees why). Any *other* non-zero exit is a non-blocking error shown to you only — the agent never sees it.
- **Matchers** are regex on the tool name: `Edit|Write` matches both, `Bash` matches every shell call, `""` matches everything.
- **Fast:** a hook adds its runtime to every matching tool call — aim for well under 100 ms; push slow work into a command or CI.
- **Loud:** a hook that fails silently is invisible damage — the agent thinks the convention is enforced, your repo says otherwise. Log to stderr, and tee to a file while debugging (`>> ~/.claude/hooks.log 2>&1`).
- **Committed, or it isn't a rule:** hooks in your user settings are personal preference; team invariants go in the project `.claude/settings.json`.

### The SDD hooks, in one `settings.json`

Take only the ones you need — usually the ADR lock first.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "[ \"${SDD_BYPASS_ADR:-0}\" = \"1\" ] && exit 0; f=$(jq -r '.tool_input.file_path // empty'); if [[ \"$f\" == *docs/adr/* ]] && grep -q '^Status: Accepted' \"$f\" 2>/dev/null; then echo 'Blocked: ADR is Accepted. Create a Supersedes ADR instead.' >&2; exit 2; fi"
          }
        ]
      },
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "cmd=$(jq -r '.tool_input.command // empty'); if echo \"$cmd\" | grep -qE 'rm[[:space:]]+-rf|git[[:space:]]+push.*--force'; then echo 'Blocked: destructive command. If intended, run manually.' >&2; exit 2; fi"
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
            "command": "echo 'Active ADRs (from docs/adr/, Status: Accepted only):' && grep -l '^Status: Accepted' docs/adr/*.md 2>/dev/null | sed 's|docs/adr/||'"
          }
        ]
      }
    ],
    "SessionEnd": [
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

| Hook | Event | What it does |
|------|-------|--------------|
| **ADR lock** | `PreToolUse` on `Edit` | Blocks edits to an ADR with `Status: Accepted`; exit 2 tells the agent to write a superseding ADR instead. It can't tell a Status-header flip from a body edit, so a supersede's header change is yours to make — or set `SDD_BYPASS_ADR=1` for that session, the documented escape hatch (visible, deliberate, no hook-disabling). The pre-commit `adr-frozen-check.sh` above is the finer-grained net for human commits. |
| **Destructive-command block** | `PreToolUse` on `Bash` | Stops `rm -rf` and `git push --force`; if you really mean it, run it yourself. |
| **Spec format feedback** | `PostToolUse` on `Edit` / `Write` | Runs `scripts/spec-format-check.sh` on a spec the agent just wrote. Exit 2 is deliberate: the edit has already happened, but exit 2 is the only code that feeds the message back to the agent so it can fix the file. |
| **Active ADR list** | `UserPromptSubmit` | Prints the Accepted ADRs before every prompt, so the agent doesn't lose track of one mid-task. |
| **Uncommitted-docs reminder** | `SessionEnd` | A nudge for *you* (terminal or log) about uncommitted `specs/` or `docs/adr/` changes — the agent never sees it. Want the agent to act on it? Use `Stop` with exit 2 instead — but `Stop` fires after *every* response, not once per session. |

The same mechanics run non-gating side effects too — `markdownlint` after a doc edit, regenerating a PDF when a guide changes — on `PostToolUse`, ending in `|| true` so they never block.

---

## When you outgrow ~10

Past ~10 people — contributors who don't share a memory, a real CI pipeline, ADR bodies worth protecting on every PR — a few more pieces start to pay. Named here, not documented, on purpose:

- **The same scripts in CI.** The pre-commit scripts above run unchanged as required PR checks (`adr-frozen-check.sh --ci` diffs against `origin/main`; check out with full history — `fetch-depth: 0` on GitHub Actions — so that ref exists). Pre-commit only catches local commits; CI catches every PR from any source.
- **An advisory LLM audit on PRs.** Run the trio-check prompt headless (`claude -p` with the body of `sdd-6-trio-check.md`, pointed at the changed spec folder) and post the result as a PR comment or step summary — never as a required check ([golden rule 3](#golden-rules)).
- **A configured audit subagent** (`.claude/agents/`) once the audit's file reading would swamp the main session — where it lives and when it pays: [`working-with-agents-guide.md` § Claude Code Building Blocks](working-with-agents-guide.md#claude-code-building-blocks).

Lift the one piece you need, not the whole apparatus.

---

## Golden rules

Each rule, and the anti-pattern it prevents:

1. **Block only when false positives are cheap.** Over-strict hooks make `--no-verify` routine, and then the *appearance* of enforcement is worse than none.
2. **Warn, don't block, on anything subjective — and never ask a regex to judge.** A regex trying to decide *"is this AC testable?"* misfires constantly and erodes trust in every other check.
3. **LLM evaluators advise; they never block.** A required CI check on a flaky LLM verdict blocks correct PRs and turns the team against the discipline. Required checks are mechanical only.
4. **Every blocking gate has a visible escape hatch.** An environment variable (`SDD_BYPASS_ADR=1`), a commit tag, an explicit gesture — logged, not secret. Without one, the hook gets disabled the first time it blocks a legitimate fix.
5. **Few, quiet, fast, local.** Warnings only for things someone must act on (alert fatigue buries the one that matters); no network access in pre-commit (offline developers can't commit); nothing slower than a few hundred milliseconds.
6. **One hook per purpose.** A 5-line check that grows to 200 lines nobody understands has crept; split it past ~30 lines.
7. **An evaluator nobody runs does nothing.** Wire it into a command, a hook, or CI — don't rely on people remembering.
8. **Mechanize only checks that fire often enough to pay for their setup.** Premature automation is technical debt.
9. **Automation is layered with review, not a replacement.** When "the bots checked it" turns review into a rubber stamp, the subtle issues slip through; the reviewer's job shifts to judgment, it doesn't disappear.
10. **The goal is durable discipline, not maximum enforcement.** A team that thinks about SDD outperforms a team that's mechanically forced into compliance.

---

*This guide complements [`spec-plan-tasks-guide.md` § "Cross-artifact consistency checks"](spec-plan-tasks-guide.md#cross-artifact-consistency-checks) (what the trio audit checks), [`adr-guide.md`](adr-guide.md) (the immutability rule the ADR lock enforces), and [`working-with-agents-guide.md` § "Claude Code Building Blocks"](working-with-agents-guide.md#claude-code-building-blocks) (when to reach for a hook vs a command, skill, or subagent).*
