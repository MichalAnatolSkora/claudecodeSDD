# Writing a Good PRD (Per Era, For Humans)

> The practical companion to [`spec-driven-development-guide.md` § "The PRD Layer"](spec-driven-development-guide.md#the-prd-layer). That section gives you the principles (PRD vs spec, the agent implements from specs not the PRD, the era pattern, the Frankenstein trap). This guide gives you the *practice*: formats, worked examples, era-boundary heuristics, AI-authoring prompts, and the review process.

---

## Table of Contents

1. [Why write a PRD — and when](#why-write-a-prd--and-when)
2. [What this guide adds](#what-this-guide-adds)
3. [PRD formats — pick the right shape](#prd-formats--pick-the-right-shape)
4. [Anatomy of a PRD, section by section](#anatomy-of-a-prd-section-by-section)
5. [Worked example 1 — Original launch (B2B billing platform)](#worked-example-1--original-launch-b2b-billing-platform)
6. [Worked example 2 — Era 2 (multi-currency expansion)](#worked-example-2--era-2-multi-currency-expansion)
7. [Era boundary heuristics — when do you need a new PRD?](#era-boundary-heuristics--when-do-you-need-a-new-prd)
8. [AI-assisted PRD authoring](#ai-assisted-prd-authoring)
9. [Success metrics — defining "what does success look like"](#success-metrics--defining-what-does-success-look-like)
10. [Stakeholder review process](#stakeholder-review-process)
11. [Cross-functional handoff — PRD to engineering](#cross-functional-handoff--prd-to-engineering)
    - [Slicing the PRD into features](#slicing-the-prd-into-features)
12. [PRD-specific anti-patterns](#prd-specific-anti-patterns)
13. [Golden rules](#golden-rules)

---

## Why write a PRD — and when

**What it's for.** A PRD (Product Requirements Document) is the product-level statement of *what we're building and why* — the problem, who has it, and what success looks like — agreed **before** anyone writes a spec or a line of code. It's the contract between intent and engineering. Humans read it; the agent never reads it *during implementation* — it implements from `SPEC.md` — though it does read the PRD when drafting, reviewing, era-checking, or slicing it. One PRD describes a whole product or a whole release *era*, not a single feature.

**Why it's worth the hour or two:**

- **The cheapest place to be wrong is a doc.** Discovering *"we're solving the wrong problem"* in a PRD costs an afternoon; discovering it in shipped code costs weeks.
- **It aligns everyone on one version of the goal.** PM, engineers, design, founder argue on paper — once — instead of in code review, repeatedly.
- **It freezes intent so it doesn't drift.** Without a written *what & why*, the goal quietly mutates feature by feature, and six months in nobody agrees what you're building.
- **It's the root of the SDD chain.** PRD → slice into features → `SPEC.md` → `PLAN.md` → `TASKS.md`. Skip it and the *codebase* becomes the de-facto spec — a pile of decisions nobody chose on purpose.

**When you actually need one.** A PRD earns its place when **more than one person has to agree**, or the bet is big enough that building the wrong thing is expensive. Skip it — or shrink it to a one-paragraph issue — when you're solo, the change is small, or the *what & why* already fits in your head. (When in doubt, the one-pager format below is the lightest real PRD.)

**When to write a *v2* (a new era).** A PRD **freezes after v1 ships** — you do *not* edit it for every new feature (features get specs, not PRD edits; an ever-edited PRD becomes a Frankenstein). You write a **new** PRD when the product's *direction* shifts materially: a new market or segment, a major new capability, a pivot, or a post-reorg change of direction. Routine features are not a new era. For the concrete test, see [Era boundary heuristics](#era-boundary-heuristics--when-do-you-need-a-new-prd) below; for the layered model (PRD vs spec, the three documentation layers), see [`spec-driven-development-guide.md` § "The PRD Layer"](spec-driven-development-guide.md#the-prd-layer).

---

## What this guide adds

The section above is the *why* and *when*. The main SDD guide places the PRD in the bigger picture — PRD vs spec, the three documentation layers.

This guide is the practice:

- **Multiple PRD formats** — PR-FAQ vs lean PRD vs one-pager vs full template — and when each fits
- **Anatomy section by section** with concrete advice on what to write (and not write) in each
- **Two complete worked examples** — what a real, well-shaped PRD actually looks like
- **Era boundary heuristics** — concrete signals that you're at a new era and should write a new PRD
- **AI-assisted authoring** — copy-pasteable prompts for drafting, reviewing, and translating a PRD into specs
- **Stakeholder review process** — who reviews, in what order, what they look for
- **Success metrics deep-dive** — how to write *"what success looks like"* measurably without overcommitting
- **Cross-functional handoff** — getting from *"PRD accepted"* to *"first specs in flight"*

For the underlying principles, go back to [`spec-driven-development-guide.md` § "The PRD Layer"](spec-driven-development-guide.md#the-prd-layer). For the upstream side — research and discovery that feeds the PRD — see [`research-guide.md`](research-guide.md). A PRD without grounding research is mostly opinion; a PRD that cites synthesized research is mostly evidence.

---

## PRD formats — pick the right shape

There's no single PRD format. For a team of **1–10**, two shapes cover almost everything — pick by *audience* and *stage*, and default to the one your readers will *actually read*:

| Format | When to use | Length | Watch out for |
|--------|-------------|--------|---------------|
| **One-pager** | Internal alignment for a small team, decision mostly made | 1 page | Can't carry rationale that outlives turnover |
| **Lean PRD** | The default for most features and most teams | 1–2 pages | Light on rationale if you rush it |

### One-pager

Single page covering: *problem, solution, target user, success metrics, what's NOT in this*. Useful when the decision is mostly already made and the doc exists to align a small group.

Avoid for major decisions — one page can't carry the rationale needed to outlive turnover.

### Lean PRD

Pioneered (under various names) by startup PMs — Lenny Rachitsky, Marty Cagan's *Inspired*, and YC-affiliated PM writing. Structure:

- One-paragraph problem
- One-paragraph solution
- Target user (who is this for, specifically)
- Success metrics (1–3 numbers — keep it tight)
- Risks and assumptions

Best default for most modern teams. The discipline of "one paragraph each" prevents bloat.

### Borrow the PR-FAQ trick for big bets

You rarely need the full Amazon "Working Backwards" PR-FAQ at 1–10, but its forcing function is worth stealing for an era-defining decision: **write the press release first** — headline, a hypothetical customer quote, what the product does — *as if it already shipped*. If you can't write a compelling one, the product isn't well-conceived yet. Drop it in as one extra section of a lean PRD; you don't need the whole ceremony (press release + full FAQ).

### When you outgrow this

Regulated industries, enterprise products, and multi-team coordination eventually want a **full PRD** — 8–20 pages with functional/non-functional requirements, compliance, timelines, and appendices for the audit trail. It's comprehensive but costs weeks, and it's deliberately out of scope here: for 1–10 people it's almost always more document than the decision needs. Reach for it only when an auditor or a dozen stakeholders force your hand — and trim it as far as compliance allows.

### Picking between formats

A rough rule for 1–10:

- **Solo / decision mostly made** → one-pager
- **Most features, most teams** → lean PRD (the default)
- **Era-defining bet** → lean PRD + the press-release section
- **Forced into enterprise / regulated** → see *When you outgrow this* above

The format isn't sacred. Mix and match — a lean PRD with a press-release section, a one-pager with a longer FAQ. Pick what your audience will *actually read*.

---

## Anatomy of a PRD, section by section

Regardless of format, most PRDs have the same underlying questions. The section list below follows [`templates/PRD.md`](../templates/PRD.md) — the canonical template in this repo: Problem / Users / Success criteria / In scope (v1) / Out of scope / Constraints / Risks and unknowns / References.

Two sanctioned variants exist, and that's fine. A `## Summary` opener (used in the worked examples below) is an optional courtesy for skimmers. And the `/sdd-1-prd-new` command produces a lean variant whose headings differ slightly. The underlying questions are what matters; the template's list is the default.

Here's how to think about each:

### Problem (1–2 paragraphs)

What pain are you solving, *for whom*, and how does it manifest today? A good problem statement has:

- A specific user or user segment (not *"users"* generically)
- A concrete behavior / cost / friction they experience now
- A hint at why this hasn't already been solved

**Bad:** *"Users want better dashboards."*
**Good:** *"Mid-market finance teams (50–500 employees) currently export bank transactions to CSV and manually reconcile against invoices in Excel, losing 8–12 hours per month per controller. Existing accounting software handles the bookkeeping but lacks the reconciliation workflow."*

### Users

Who exactly is this for? Specificity matters more than feels natural.

- Job title (often)
- Company size / org size (B2B)
- Demographic (consumer)
- Skill level / sophistication
- Frequency of use (daily? occasional?)

If you can't list the user precisely, the rest of the PRD will drift.

### Success criteria

How will you know this worked? See [§ "Success metrics — defining what does success look like"](#success-metrics--defining-what-does-success-look-like) below for the deep treatment. Short version: pick 1–3 measurable outcomes, with rough targets and timeframes.

### In scope (v1)

What you're going to build in this era, at the *product* level — not implementation. 3–5 sentences. Reference patterns the reader recognizes (*"like Stripe Dashboard, but for…"*) when it helps clarity.

Do **not** include:
- Specific libraries, frameworks, databases, ORMs
- File layouts, class names, API shapes
- Test strategies
- Anything an engineer would put in a spec or plan

If you find yourself writing those, you're writing a spec, not a PRD.

### Out of scope (deliberately, not now)

Just as important as scope. What you're *not* doing in this era — even if it's tempting.

This section saves more drift than any other. List anything a reasonable reader might assume is in scope:

- Adjacent features that look related but aren't
- Platforms you're not supporting
- User segments you're explicitly not targeting
- Integrations you'll skip
- "Future" capabilities someone might confuse with this era

### Constraints

What's fixed, regardless of preference:

- **Technical** — must integrate with existing system X, must work offline, must run on legacy browsers
- **Legal/regulatory** — GDPR, HIPAA, SOC 2, country-specific data residency
- **Business** — budget, deadline, headcount, vendor contracts
- **Cultural** — *"can't break the current self-service flow"*, *"can't require a sales call"*

Constraints define the option space the team must work within. Without them, the spec writers will propose solutions that don't fit reality.

### Risks and unknowns

What could make this fail? What are you taking on faith?

- *"We assume partner banks will sign data-sharing agreements within 6 months"*
- *"We're betting the SMB segment will pay $X/month for this"*
- *"Risk: existing integrations team is at capacity; this depends on contractor pipeline"*

Surfacing these makes the unspoken explicit. Often resolves disagreements that would otherwise emerge during implementation.

### References (optional but useful)

- Customer interviews / research findings
- Competitive analysis
- Internal discussions / decision documents that led to this PRD
- Related prior PRDs (especially for era-2+ PRDs)
- Concept-level product mocks, if any — as frozen exports in `docs/prd/assets/`, frozen with the PRD; the live design file stays outside the repo, linked here. (Per-feature mockups belong to the feature's spec instead — see [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md) § "Mockups and visual artifacts".)

---

## Worked example 1 — Original launch (B2B billing platform)

A neutral, fictional B2B billing platform. This is the *original era* PRD, the one that ships with v1. Format: lean PRD.

```markdown
# PRD: Ledger — B2B Billing Reconciliation for Mid-Market Finance

**Author:** Founding team
**Date:** 2025-12-08
**Status:** Accepted (era 1 — original launch)
**Frozen:** 2026-04-15 (v1 shipped)

## Summary

Mid-market finance teams (50–500 employees) spend 8–12 hours per controller per
month manually reconciling bank transactions against invoices. Existing accounting
software handles bookkeeping but not reconciliation. **Ledger** is a SaaS workflow
tool that ingests bank-feed and accounting-system data, surfaces unmatched
transactions, and lets controllers reconcile in a fraction of the time.

## The problem

Finance teams use accounting platforms (QuickBooks, Xero, NetSuite) for the
ledger of record. They use bank feeds (Plaid, MX) to see incoming and outgoing
money. But **reconciliation is the gap**: matching what the bank says happened
against what the books say should have happened.

Today, controllers:
- Export bank feed to CSV
- Export accounting-system transactions to CSV
- VLOOKUP / sort / mentally diff in Excel
- Mark matches; investigate mismatches manually
- Re-import results into the accounting system

This takes 8–12 hours/month per controller (observed across 14 interview
participants from mid-market companies, Q4 2025). The pain scales linearly with
transaction volume.

## Target users

- **Primary:** Controllers at companies with 50–500 employees, transacting 500–5,000
  bank movements per month, using one of QuickBooks Online / Xero / NetSuite
- **Secondary:** Accounting managers who supervise the reconciliation process
- **NOT a target:** SMBs (<50 employees — too few transactions to justify the
  cost), enterprises (>500 employees — they need ERP integration, out of scope)

## Solution overview

Web SaaS app. Users connect their accounting system and their bank feeds via
existing aggregator APIs. Ledger pulls transactions from both sides nightly,
runs a matching algorithm (exact match + heuristics for partial matches), and
presents an "unmatched queue" the controller works through in the UI. Confirmed
matches sync back to the accounting system.

## Success metrics (12 months post-launch)

- **Primary:** 50 paying companies on a $200–500/month tier (≈ $200k ARR)
- **Activation:** New customer reconciles their first month within 14 days of signup
- **Retention:** < 5% monthly churn after month 3

## Constraints

- **Technical:** Must integrate with QuickBooks Online, Xero, NetSuite (in that
  order of priority) and Plaid + MX. No on-prem version in era 1.
- **Legal:** SOC 2 Type 1 within 6 months of GA. GDPR compliance from day 1.
- **Business:** $1.2M seed budget; team of 5 engineers + 1 PM + 1 designer; ship
  v1 within Q1 2026.

## Out of scope (deliberately, era 1)

- Multi-currency support (US-only in era 1)
- Custom matching rules / per-customer ML (heuristics are enough for v1)
- Mobile app (web-only)
- ERP integrations (SAP, Oracle) — enterprise segment is era 2+
- Audit-trail export for tax purposes (post-v1 enhancement)
- White-label / OEM offering
- Direct integration with bank APIs (we rely on aggregators for v1)

## Risks and assumptions

- **Assumption:** Mid-market controllers will pay $200–500/month for reconciliation
  alone (separate from their accounting platform). Validated in 14/20 interview
  conversations; not yet validated commercially.
- **Risk:** Plaid / MX coverage of mid-market US banks is uneven; some prospects
  may have banks neither aggregator supports.
- **Risk:** Accounting-system APIs (QB / Xero / NetSuite) are notoriously
  inconsistent. We may need 60% of engineering effort on data plumbing, not UI.

## References

- Customer interviews 2025-Q3 (20 participants, summary in `docs/research/2025-Q3-finance-interviews.md`)
- Competitive analysis: Vic.ai, Centime, Botkeeper (`docs/research/2025-Q3-competitive.md`)
- Founder origin story / problem articulation: `docs/research/2025-08-founder-memo.md`
```

That's ~80 lines. Notice what's *not* in it:
- No mention of programming language, framework, database
- No mention of file structure, API design, class names
- No test strategy
- No "the agent should..."

All of those land in `SPEC.md` / `PLAN.md` / `ARCHITECTURE.md` / `ADR-*` once engineering starts era 1 implementation.

---

## Worked example 2 — Era 2 (multi-currency expansion)

Same platform, two years later. The team has hit product-market fit in the US ($350k ARR, 75 customers); now they're expanding to European mid-market. This requires a new PRD because the user segment, regulatory constraints, and core matching logic all change.

```markdown
# PRD: Ledger Era 2 — Multi-Currency for European Mid-Market

**Author:** Product team (Sarah K., Mark T.)
**Date:** 2027-Q2 kickoff
**Status:** Accepted (era 2 — European expansion)
**Frozen:** 2027-12-05 (era 2 shipped)
**Builds on:** `docs/prd/2025-12-original-prd.md`

## Why this is a new PRD (not a feature in era 1)

European mid-market reconciliation differs from US in ways that touch every
layer of the product:

- **Multi-currency** in every transaction (US-only assumed single currency)
- **VAT reconciliation** as a first-class workflow (US has no VAT)
- **SEPA / IBAN payments** instead of ACH (different bank-feed semantics)
- **Pan-European customers** want consolidated reporting across countries
- **GDPR data residency** — some customers require EU-only data storage

These aren't "add a currency dropdown" features. They reshape the matching
engine, the schema, the bank-feed integration layer, and the reporting model.
A separate PRD makes the era boundary explicit.

## Summary

Extend Ledger to support EU mid-market finance teams, with first-class
multi-currency, VAT reconciliation, SEPA/IBAN bank-feed support, and EU data
residency. Target launch: Q4 2027 in the UK, DE, FR.

## The problem

EU mid-market finance teams have the same reconciliation pain as US controllers
(see era 1 PRD), but compounded by:
- Daily FX movements across customer / supplier accounts
- Quarterly VAT filing that requires matching transactions to VAT-classified
  invoices
- Pan-European operations: customers running ops across UK / DE / FR / NL want
  one view, in their reporting currency

Existing US reconciliation tools (us included) treat currency as an afterthought.
Local incumbents (e.g., per-country accounting tools) don't cross borders. A
unified EU offering is the gap.

## Target users

- **Primary:** Controllers at companies with 50–500 employees operating in 2+
  EU countries, transacting in 2+ currencies (typically EUR + GBP, or EUR + USD
  via a US subsidiary)
- **Secondary:** Group financial controllers consolidating across subsidiaries
- **NOT a target:** Single-country, single-currency UK SMBs (existing UK
  accounting tools cover this segment well); enterprise (still out of scope)

## Solution overview

Add multi-currency to the matching engine (with daily FX rate ingestion and
configurable rounding), build a SEPA/IBAN aggregator layer, add VAT
classification at the transaction level, build a consolidated reporting view
across currencies, and ship an EU data-residency tier (workloads pinned to
Frankfurt or Dublin region).

## Success metrics (12 months post-EU launch)

- **Primary:** 30 paying EU companies on a €250–600/month tier (≈ €120k EU ARR)
- **Activation:** EU customers complete their first cross-currency reconciliation
  within 21 days of signup (allowing for slower commercial onboarding)
- **Retention:** < 6% monthly churn (slightly higher than US benchmark; EU sales
  cycles run longer)
- **No regression:** US churn unchanged from era-1 baseline of 4%

## Constraints

- **Technical:** EU data residency for EU customers — workloads pinned to
  AWS eu-central-1 (DE) or eu-west-1 (IE). Cross-region traffic minimized.
- **Legal:** GDPR strict; explicit data-processing agreements with all bank-feed
  partners. UK-specific (post-Brexit) data handling for UK customers.
- **Business:** €2M EU expansion budget; hire local PM in DE and CSM in UK.
  Maintain US team velocity — EU expansion can't slow US roadmap.
- **Continuity:** Existing US customers experience zero disruption (no breaking
  schema migrations, no downtime, no UI changes that confuse them).

## Out of scope (deliberately, era 2)

- Asia-Pacific expansion (era 3, if ever)
- Cryptocurrency reconciliation (interesting but no demand signal yet)
- Enterprise segment (still era 3+)
- Localization beyond English (we'll ship English-only UI in EU for era 2; FR/DE
  localization is era 2.5 if customer demand warrants)
- Mobile app (still web-only)

## Risks and assumptions

- **Assumption:** Multi-currency support is the actual blocker for EU customers,
  not localization or sales presence. (Validated in 11/15 EU prospect conversations
  Q1 2027; not yet validated commercially.)
- **Risk:** SEPA aggregator landscape is more fragmented than Plaid/MX in the
  US. May need to support 3+ aggregators (Plaid is launching EU coverage but
  isn't comprehensive; Truelayer / Tink / Yapily are alternatives).
- **Risk:** VAT reconciliation might be too country-specific to handle generically;
  we may need per-country VAT rule packs (DE, UK, FR distinct).

## References

- Era 1 PRD: `docs/prd/2025-12-original-prd.md`
- EU prospect interviews 2027-Q1 (`docs/research/2027-Q1-eu-prospects.md`)
- SEPA aggregator landscape analysis (`docs/research/2027-Q1-sepa-aggregators.md`)
- Postmortem from failed German pilot, Q4 2026 (`docs/postmortems/2026-Q4-de-pilot.md`)
```

That's ~75 lines. Compared to era 1:

- **Explicit "Why this is a new PRD"** — names the era boundary
- **Builds on era 1** — references the prior PRD, doesn't claim to replace it
- **More precise out-of-scope** — leverages what the team learned in era 1
- **Different risk profile** — era 2 risks are about new geo / new regulations, not new product

Both PRDs freeze after their era ships. The era-1 PRD is read in 2027 by someone wanting to know *why the US product exists in its current shape*; the era-2 PRD is read in 2030 by someone wanting to know *what we believed about EU mid-market in 2027*.

---

## Era boundary heuristics — when do you need a new PRD?

The main SDD guide lists *when to write a new PRD* (new market, major capability, pivot, post-reorg). This section goes deeper: how do you *recognize* an era boundary in practice?

### Five concrete signals

**1. The current PRD's "target user" no longer describes who's buying.**

If your era-1 PRD says *"mid-market US controllers"* and you're now selling to *"EU group CFOs running pan-European ops"* — that's not a feature expansion, that's a new audience. New audience → new PRD.

**2. The current PRD's "out of scope" list is being violated.**

Out-of-scope items get pulled into scope all the time. *Two* of them at once, in a coordinated push, often means an era boundary. If your era-1 PRD said *"out of scope: multi-currency, EU expansion"* and you're now doing both — new PRD.

**3. Adjacent functions need to retool, not just engineering.**

A new feature affects engineering. A new era affects engineering, sales, support, marketing, finance, legal. If sales is changing its motion, marketing is repositioning, and support is hiring different roles — that's an era boundary signaled by org-wide change.

**4. The competitive frame changes.**

Era 1 competed against Excel + manual reconciliation. Era 2 competes against per-country accounting tools and pan-European ERP modules. *Different competitive set = different era.*

**5. You'd need to re-pitch the product to existing customers.**

If you'd give your existing customers a substantively different elevator pitch *because* of the change, it's a new era. Era 1: *"reconcile faster."* Era 2: *"reconcile across borders, with VAT, in your reporting currency."* Different pitch → new PRD.

### Signals it's *not* an era boundary (still a feature, write a spec)

- Adding a third bank-feed aggregator (era-1 PRD said "Plaid + MX"; adding Yodlee is a spec, not a new era)
- Improving the matching algorithm (still serving the same workflow)
- Adding a settings page (doesn't change anything fundamental)
- Performance work (doesn't change the product proposition)
- Bug fixes (obviously)

### When in doubt

Ask: *"would a new hire on the product side need this written down to understand what we're building?"* If yes — era boundary. If they'd understand from the existing PRD plus a handful of specs — feature work.

Don't fall into the trap of writing a new PRD every quarter to feel productive. Era boundaries are rare. Three to five PRDs over a decade is healthy; thirty is bloat.

---

## AI-assisted PRD authoring

The agent can speed up PRD drafting and review meaningfully, *as long as you remember the agent can't make the product decisions for you*. Five reusable prompts — and prompt 1 is also packaged as the **`/prd-new`** command (same idea→draft, but interactive: it sketches a lean PRD, then asks you the open questions and fills them in). The shipped command files are namespaced and phase-numbered (`templates/.claude/commands/sdd-1-prd-new.md` installs as `/sdd-1-prd-new`); this guide writes the short forms — keep or drop the prefix in your repo.

### 1. Draft a PRD from a one-paragraph idea

Use when you have a rough sense of what to build but need help structuring.

**Prompt:**

```text
We want to build the following: [one-paragraph description].

Draft a lean PRD using this structure:
- Summary (3-5 sentences)
- The problem (1-2 paragraphs, with specific user pain)
- Target users (primary, secondary, explicitly NOT a target)
- Solution overview (product-level, no implementation detail)
- Success metrics (1-3 measurable outcomes with rough numbers and timeframes — flag [VERIFY] if guessing)
- Constraints (technical, legal, business)
- Out of scope (be explicit; list at least 5 things)
- Risks and assumptions (3-5)
- References (placeholders if you don't know)

DO NOT include: programming language, framework, database, API design, class names,
test strategy. Those go in spec/plan/ADRs after the PRD is accepted.

Mark anything you're inventing as [VERIFY]. Show me the draft before saving.
```

### 2. Review a draft PRD for completeness and clarity

Use after a first draft (yours or the agent's) to catch gaps. Ships ready-made as `/prd-review` (`templates/.claude/commands/sdd-1-prd-review.md`).

**Prompt:**

```text
Read the draft PRD at docs/prd/[filename].md.

Audit it against this checklist:
1. Is the "Target users" section specific (job title + company size or equivalent)
   or generic ("users")?
2. Does the problem statement describe a specific behavior or cost today, not
   just "users want X"?
3. Are success metrics measurable (numbers and timeframe), or vague?
4. Is "Out of scope" at least 5 items long? Are any of them surprising / things a
   reader might assume are in scope?
5. Does the solution section avoid implementation detail (languages, frameworks,
   class names)? Flag any leakage.
6. Are risks named with consequences ("if X, then Y"), or are they generic?
7. Are there any contradictions between sections (e.g., out-of-scope item
   appears as a constraint)?
8. Is it still a page or two? A PRD that needs a table of contents is doing a
   spec's job.
9. Is it factual and skimmable — claims, numbers, decisions, not marketing prose
   or filler? A short PRD can still be padded and vague.

Return a numbered list of concerns. Don't modify the file — list the gaps.
```

### 3. Detect an era boundary from a roadmap

Use periodically (annually or per planning cycle) to check whether you should be writing a new PRD vs. a series of specs.

**Prompt:**

```text
Read the most recent PRD at docs/prd/[latest].md.
Then read ROADMAP.md (or the equivalent) and the last 10 shipped specs in specs/.

Apply these signals — return one paragraph of analysis per signal:
1. Is the target user described in the PRD still the target user the roadmap implies?
2. Are any "Out of scope" items from the PRD being pulled into recent or planned work?
3. Are sales / marketing / support indications (if mentioned anywhere) suggesting
   reorientation, or steady state?
4. Has the competitive frame mentioned in the PRD shifted?

Conclude: "Likely era boundary" / "Feature accumulation, not era boundary" /
"Inconclusive — needs human discussion." Cite specific evidence.
```

### 4. Generate a competitive-analysis section

For PR-FAQ or full PRD formats. Useful for confirming you've thought about
adjacent solutions.

**Prompt:**

```text
For the product idea: [one paragraph description].

Generate a competitive analysis section covering:
- 3-5 named competitors or adjacent solutions (use real names you know exist —
  do NOT invent companies)
- For each: what they do, what they don't do, where we differ
- A summary paragraph: what's our specific wedge or differentiation

If you don't know the space well enough to name real competitors, say so —
don't fabricate. Mark uncertainty as [VERIFY].
```

The "don't fabricate" instruction is important — agents will invent plausible-sounding company names. Always verify the named competitors actually exist before publishing.

### 5. Translate an accepted PRD into era's first set of specs

Use when a PRD is accepted and engineering is kicking off the era. This prompt does one narrow thing: name the first 2–3 specs to start with. For the real slicing pass — vertical slices, the walking skeleton, prioritization — use the prompt in [§ "Slicing the PRD into features"](#slicing-the-prd-into-features).

**Prompt:**

```text
Read the accepted PRD at docs/prd/[filename].md.

Propose the first 2-3 specs needed to start implementing this era.
For each:
- Proposed filename (specs/YYYY-MM-feature-slug/)
- One-sentence scope
- The PRD section(s) it derives from
- A note on dependencies (which spec must ship before which)

This is NOT writing the specs — just identifying what specs we'd need. The
engineering team will then write each spec individually.

Don't include implementation detail. Specs at this stage are placeholders with
scope only.
```

This is the bridge from PRD-as-strategy to specs-as-implementation. The output is the starting set; the full feature backlog comes from the slicing pass.

---

## Success metrics — defining "what does success look like"

The single section most teams get wrong. Two failure modes:

- **Too vague** — *"users love it"*, *"adoption"*, *"engagement"*. Unmeasurable. Six months later nobody can say whether you succeeded.
- **Too many** — 12 KPIs that all matter. None actually drive decisions. Optimization happens nowhere.

### The 1–3 rule

Pick **1 primary metric** that captures the bet (*"50 paying customers on the $200+/month tier within 12 months"*) and at most 2 supporting ones (activation, retention, leading indicators).

If you can't name your *primary* metric, the PRD isn't ready. If you have 5+ primary metrics, you have no primary metric.

### Metric shapes that work

- **Revenue/users at price point Y** — clearest for B2B SaaS
- **Active usage at frequency F** — clearest for consumer / engagement-heavy products
- **Cost reduction of X% in process P** — clearest for internal tools
- **Adoption rate among segment S within T months** — clearest for cross-sell or expansion
- **Quality outcome (error rate ↓ X%, accuracy ↑ Y%)** — clearest for reliability or trust-driven products

### Metric shapes that don't work

- *"User satisfaction"* unless tied to a specific instrument and target (NPS ≥ X, CSAT ≥ Y)
- *"Engagement"* without a definition (sessions? actions? time?)
- *"Adoption"* without a baseline and target rate
- *"Best-in-class"* — meaningless
- Anything you can't put a number on

### Include the timeframe

*"X by Y date"*. Without a timeframe, success has no expiry; teams optimize indefinitely without knowing if they've reached the bar.

### Leading indicators are not success metrics

*"Customers click the new button"* is a leading indicator of engagement. It's not success. Don't promote leading indicators to primary metrics — measure them, but don't confuse activity for outcome.

---

## Stakeholder review process

A PRD that's been written but not reviewed isn't yet a PRD — it's a draft. Two patterns work, pick one.

### Pattern A: Async PR-style review

The PRD is committed as a `Status: Draft` PR. Reviewers comment inline. Once consensus emerges, a second small PR flips to `Status: Accepted` and adds the freeze date.

**Reviewers (typical):**

- **Founding team / executive sponsor** — does this match strategic direction?
- **Engineering lead** — is this feasible given the constraints and team?
- **Product manager** (if not the author) — does this make commercial sense?
- **Design lead** — does the user framing make sense from a UX standpoint?
- **Finance** — are the success metrics realistic? Budget assumptions hold?
- **Legal/compliance** — for regulated industries, any showstoppers in constraints?
- **Sales / customer success lead** — does this match what customers are asking for?

Not every reviewer is mandatory — match to the PRD's scope. A small internal tool might only need engineering + the requesting team's lead.

**Pros:** standard tooling, async-friendly, comment trail.
**Cons:** can drag on without an owner forcing closure.

### Pattern B: Review meeting (Amazon-style)

For PR-FAQs especially. The PRD is distributed in advance; the meeting begins with 20 minutes of silent reading; then group discussion. Decision made (and noted) by end of meeting.

**Pros:** forcing function for closure; everyone hears the same context.
**Cons:** remote-unfriendly; requires senior alignment of calendars.

### Hybrid

Many teams: async comment phase to surface issues, then one short alignment meeting to ratify. Most efficient for medium-sized teams.

### What reviewers check (universal)

A working PRD review checklist:

- [ ] **Problem** is specific (named user, named pain, named cost)
- [ ] **Target users** are precisely scoped (and a non-target segment is named)
- [ ] **Solution** stays at the product level (no implementation creep)
- [ ] **Success metrics** are 1–3, measurable, with timeframes
- [ ] **Out of scope** is at least 5 items long and includes some "surprising" exclusions
- [ ] **Risks** are named with consequences, not generic
- [ ] **Constraints** don't contradict the solution
- [ ] **References** point to sources (research, prior PRDs, decision docs) — not fabricated

### How long should review take?

For a lean PRD: 24–72 hours, with no more than two review rounds.
For a PR-FAQ: 1–2 weeks, possibly with multiple feedback cycles.
For a full PRD: 2–6 weeks (the format implies the slow review).

PRDs that sit in `Draft` for months are dead PRDs. Either ratify or kill.

---

## Cross-functional handoff — PRD to engineering

The PRD is accepted. Now what? The handoff from PRD-as-strategy to specs-as-implementation is a high-leverage moment.

### The kickoff meeting

A 60-minute meeting with PM, eng lead, design (if applicable), and 1–2 senior engineers who will own the work. Agenda:

1. **PM walks the PRD** (15 min) — focus on Problem, Target user, Success metrics, Out of scope. Skip what's obvious.
2. **Engineering reads through Constraints + Risks** (10 min, often silent) — surface technical concerns.
3. **Open discussion** (25 min) — what's unclear? Where are the biggest risks? What's the right shape for the first 3–5 specs?
4. **Output: spec backlog** (10 min) — list of specs to create, with rough order and ownership.

The output of the kickoff is *not* the specs themselves. It's the *list* of specs to write next.

### Slicing the PRD into features

The kickoff's output — the spec backlog — is a list of **features**, each sized to its own spec → plan → tasks trio. Getting that list right is mostly one skill: **slicing**. (Solo? You do this on your own; the meeting is optional, the slicing isn't.)

**Slice vertically, not horizontally.** The most common mistake is slicing by layer — "the database work," "the API work," "the UI work." None of those ship anything a user can use, and none can be verified end-to-end. Slice by **capability** instead: a thin path that runs through every layer and delivers one visible outcome. (Standard terms: *vertical slice*, *walking skeleton*.)

*Bad vs good, concretely.* For an order-export platform, a **horizontal** (bad) first slice is *"build the database schema for every export type"* — it ships nothing a user can see and can't be verified end-to-end. A **vertical** (good) first slice is *"one partner's orders → XML → SFTP, hardcoded trigger"* — thin and ugly, but it runs through every layer and proves the core path. The bad slice front-loads infrastructure nobody can use yet; the good one front-loads a working skeleton you thicken later.

**Find the walking skeleton first.** The first feature should be the thinnest end-to-end slice that proves the PRD's core value — even if it hardcodes the boring parts. Ship it, then thicken. This beats building the whole foundation before anything works.

**Derive features from the PRD's success criteria.** Each measurable outcome in the PRD maps to one or a few features. If a success criterion has no feature, you missed one; if a feature serves no criterion, question it — it may be out of scope.

**Prioritize and sequence.** Tag each feature P1/P2/P3 (or MoSCoW). Order by *value × dependency*: build only the foundation the next valuable slice actually needs — not all of it up front. Mark hard dependencies so nothing is scheduled before its prerequisite.

**Re-slice as you learn — the list is a first guess, not a contract.** A shipped slice routinely changes what the next one should be: a partner edge case you didn't anticipate, a step that turned out trivial, a whole slice that became unnecessary. After a slice ships, let what you learned re-rank, add, or drop slices before you start the next trio — in `FEATURES.md` if you keep one, otherwise just in your tracker or your head. This is the upstream half of the loop the trio runs downstream: the PRD freezes, the slice list does not. (Guardrail: re-slice when a *ship* taught you something — not endlessly; if you're re-slicing without having shipped anything, see the anti-patterns below.)

**Right-size to a trio.** Each feature should fit one trio: a `SPEC.md` under ~150 lines, a few days of work. Too big (multi-week, sprawling spec) → split it. Too small (a one-liner) → fold it into a neighbor or treat it as a bugfix-shape spec.

**The output is a lightweight list, not an artifact.** A prioritized table — feature, the PRD outcome it serves, rough size, dependencies — is enough. It can live in the PRD or an issue tracker. Each row becomes a `specs/YYYY-MM-slug/` when you pick it up. Don't turn the backlog into a second heavy document.

**Let the agent draft it, then you triage.** `/features-from-prd` (or the prompt below) turns the PRD into a candidate breakdown in seconds; you cut and reorder.

**Prompt:**

```text
Read docs/prd/<name>.md. Propose a feature breakdown as vertical slices —
each independently shippable and user-visible (NOT "DB layer" / "API layer").
Size each to ~a few days (a spec under ~150 lines); split anything bigger.
Flag the walking skeleton (thinnest end-to-end slice that proves the core),
then order the rest by value × dependency and mark dependencies. No feature
may violate the PRD's Out of scope. Output a table:
feature | PRD outcome it serves | rough size | depends on | P1/P2/P3.
Don't write specs — just the breakdown.
```

**Anti-patterns:** horizontal slices (layers that ship nothing); one giant spec for the whole PRD (big-bang); slicing by team or component instead of user value; building all the infrastructure before any user-visible slice; decomposition that never ends (if you're on the third re-slice, pick the obvious P1 and start).

Each feature you pick up becomes a spec — for the mechanics, see [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md).

### The first specs

From the backlog identified in the kickoff, the engineering team writes the first 2–3 specs in parallel. These specs:

- Reference the PRD by path in their `## References` section
- Don't violate the PRD's `Out of scope`
- Translate constraints into concrete acceptance criteria
- Don't try to cover the entire PRD in one spec — pick the foundational pieces first

For full mechanics of spec authoring, see [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md).

### PM's role post-PRD

After the PRD is accepted:

- PM **doesn't** write specs (engineers do)
- PM **does** review specs for scope alignment with PRD
- PM **does** sign off on `STATUS: shipped` for each spec
- PM **does** track success metrics as features ship

The PRD is the contract. Specs are the implementation of that contract. PM keeps the contract honest.

---

## PRD-specific anti-patterns

Beyond the Frankenstein-PRD anti-pattern already covered in the main SDD guide (*editing an old PRD instead of writing a new one*), several traps are PRD-specific.

### 1. Marketing copy disguised as PRD

The Summary section reads like a landing-page hero: *"Revolutionizing the way modern teams reconcile finances with AI-powered intelligence."* No specific user, no specific pain, no measurable outcome.

**Fix:** if you couldn't show this to a customer and ask *"is this you?"*, you're writing marketing, not a PRD. Rewrite with specific job titles, specific behaviors, specific cost.

### 2. "Everything is in scope"

The PRD lists 20 features. Out-of-scope section is missing or has 1 item. Six months in, engineering ships 6 features and the others slip — but the PRD still claims to cover all 20.

**Fix:** out-of-scope must be at least 5 items, ideally including some that *look* in scope but aren't. Force the conversation about what you're *not* doing in this era.

### 3. Missing "for whom"

The product solves a problem in the abstract but doesn't name the user. *"Finance teams"* is too generic.

**Fix:** name a primary user with job title + company size + behavior. Name a secondary user. Name a non-target ("NOT for X").

### 4. Implementation detail leaking in

*"Built on React + Postgres, with a microservices architecture, using OAuth 2.0…"* These are engineering decisions, not product decisions. They lock options before specs are written.

**Fix:** if a sentence could go in `PLAN.md`, it doesn't belong in the PRD. Move it.

### 5. PRD that reads like a spec

The PRD has acceptance criteria, file paths, API shapes, test strategies. Engineers will treat it as the implementation guide, which it isn't.

**Fix:** PRD is *what + why* at the product level. If you find yourself writing *"the endpoint shall…"*, that's a spec.

### 6. Vague success metrics

*"Users will love it."* Six months later, did they? Nobody can say.

**Fix:** the 1–3 measurable-outcomes-with-timeframes rule. See [§ "Success metrics"](#success-metrics--defining-what-does-success-look-like).

### 7. Era expansion masquerading as feature work

The product is quietly drifting into a new market / use case / segment without a new PRD. The era-1 PRD still claims to describe the product, but the new direction is happening anyway.

**Fix:** apply the era-boundary heuristics. If two signals fire, write a new PRD before continuing.

### 8. Fabricated competitive analysis

Especially common with AI-assisted drafting: the PRD lists competitors that don't exist or misrepresents what real competitors do.

**Fix:** verify every named company. Mark any uncertain ones [VERIFY] and resolve before accepting.

### 9. No risks named

The Risks section is empty or generic ("technical risk", "market risk"). When something blows up six months in, nobody can claim it was foreseen.

**Fix:** name at least 3 specific risks with named consequences. *"If aggregator coverage is uneven, ~30% of prospects won't be servable on launch."*

### 10. PRD freezes never happen

The PRD says *"Status: Draft"* for 18 months. Nobody ratifies it. Engineering builds anyway, against an unaccepted document.

**Fix:** enforce review SLA. A draft PRD older than 4–8 weeks (depending on format) gets escalated to a decision: ratify, revise, or kill.

---

## Golden rules

1. **PRD is for humans.** The agent implements from `SPEC.md → PLAN.md → TASKS.md` — it only reads the PRD to draft, review, or slice it. Don't write PRD with implementation detail "for the agent" — that detail belongs in `PLAN.md`, an ADR, or `CLAUDE.md`.

2. **Specificity > scope.** A PRD that names one specific user with one specific pain outperforms a PRD that gestures at "all our users."

3. **The 1–3 success metrics rule.** Anything more is performative; anything less is unaccountable.

4. **Out of scope is the most underrated section.** It does more to prevent drift than any positive guidance.

5. **One PRD per era.** Not per feature, not per quarter. Era boundaries are recognizable events, not calendar entries.

6. **Each PRD freezes after its era ships.** Body never edited again. New direction → new PRD.

7. **Pick the format your audience will read.** A lean PRD nobody reads beats nothing; a full PRD that gets skimmed beats one that gets ignored.

8. **The agent drafts; humans decide.** PRD content is product strategy. The agent can speed up drafting and review; it can't make the call.

9. **Verify every named competitor and statistic.** Agent-drafted PRDs frequently include fabricated specifics. Audit before accepting.

10. **Drafts older than a month are rot.** Ratify, revise, or kill. A PRD in perpetual draft state is worse than no PRD.

---

*This guide complements [`spec-driven-development-guide.md` § "The PRD Layer"](spec-driven-development-guide.md#the-prd-layer) (principles), [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md) (writing the spec → plan → tasks trio for each feature you slice out), and [`sdd-in-teams-guide.md`](sdd-in-teams-guide.md) § "Who owns what" (who owns the PRD on a small team). The PRD is where strategy enters the SDD discipline; everything downstream is its consequence.*
