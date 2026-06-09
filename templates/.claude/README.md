# SDD trio commands (copy into your project's `.claude/`)

Copy-pasteable Claude Code slash commands for the full SDD loop: idea → PRD → features → **spec → plan → tasks** → check → implement. Every command is namespaced `sdd-` and carries its **phase number**, so `/sdd` + Tab lists them in flow order and you can always see which phase you're in. See [`guides/spec-plan-tasks-guide.md`](../../guides/spec-plan-tasks-guide.md) § "Slash commands worth having" and [`guides/flow-guide.md`](../../guides/flow-guide.md) for the whole flow.

## Get them

Drop the whole `.claude/` into your repo (commands **+ this cheat sheet**):

```bash
npx degit MichalAnatolSkora/claudecodeSDD/templates/.claude .claude
```

Or copy `commands/` by hand:

```
your-project/.claude/
├── README.md                       # this cheat sheet
└── commands/
    ├── sdd-1-prd-new.md
    ├── sdd-2-features-from-prd.md
    ├── sdd-3-spec-new.md
    ├── sdd-3-spec-review.md
    ├── sdd-4-plan-from-spec.md
    ├── sdd-4-plan-validate.md
    ├── sdd-5-tasks-from-plan.md
    ├── sdd-5-tasks-add.md
    ├── sdd-6-trio-check.md
    └── sdd-7-implement.md
```

## Cheat sheet — commands by phase

The number **is** the phase in the flow. `<...>` is required input; `[path]` is optional and defaults to the most recent folder under `specs/` (or the PRD under `docs/prd/`).

| Phase | Command | What it does |
|-------|---------|--------------|
| **1 · PRD** | `/sdd-1-prd-new <1–3 sentences>` | Sketch a lean PRD from a seed idea, then fill the gaps by Q&A |
| **2 · Features** | `/sdd-2-features-from-prd [path]` | Slice an accepted PRD into a prioritized, vertically-sliced feature list (read-only) |
| **3 · Spec** | `/sdd-3-spec-new <idea>` | Draft a `spec.md` from a one-paragraph idea |
| | `/sdd-3-spec-review [path]` | Audit a spec for gaps (read-only) |
| **4 · Plan** | `/sdd-4-plan-from-spec [path]` | Draft `plan.md` from a reviewed spec |
| | `/sdd-4-plan-validate [path]` | Check the plan against ADRs + `ARCHITECTURE.md` (read-only) |
| **5 · Tasks** | `/sdd-5-tasks-from-plan [path]` | Draft an ordered, verifiable `tasks.md` |
| | `/sdd-5-tasks-add <what> [path]` | Append/insert task(s) into an existing `tasks.md` (or a one-file trio's Tasks section) |
| **6 · Check** | `/sdd-6-trio-check [path]` | Final consistency audit across spec/plan/tasks — the gate before code (read-only) |
| **7 · Implement** | `/sdd-7-implement [path]` | Work `tasks.md` red→green, commit per task, then the break-the-code check |

**Full run:** `/sdd-1-prd-new` → `/sdd-2-features-from-prd` → per feature: `/sdd-3-spec-new` → `/sdd-3-spec-review` → `/sdd-4-plan-from-spec` → `/sdd-4-plan-validate` → `/sdd-5-tasks-from-plan` → `/sdd-6-trio-check` → `/sdd-7-implement`.

**Enter where your change starts, skip the rest** — most work isn't the whole pipeline. A bugfix is often just `/sdd-3-spec-new` then `/sdd-7-implement`; a known single feature starts at phase 3.

These assume the standard SDD layout (`specs/`, `templates/`, `docs/adr/`, `ARCHITECTURE.md`, `CLAUDE.md`). Adjust the paths inside each command file to match your project.
