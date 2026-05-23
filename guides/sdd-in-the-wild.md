# SDD in the Wild: Who Practices These Principles

> Notes on teams and companies that publicly endorse spec-driven (or spec-adjacent) engineering — the "write structured docs before code" discipline that SDD codifies for the AI-coding-agent era.

---

## Framing

The phrase *"spec-driven development"* is recent — it crystallized in 2024–2025 as AI coding agents (Claude Code, Copilot, Cursor) made *structured context up front* visibly profitable. The term gets used most often in the context of working with agents.

But the underlying principles — *write the design before the code, capture decisions as they're made, treat docs as a first-class artifact* — are decades old and practiced (under different names) by many engineering organizations. This page documents the explicit AI-era SDD endorsements plus the legacy practices that share the same DNA.

This is a working notes page, not a definitive survey. Adoption shifts fast; treat the list as a starting point.

---

## Explicit AI-era SDD adopters

These organizations have publicly endorsed SDD (or a near-identical workflow) specifically for AI-augmented engineering.

### GitHub (Microsoft) — [`github/spec-kit`](https://github.com/github/spec-kit)

The most direct, named SDD reference implementation. GitHub published `spec-kit` in August 2025 as a "toolkit to help you get started with Spec-Driven Development" — literal phrasing. The workflow is `spec.md → plan.md → tasks.md`, paired with Copilot to ground generated code in the spec.

The repo has accumulated 100k+ stars in under a year (verified at time of writing), which makes it the most visible SDD artifact in the ecosystem. If you're pitching SDD internally, this is exhibit A: *"GitHub themselves ship this."*

### Anthropic — Claude Code + `CLAUDE.md`

Anthropic's Claude Code product treats a repo-level instruction file (`CLAUDE.md`) as a first-class concept. Their official engineering write-ups on Claude Code best practices cover the spec-driven shape directly: hub file in the repo, "plan before code" prompting, structured handoffs between sessions.

The Claude Code documentation and engineering blog are the canonical references; the practice they describe maps almost 1:1 to the SDD workflow described in this repo.

### Cursor (Anysphere) — `.cursorrules`

Cursor's `.cursorrules` files are functionally equivalent to `CLAUDE.md`: a repo-level conventions file the agent always reads before doing anything. The community-curated `awesome-cursorrules` catalog on GitHub has become a de-facto pattern library — different rule files for different stacks, all sharing the SDD assumption that *the agent needs explicit conventions, not its priors*.

Cursor's own documentation pushes the same message: *"write rules so the agent doesn't have to guess."*

### Aider, Continue.dev, Cline, and the broader agent-tool ecosystem

The convergence on "one file the agent always reads" has become table stakes:

