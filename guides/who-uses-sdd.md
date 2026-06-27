# Who uses spec-driven development (and what they call it)

> Real teams and tools that do the same thing: **write down what you're building, before you build it.** Some call it spec-driven development; most call it something else. Handy when someone asks *"is this just a Claude Code fad?"* — it isn't.

---

## The short version

"Spec-driven development" is a new name (2024–2025, out of the AI-coding-agent world) for an old habit: **decide what you're building and why, in writing, before you write the code.** AI agents made the habit pay off visibly — give the agent structured context up front and it stops guessing — but the habit itself is decades old.

So two groups show up below: the new AI-coding tools that bake it in, and the engineering orgs that have done it for years under other names. None of the AI tools has nailed it, though — they land either *too generic* or *too binding* (more below).

This is a notes page, not a survey — adoption moves fast, so treat it as a starting point.

---

## AI-coding tools — who uses what

Every serious AI-coding tool landed on the same idea: **one file the agent always reads before it does anything.** That they all reinvented it separately is the strongest sign this isn't a fad.

| Tool / team | What they use | What it is |
|-------------|---------------|------------|
| **GitHub** (Microsoft) | [`spec-kit`](https://github.com/github/spec-kit) | A literal "Spec-Driven Development" toolkit: `SPEC.md → PLAN.md → TASKS.md`, driven by slash commands (`/constitution`, `/specify`, `/plan`, `/tasks`, `/implement`). Agent-agnostic — works with Copilot, Claude Code, Gemini CLI, Cursor, and others. The headline if you need to convince someone: *GitHub ships this themselves.* |
| **Anthropic** | Claude Code + `CLAUDE.md` | A repo-level instruction file the agent reads every session, plus Plan Mode / "plan before code" guidance. These are the **building blocks this repo builds on** — Claude Code ships the primitives (the instruction hub, planning), not the full PRD → spec → plan → tasks → ADR method, which this repo adds on top. |
| **AWS** | **Kiro** | A spec-driven agentic IDE: each feature gets `requirements.md` (EARS-style acceptance criteria) / `design.md` / `TASKS.md`, plus steering files for project conventions. The closest commercial cousin of this repo's trio. |
| **Cursor** | Project Rules (`.cursor/rules/*.mdc`) | Same idea as `CLAUDE.md` — conventions files the agent always reads. The older single-file `.cursorrules` is deprecated legacy. |
| **Cross-tool** | [`AGENTS.md`](https://agents.md) | A tool-neutral instructions-file convention, adopted by OpenAI Codex, Google Jules, Cursor, and many others. This repo ships one too — a thin pointer to `CLAUDE.md`. |
| **OpenSpec** | open-source workflow | A tool-agnostic spec-driven workflow for AI coding assistants: each change is a spec proposal (delta) reviewed before code and archived after it ships. |
| **Aider** | [conventions file](https://aider.chat/docs/usage/conventions.html) | Loaded when configured (`--read` / `.aider.conf.yml`). |
| **Continue.dev** | rules files (`.continue/rules/`, formerly `.continuerules`) | Per-repo rules. |
| **Cline** | `.clinerules` | Per-repo rules. |

Different filenames, same move: **tell the agent your conventions instead of letting it guess.**

---

## There's no single "SDD" — and every tool picks a side

Read that table again and the uncomfortable part is that **none of these is *the* answer.** "Spec-driven development" isn't one method with a canonical implementation — it's a loose family, and today's tools cluster at two unhappy extremes:

- **Too generic.** The rules-file tools (`CLAUDE.md`, `.cursor/rules`, `AGENTS.md`, `.clinerules`, Aider conventions) give you *one file the agent reads* — and stop there. That's the floor, not a method: nothing makes you write a spec before a plan, a plan before tasks, or freeze the spec at merge. You get a convention, not a loop.
- **Too binding.** The full products give you the loop but make you rent it. **Kiro** is an entire IDE — adopt it and your specs live in *its* `requirements.md` / `design.md` / `tasks.md` format, in *its* tool, on *its* terms. **spec-kit** is lighter (agent-agnostic, open source) but still ships a scaffolder and a fixed ceremony — `/constitution`, `/clarify`, `/analyze`, generated files — that you mostly take whole. A real method, with a real switching cost.

The middle — **structured enough to be a method, light enough not to own you** — is the gap this repo aims at: plain markdown you control, a spec → plan → tasks loop any file-reading agent can run, no scaffolder, no IDE, no format you can't walk away from. The discipline lives in *how you write three files*, not in a product you adopt.

That's the quiet payoff of staying this light: **simple enough to start in an afternoon, and cheap to leave.** The artifacts are just markdown you own, so you're never locked in — outgrow this and decide spec-kit's ceremony or Kiro's IDE is finally worth it, and your specs are already plain files you carry straight over. The migration only runs the easy way, too: markdown → tool is a copy; tool → markdown is an export you'll wish you never needed. So adopt the heavy thing *if and when* it earns its keep — not on day one, as a bet you can't unwind.

And there's a gap on a different axis entirely — not *how heavy* a tool is, but *when you're allowed to start*. The tools that are actually a method — spec-kit, Kiro — are all **spec-first**: they assume you begin by writing a spec (or a constitution, or a requirements doc) before any code. A new product usually doesn't begin there. It begins with a **PoC**, the smallest thing that proves the idea is worth pursuing, and only then is there anything worth specifying. No spec-first tool offers an honest way in from that proven prototype, so the most normal way real businesses start is the one case these tools skip. This repo doesn't skip it: `/sdd-1-prd-from-poc` reads the PoC and backfills a PRD from it — no faked specs, the spec-before-code discipline simply begins with your *next* change. **Start messy; formalize once it's earned.**

So the honest framing isn't *"SDD is solved — pick a tool."* It's: the idea is real and worth doing, the existing tools force a trade-off you don't have to accept, and markdown plus a habit gets you most of the way there.

---

## Engineering orgs — who did it before AI

These teams wrote structured docs before code long before AI agents existed. SDD is partly the AI-era repackaging of the same instinct — useful references when you're pitching it to a team that hasn't adopted agents yet.

| Org | What they call it | What it is |
|-----|-------------------|------------|
| **Amazon** | Working Backwards / **PR-FAQ** | A new product starts as a fake press release + an FAQ, reviewed *before* any code. Plus the "6-page memo, read in silence, then discuss" meeting culture. (Book: *Working Backwards*, 2021.) |
| **Google** | **Design docs** + RFCs | Context, goals, non-goals, design, alternatives, rollout — a PRD + ADR + plan in one. (See Malte Ubl, [*Design docs at Google*](https://www.industrialempathy.com/posts/design-docs-at-google/).) |
| **Stripe** | **RFCs** | Long-form written proposals, treated with the same rigor as code. Decisions land in writing before code does, and the doc survives as memory. |
| **Basecamp / 37signals** | **Shape Up** | Work starts from a *pitch* — problem, appetite, solution sketch, rabbit holes, no-gos — which is basically a modern spec. ([Free book](https://basecamp.com/shapeup).) |
| **Microsoft** (broadly) | Design docs, post-mortems, ADRs | A long history across teams; GitHub (Microsoft-owned) productized it for the AI era with spec-kit. |

---

## Older ideas SDD borrows from

The pieces have a 10–50 year track record — handy when someone calls SDD a new fad:

- **IETF RFCs** (1969–) — the original "write the spec before the code." Every internet standard starts as one.
- **ADRs** — Architecture Decision Records, formalized by Michael Nygard (2011). Now standard tooling, and the basis for this repo's ADR layer.
- **C4 model** (Simon Brown) — structured architecture diagrams: Context → Container → Component → Code.
- **Diátaxis** (Daniele Procida) — splits docs into Tutorial / How-To / Reference / Explanation.
- **Pre-mortems** (Gary Klein) — write the failure story *before* launch, to surface risks early. Maps to a spec's Risks / Open questions.

---

## What this list doesn't tell you

- **Most companies do this privately.** The list leans toward orgs with public engineering blogs. Regulated industries (finance, health, defense) do SDD-like work out of necessity but rarely publish.
- **Depth varies.** "We have an ADR folder" is not "every change ships with a spec." The list mixes both ends.
- **The AI-era name is new.** Amazon, Google, Stripe, and Basecamp have done this for years without calling it SDD.
- **A `CLAUDE.md` in a repo proves little** on its own — its presence is a weak signal that real spec-driven work is happening.

Found a public example that belongs here? Add it — the landscape moves fast, so this page is best kept up collaboratively.

---

*Companion to [`spec-driven-development-guide.md`](spec-driven-development-guide.md) (the method) and [`working-with-agents-guide.md`](working-with-agents-guide.md) (the practice). It's here so you can check SDD's lineage and footprint without leaving the repo.*
