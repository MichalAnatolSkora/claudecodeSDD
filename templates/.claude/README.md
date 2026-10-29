# SDD commands (copy into your project's `.claude/`)

Copy-pasteable Claude Code slash commands for the whole SDD loop: idea → PRD → features → **spec → plan → tasks** → check → build. Four commands do the per-change work, plus one you run once per project. See [`guides/spec-plan-tasks-guide.md`](../../guides/spec-plan-tasks-guide.md) and [`guides/flow-guide.md`](../../guides/flow-guide.md) for the whole flow.

## Get them

Drop the whole `.claude/` into your repo (commands **+ this cheat sheet**):

```bash
npx degit MichalAnatolSkora/claudecodeSDD/templates/.claude .claude
```

## Cheat sheet

`<...>` is required input; `[...]` is optional — with no path, a command uses the most recent spec under `specs/` (or the PRD under `docs/prd/`) and tells you which.

| Command | What it does |
|---------|--------------|
| `/sdd-prd [idea \| PoC path \| PRD path]` | Draft a lean PRD from a 1–3 sentence idea, or backfill one from a kept PoC (never a spec) — then, on an accepted PRD, slice it into the `## Feature breakdown` (run it again to re-slice) |
| `/sdd-spec [idea \| spec path \| tasks to add]` | Write the next missing piece of a feature's trio, in order — spec, then plan, then tasks — showing each draft before saving; on a finished trio, insert tasks |
| `/sdd-check [path]` | Read-only audit of whatever exists — PRD, spec, plan vs ADRs + `ARCHITECTURE.md`, trio consistency. The gate before code |
| `/sdd-build [path]` | Work `tasks.md` red→green, commit per task, then the break-the-code check |
| `/sdd-architecture [path]` | **Once per project:** `ARCHITECTURE.md` + an ADR per hard-to-reverse decision — read off existing code, or decided by Q&A from the PRD (greenfield) |

**Full run:** `/sdd-prd` → `/sdd-check` → `/sdd-prd` (slice) → per feature: `/sdd-spec` (spec) → `/sdd-check` → `/sdd-spec` (plan) → `/sdd-spec` (tasks) → `/sdd-check` → `/sdd-build`.

**Enter where your change starts, skip the rest** — most work isn't the whole run. A bugfix is often a short `/sdd-spec` then `/sdd-build`; a known single feature starts at `/sdd-spec`.

**When work loops** — no extra commands, three rules:

- **Can't write the acceptance criteria yet?** Spike first on a throwaway branch, then `/sdd-spec` from what it taught.
- **The code shows an AC was wrong?** Fix the spec before merge — unmerged means not frozen; a dated line in the trio's Notes says why; re-run `/sdd-check`.
- **A shipped slice changed the backlog?** `/sdd-prd` re-slices the PRD's Feature breakdown.

No status fields anywhere: git is the status. A spec on a branch is editable; merged, it's frozen history.

These assume the standard SDD layout (`specs/`, `templates/`, `docs/prd/`, `docs/adr/`, `ARCHITECTURE.md`, `CLAUDE.md`). Adjust the paths inside each command file to match your project.