- **Aider** — [conventions files](https://aider.chat/docs/usage/conventions.html) loaded into every session
- **Continue.dev** — `.continuerules` per repo
- **Cline** — `.clinerules`
- Various forks/wrappers — same pattern under different filenames

The fact that every major agent tool independently reinvented the same construct is itself the strongest signal that SDD-shaped workflows are not a fad.

---

## Pre-AI organizations practicing spec-adjacent discipline

These organizations practiced "write structured docs before code" long before AI coding agents existed. SDD is, in part, the AI-era repackaging of the same instinct — which means these are useful references when pitching the discipline to a team that hasn't yet adopted AI agents.

### Amazon — Working Backwards / PR-FAQ

New products at Amazon famously start with a **fake press release** plus an **FAQ document**. The press release describes the launched product from the customer's perspective; the FAQ pre-answers the hard questions. Code starts only after the PR-FAQ is reviewed and accepted.

Structurally this is a PRD written before any implementation. The "6-page memo" culture (no slides in meetings — write a structured doc, read it in silence for 20 minutes, then discuss) is the same discipline applied to decisions broader than products.

Reference: Colin Bryar & Bill Carr, *Working Backwards* (2021).

### Google — Design docs and RFCs

Design-doc culture is the default for non-trivial engineering work at Google. The shape — context, goals, non-goals, design, alternatives considered, security/privacy review, rollout — is essentially a PRD + ADR + plan in one. Templates and norms have been written about publicly; the most-cited write-up is Malte Ubl's [*Design docs at Google*](https://www.industrialempathy.com/posts/design-docs-at-google/).

Internal RFC processes formalize decision capture; ADRs in the open-source sense are a direct descendant.

### Stripe — RFC-driven engineering

Stripe is widely cited for written-engineering culture. Patrick McKenzie (formerly Stripe) and other ex-Stripe engineers have written publicly about the role of RFCs as engineering deliverables — long-form prose treated with the same rigor as code, not as overhead.

The practical effect: decisions of any consequence land in writing before code does, and the written artifact survives as institutional memory.

### Basecamp / 37signals — Shape Up

The [Shape Up](https://basecamp.com/shapeup) methodology (free book, available online) structures product work as **pitches → betting table → 6-week cycles → 2-week cool-down**. A pitch contains: problem, appetite, solution sketch, rabbit holes, no-gos — structurally near-identical to a modern SDD spec.

Shape Up predates AI coding agents but maps almost perfectly onto the SDD framing: write the pitch, then build only what's pitched.

### Microsoft (broader engineering practice)

Long history of design docs, post-mortems, and ADRs across product teams. GitHub (Microsoft-owned) productized this for the AI era with spec-kit + Copilot, so the AI-era and pre-AI-era references unify under the same parent company.

---

## Adjacent methodologies that informed SDD

These pre-date the SDD label but contributed underlying patterns. Pointing at them helps when a skeptic insists SDD is a new fad — most of the ideas have 10–50 years of track record.

- **IETF RFC process** (1969–present) — the original "write the spec before the code." Every internet standard starts as an RFC.
- **Architecture Decision Records (ADRs)** — formalized by Michael Nygard in 2011. Now standard tooling in most large engineering orgs (and the basis for the ADR layer in [`spec-driven-development-guide.md`](spec-driven-development-guide.md)).
- **C4 model** (Simon Brown) — structured architecture diagrams: Context → Container → Component → Code. Often pairs with ADRs in the same `docs/` folder.
- **Diátaxis framework** (Daniele Procida) — splits documentation into Tutorial / How-To / Reference / Explanation. Increasingly common in `docs/` folder structures.
- **Pre-mortems** (Gary Klein) — write the failure narrative before launching, as a forcing function for explicit risk capture. Maps onto the "Risks / Open questions" section of a spec.

---

## What we don't have data on

- **Most companies practice this discipline internally without publicly endorsing the term.** The list above is biased toward organizations with visible engineering blogs. Many regulated industries (finance, healthcare, defense) practice SDD-like discipline by necessity — auditability, compliance — but rarely publish.
- **Adoption depth varies.** *"We have an ADR folder"* is not the same as *"every change ships with a spec."* The references above mix both ends of the spectrum.
- **The AI-era angle is new.** Most data points for *SDD-as-a-named-methodology* are from 2024 onward. Older references (Amazon, Google, Stripe, Shape Up) practice spec-driven discipline without calling it SDD.
- **Open-source adoption is hard to measure.** A repo with a `CLAUDE.md` or `.cursorrules` might or might not be doing real spec-driven work — the file's presence is a weak signal at best.

---

## How to use this page

When pitching SDD internally — *"this isn't just a Claude Code fad"* — these references help. The underlying discipline is decades old; SDD is the operational form for AI-augmented teams. If a skeptical colleague asks *who else does this*, you have answers across both the AI-era and pre-AI eras.

If you find a public endorsement, blog post, or conference talk that belongs on this list, add it. This file is best maintained collaboratively because the landscape moves fast.

---

*This page is a companion to [`spec-driven-development-guide.md`](spec-driven-development-guide.md) (the methodology) and [`working-with-agents-guide.md`](working-with-agents-guide.md) (the practice). It exists so the reader can verify SDD's lineage and current footprint without leaving the repo.*
