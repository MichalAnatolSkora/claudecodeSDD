# Research in the Repo: What Goes In, What Stays Out, How the Agent Uses It

> The practical companion to [`spec-driven-development-guide.md` § "Before the PRD: Research and Discovery"](spec-driven-development-guide.md#before-the-prd-research-and-discovery). That section gives you the principle (research feeds PRD; humans + agent context, never code source). This guide gives you the practice: folder layout, artifact types, the synthesis discipline, PII handling, AI-assisted synthesis prompts, and the anti-patterns specific to research material.

---

## Table of Contents

1. [What this guide adds](#what-this-guide-adds)
2. [The principle, restated](#the-principle-restated)
3. [PII and anonymization — the gating concern](#pii-and-anonymization--the-gating-concern)
4. [Folder structure for `docs/research/`](#folder-structure-for-docsresearch)
5. [Research artifact types](#research-artifact-types)
6. [The synthesis discipline — raw → patterns → themes](#the-synthesis-discipline--raw--patterns--themes)
7. [AI-assisted synthesis](#ai-assisted-synthesis)
8. [The PRD ↔ research interface](#the-prd--research-interface)
9. [Research-specific anti-patterns](#research-specific-anti-patterns)
10. [Golden rules](#golden-rules)

---

## What this guide adds

The main SDD guide (§ "Before the PRD: Research and Discovery") tells you:

- *Why* research lives in the repo (agent context, PRD source)
- *What* the agent does with it (3 legitimate use cases; never code generation)
- *What* goes in vs stays out (synthesized artifacts in; raw PII out)
- *When* anonymization happens (before commit)

This guide adds:

- **The complete folder layout** for `docs/research/` with subfolders by artifact type
- **PII handling** as a prominent section with concrete examples and a pre-commit-hook recipe
- **Artifact-type breakdown** with structure templates for each (interview synthesis, competitive, sizing, validation)
- **The synthesis discipline** — how to turn raw notes into useful artifacts without losing rigor
- **AI-assisted synthesis prompts** for the most common research tasks the agent can help with
- **The PRD ↔ research interface** — how research feeds PRDs, how PRDs cite research
- **Research-specific anti-patterns** — the failure modes you won't see in other guides

This guide deliberately does **not** cover *"how to conduct user research"* — that's a discipline of its own with established literature (Nielsen Norman Group, Erika Hall's *Just Enough Research*, etc.). Scope here: how research artifacts live in an SDD repo.

---

## The principle, restated

For clarity, the principle from the main SDD, slightly expanded:

**Research artifacts are for humans + agent context. The agent never generates code from research.**

Three legitimate uses of research by the agent:

| Use | Example |
|-----|---------|
| **As context when drafting PRDs** | *"Read `docs/research/2025-Q3-interviews.md`, then draft a PRD for the order-export feature. Cite specific findings."* |
| **As context when implementing user-facing work** | *"Per the user-research synthesis in `docs/research/`, our users call this 'reconciliation' not 'matching'. Use that vocabulary in UI copy."* |
| **As a synthesis assistant** | *"Read the 14 anonymized interview notes in `docs/research/2025-Q3-mid-market-finance/notes/`, extract 3-5 themes, and quote evidence for each."* |

What the agent **doesn't** do:

- Generate features directly from a research finding
- Treat a single quote as a requirement
- Skip the PRD step and go from research to spec

Research is the most upstream artifact in the SDD pipeline. Everything downstream — PRD, spec, plan, tasks, code — passes through human-gated synthesis along the way.

---

## PII and anonymization — the gating concern

This is the single highest-risk part of putting research in a repo. Get it wrong and you've leaked private data into version control history, which is functionally permanent.

For the mechanical side — pre-commit hooks that scan for PII patterns, CI checks on `docs/research/` paths, and the broader pattern of quality gates (mechanical vs LLM evaluator vs human) — see [`quality-gates-guide.md`](quality-gates-guide.md). This section covers the principle and discipline; that guide covers the gates that mechanize it.

**Read this section before anything else.**

### What absolutely never enters the repo

- **Customer names** (individual or company), unless explicitly given written permission *for this specific use*
- **Personal contact info** (email, phone, address, social handles)
- **Direct identifiable quotes** (anything that could be traced to a specific person)
- **Contractual or pricing terms** of specific deals
- **NDA-protected material** of any kind
- **Salary, revenue, or financial figures** identifiable to a specific company or individual
- **Recordings, transcripts, or screen captures** of customer sessions (raw form)
- **Internal politics or stakeholder names** unrelated to the research substance

### What can enter the repo, with anonymization

- **Themed synthesis** — *"Mid-market controllers describe reconciliation as 'the worst part of month-end close,' often with frustration."* No name, no company.
- **Role-attributed quotes** — *"'I'd happily pay for this if it worked' — Controller, mid-market SaaS company"*. Role + segment, not identity.
- **Aggregated statistics** — *"12 of 14 interviewees mentioned Excel as their current tool."* Counts and proportions, not identifiable individuals.
- **Competitive observations** of *public* facts about *public* products

### The pre-commit discipline

Anonymization happens **before** files enter Git. Once committed, the data is in history forever — *even if you delete the file*, the content stays in the Git log unless you do a full history rewrite (which is disruptive).

Two practical patterns:

**Pattern A: separate raw + synthesized.** Raw sources stay in your research-ops tool (Dovetail, Grain, Notion, etc.). Only the synthesized artifact — created by you, with anonymization applied as part of writing — enters the repo. This is the default and recommended pattern.

**Pattern B: anonymize-then-commit.** If you absolutely need raw notes in the repo (e.g., regulated industry with auditable trail), apply anonymization in a separate pass, review the output, then commit. Use a PII linter (`detect-secrets`, custom regex hook) as a safety net.

**Pre-commit hook recipe** (for Pattern B or as a safety net for Pattern A):

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "case \"$CLAUDE_FILE_PATHS\" in docs/research/*) if grep -qE '\\b[A-Z][a-z]+ [A-Z][a-z]+\\b|@[a-zA-Z0-9.-]+\\.|[0-9]{3}-[0-9]{3}-[0-9]{4}' \"$CLAUDE_FILE_PATHS\"; then echo 'WARNING: possible PII in research file. Names, emails, or phone numbers detected. Verify anonymization before committing.' >&2; fi ;; esac"
          }
        ]
      }
    ]
  }
}
```

(This is a heuristic — `Firstname Lastname` patterns, email addresses, phone formats. It will have false positives, but a warning is cheap insurance.)

### "When in doubt, leave it out"

The rule when you're not sure whether something is PII: it doesn't go in. The cost of over-anonymizing is small (slightly less specific synthesis). The cost of under-anonymizing is enormous (legal exposure, reputational damage, breach of customer trust).

---

## Folder structure for `docs/research/`

Recommended layout for a repo with active research material:

```
docs/research/
├── README.md                                  # index of what lives here, by date
├── interviews/
│   ├── 2025-Q3-mid-market-finance/
│   │   ├── synthesis.md                       # themes, patterns, anonymized quotes
│   │   └── methodology.md                     # who (by role), how many, when, how (no raw transcripts)
│   └── 2027-Q1-eu-prospects/
│       ├── synthesis.md
│       └── methodology.md
├── competitive/
│   ├── 2025-Q3-reconciliation-landscape.md    # named competitors, sourced
│   └── 2027-Q1-sepa-aggregators.md
├── sizing/
│   ├── 2025-mid-market-tam.md                 # TAM/SAM/SOM with assumptions
│   └── 2027-eu-mid-market-tam.md
├── validation/
│   ├── 2026-Q4-failed-de-pilot.md             # what we learned from the failed pilot
│   └── 2026-Q2-fx-rate-assumption-validation.md
└── opportunity-briefs/
    └── 2027-Q3-vat-reconciliation-brief.md    # PRD candidates not yet ratified
```

### Why subfolders by artifact type

- **Discoverable** — *"where's our competitive analysis?"* has one answer (`docs/research/competitive/`), not *"grep through docs/research/ and see what looks competitive"*
- **Scopes the agent's reads** — *"read `docs/research/interviews/`"* is precise; *"read all research"* loads everything
- **Different lifecycles** — competitive analyses go stale faster than interview synthesis; subfolder boundaries help mark *"interviews don't need quarterly refresh, competitive does"*

### Why dated filenames

`2025-Q3-mid-market-finance/` not `mid-market-finance/`. Research is **time-stamped truth** — what was true in Q3 2025 may not be true now. The date forces the reader (and the agent) to consider currency before relying on a finding.

### `docs/research/README.md` as the index

A short README listing what's in the folder, when each was last updated, and what each was used for (which PRD, which decision). Without this, the folder rots into a graveyard of one-time analyses.

Sample:

```markdown
# Research index

| Artifact | Date | Used for | Currency |
|----------|------|----------|----------|
| `interviews/2025-Q3-mid-market-finance/` | 2025-09 | Original launch PRD | Likely stale (re-validate Q3 2026) |
| `competitive/2025-Q3-reconciliation-landscape.md` | 2025-09 | Original launch positioning | Stale — competitors have shifted |
| `interviews/2027-Q1-eu-prospects/` | 2027-03 | Era 2 EU PRD | Current |
| `sizing/2027-eu-mid-market-tam.md` | 2027-02 | Era 2 EU PRD | Current |
```

The `Currency` column is what keeps stale findings from quietly informing current decisions.

---

## Research artifact types

Five common shapes, each with a recommended structure.

### 1. Interview synthesis

The output of a batch of customer/user interviews, anonymized and themed.

Structure:

```markdown
# Interview synthesis: [topic, audience, date]

**Period:** [start date – end date]
**Interviewer(s):** [names — internal team only, not subjects]
**Participants:** [N interviews with people in role X at company size Y]
**Methodology:** [brief — semi-structured? scripted? duration?]
**Raw sources:** [stays in Dovetail / Grain / Notion; not in this repo]

## Themes

### Theme 1: [one-sentence claim]

**Evidence (frequency: N/M interviews):**
- *"Direct quote, anonymized"* — Controller at mid-market SaaS company
- *"Another direct quote"* — Finance manager at consumer-brands company
- Paraphrased pattern: 9 of 14 participants described X as Y

**Implications:**
- For the product: [...]
- For the PRD: [...]
- Confidence: [high / medium / low — based on sample size and consistency]

### Theme 2: ...
```

A copy-pasteable starter lives at [`templates/research-synthesis.md`](../templates/research-synthesis.md).

**Notes:**
- Quote attribution: *role + segment*, never *name + company*.
- Each theme includes evidence frequency — "12 of 14" beats vague "most."
- Implications are explicit and structured — what does this finding tell PRD-writers?

### 2. Competitive analysis

Named competitors, what they do, what they don't, where you differentiate.

Structure:

```markdown
# Competitive analysis: [market segment, date]

**Period:** [date of analysis]
**Sources:** [public websites, product trials, public pricing, analyst reports — cite each]

## Competitor 1: [Name]

- **What they do well:** [...]
- **What they don't do (or do poorly):** [...]
- **Pricing:** [if public; otherwise "not public"]
- **Where we differentiate:** [...]
- **Sources:** [URLs, dated]

## Competitor 2: ...

## Summary

- Our specific wedge: [one paragraph]
- Adjacent threats / non-competitors: [who could enter? who's "almost" a competitor?]
- Currency: [when this analysis goes stale — usually 6–12 months]
```

**Notes:**
- Use **real, named** competitors. If you can't name them, you haven't done competitive analysis — you've written speculation.
- Source every claim. *"Pricing $X/month"* needs a URL or screenshot reference (kept elsewhere).
- Date everything. Competitive landscape shifts quarterly.

### 3. Market sizing

TAM / SAM / SOM with assumptions stated explicitly.

Structure:

```markdown
# Market sizing: [market, segment, date]

**Methodology:** [top-down? bottom-up? hybrid?]
**Currency:** [when this should be refreshed]

## TAM (Total Addressable Market)
- Claim: $X by 2030
- Source: [analyst report name, URL, page; or own calculation]
- Assumptions: [...]

## SAM (Serviceable Available Market)
- Claim: $Y (subset of TAM that fits our model)
- Filtering criteria: [why we exclude parts of TAM]

## SOM (Serviceable Obtainable Market)
- 3-year obtainable: $Z
- Assumptions about market share growth: [...]
- Comparable trajectories (other products that went from $0 → $Z in 3 years): [examples]

## Sensitivity
- If [assumption] is off by ±30%, what happens to SOM?
- Which assumption is the biggest risk?
```

**Notes:**
- Sizing is mostly assumption-stating, not number-finding. The assumption list is more valuable than the bottom-line figure.
- Source every external claim.

### 4. Problem validation studies

Did the problem we *thought* existed actually exist? Quantitative or mixed-methods.

Structure:

```markdown
# Problem validation: [problem, audience, date]

## Hypothesis (going in)

[The specific problem statement we set out to validate.]

## Method

[Survey / structured interviews / paid pilot / etc.]
[N participants, recruiting method, length of study]

## Results

- [Specific quantitative findings, anonymized]
- [Specific qualitative observations, themed]

## Verdict

- [Validated / partially validated / invalidated]
- [Confidence: high / medium / low]

## Implications for PRD

- [If validated, the PRD can rest on this finding. Cite it.]
- [If invalidated, the PRD needs a different problem statement, or shouldn't be written.]
```

### 5. Opportunity briefs (PRD candidates)

A pre-PRD document arguing *"this might be worth a PRD."* Lives in `docs/research/opportunity-briefs/`.

Structure:

```markdown
# Opportunity brief: [candidate name, date]

**Status:** [Open / Promoted to PRD / Closed (won't pursue)]

## Hypothesis

[One paragraph: what we'd build, for whom, why we think it matters.]

## Research backing this

- [Link to interview synthesis]
- [Link to competitive analysis]
- [Link to sizing]

## Why this might be the next era

[Reference to the era-boundary heuristics — what makes this a new era vs feature work.]

## What we'd need to validate before committing

- [Open questions, unresolved assumptions]

## Decision-by

[Date by which we ratify or close this.]
```

**Notes:**
- Opportunity briefs are *promotions candidates* for PRDs. Most never get promoted.
- Close briefs explicitly (*"Closed 2027-Q1, decision in `docs/decisions-log.md` 2027-01-15"*) rather than letting them rot.

---

## The synthesis discipline — raw → patterns → themes

The hardest skill in research is turning raw input into something useful without losing rigor. Three-stage pattern:

### Stage 1: Raw (stays out of repo)

Interview transcripts, recordings, survey responses, screenshots. Lives in research-ops tool (Dovetail, Grain, Notion).

### Stage 2: Patterns (lives in repo as field notes)

Anonymized, lightly organized observations. Things like:

> *7 of 14 participants mentioned bank-feed reconciliation as their #1 pain. 4 of 14 mentioned VAT specifically. 2 mentioned multi-currency.*

These are not yet themed — just patterns visible in the data.

### Stage 3: Themes (the published synthesis)

Patterns shaped into *claims with evidence*:

> **Theme: Bank-feed reconciliation is the dominant pain in mid-market finance.**
>
> Evidence (frequency: 7/14):
> - *"Reconciliation is the worst part of month-end."* — Controller at mid-market SaaS
> - *"I spend 8 hours a month on this."* — Finance manager at consumer-brands company
> - 7 of 14 named reconciliation as #1 pain unprompted
>
> Implications: this is the right starting wedge for the original launch PRD.

The synthesis is what enters the repo. The transition from raw → themes is **lossy and judgment-laden** — be honest about that.

### The opinion-vs-evidence test

For every claim in a synthesis, ask: *"could a reader trace this back to evidence in the methodology section?"*

- *"Users hate Excel"* — opinion (or maybe just sloppy phrasing)
- *"9 of 14 interviewees described Excel-based reconciliation as 'tedious' or 'error-prone'"* — evidence

Synthesis is a *summary of evidence*, not a *summary of conclusions*.

---

## AI-assisted synthesis

The agent can help with research synthesis — provided you keep raw sources OUT of the repo and only feed it material that's already been anonymized. Four common prompts.

### 1. Extract themes from anonymized interview notes

**Prompt:**

```text
Read every file in docs/research/interviews/[date-slug]/notes/.
These are anonymized notes from N interviews with [audience description].

Identify 3-5 themes. For each:
- One-sentence claim
- Evidence (anonymized quotes with role attribution, count of how many interviews mentioned)
- Confidence (high/medium/low based on consistency and sample size)
- Implications for PRD-writing

Output as a complete synthesis.md following templates/research-synthesis.md structure.

Do NOT introduce names, companies, or any identifying detail not already in the notes.
Do NOT extrapolate beyond what the evidence supports. Mark anything you're unsure
about as [VERIFY].
```

### 2. Cross-reference themes against an existing PRD

**Prompt:**

```text
Read docs/prd/[era].md and docs/research/interviews/[date-slug]/synthesis.md.

For each claim in the PRD's "The problem" section, identify:
- Which research theme(s) support it (cite by theme name)
- Whether any research findings contradict it
- Whether any research themes go unaddressed by the PRD

Return a markdown table: PRD claim, supporting evidence, contradictions, unaddressed.
Do not modify either file.
```

### 3. Identify candidate PRDs from accumulated research

**Prompt:**

```text
Read all files in docs/research/. Cross-reference against the active PRDs in
docs/prd/.

Identify themes or findings that suggest opportunities NOT covered by current
PRDs. For each:
- Brief description
- Supporting research (by file path + theme name)
- Why this might warrant its own PRD (vs being a feature within an existing PRD)
- Counter-argument (why this might NOT warrant a PRD)

Output as a list of opportunity-brief candidates. Don't draft the briefs — just
list the candidates.
```

### 4. Detect stale research

**Prompt:**

```text
Read docs/research/README.md (the index) and the file dates in docs/research/.

For each artifact:
- Is its "Currency" still accurate?
- Has the world changed in a way that would invalidate this finding? (Especially
  for competitive analyses, market sizing — sectors that move fast)
- Is any active PRD or spec still relying on a finding that's likely stale?

Return a markdown table: artifact, last update, suggested action (still current /
re-validate / replace / archive). Don't modify any files.
```

### What the agent will NOT do reliably

- Conduct interviews (obviously)
- Identify novel themes that humans haven't already seen
- Replace a researcher's judgment about which themes matter
- Recognize when interview content is biased by question framing

The agent assists synthesis; it does not *do* research.

---

## The PRD ↔ research interface

Research and PRDs are tightly coupled but distinct artifacts. The interface between them is the most important coupling in the SDD pipeline.

### Research feeds PRD

A PRD's *"Problem"* and *"References"* sections cite research:

```markdown
## The problem

Mid-market finance teams (50–500 employees) currently export bank
transactions to CSV and manually reconcile against invoices in Excel,
losing 8–12 hours per month per controller (per `docs/research/interviews/
2025-Q3-mid-market-finance/synthesis.md`, Theme 1).

## References

- Interview synthesis: `docs/research/interviews/2025-Q3-mid-market-finance/`
- Competitive landscape: `docs/research/competitive/2025-Q3-reconciliation-landscape.md`
- Market sizing: `docs/research/sizing/2025-mid-market-tam.md`
```

The PRD reader can trace claims back to research; the research synthesis is the authoritative source for the user problem.

### PRD changes can prompt new research

When you're writing a new era's PRD (era 2+), you often need fresh research:

- *"We don't actually know if EU mid-market shares the same pain pattern as US."* → schedule EU prospect interviews; add `docs/research/interviews/2027-Q1-eu-prospects/`
- *"Our competitive analysis is two years old."* → refresh competitive section; add `docs/research/competitive/2027-Q1-sepa-aggregators.md`

The new research enters the repo *before* the new PRD references it.

### Research can outlive its PRD

A 2025 research synthesis might inform the 2025 launch PRD, *and* feed into a 2027 era-2 PRD by contributing background context. Research is **time-stamped truth** that ages on its own timeline.

The currency column in `docs/research/README.md` is what keeps this honest — *"used for original launch PRD; re-validate before relying on for era 2."*

### Opportunity briefs are the bridge

A `docs/research/opportunity-briefs/` entry is essentially *"here's research that might justify a future PRD."* When a brief gets promoted, it becomes a PRD in `docs/prd/` and the brief is marked *"Promoted on [date]."* When it doesn't, it's marked *"Closed on [date]"* with reasoning.

This keeps the research → PRD pipeline visible without forcing every interview synthesis to become a PRD.

---

## Research-specific anti-patterns

The failure modes most common to research artifacts, with how to fix each.

### 1. Raw transcripts in repo

*Symptom:* Someone commits `interview-with-jane-doe.txt` containing 45 minutes of transcribed conversation with a customer's full name and email at the top.

*Cure:* enforce the anonymize-before-commit rule. Use a pre-commit hook (see § PII). When in doubt, don't commit.

### 2. Opinion presented as research

*Symptom:* A "research" doc says *"users want X"* with N=1 conversation, or worse, N=0 (just the author's intuition dressed up as user-derived).

*Cure:* require evidence-frequency annotations. Every claim needs *"M of N interviews mentioned this"* or *"per [source]"*. If neither, it's not research — it's an opinion, and belongs in an ADR or in a discussion thread.

### 3. Advocacy disguised as synthesis

*Symptom:* The researcher had a conclusion before starting. Quotes are cherry-picked to support it. Contradictory evidence is omitted or downplayed.

*Cure:* synthesis includes a *"What contradicts this view?"* sub-paragraph for every major theme. If you can't find any, look harder — interview research with zero contradictions usually means biased framing or selective quoting.

### 4. Research that quietly becomes a PRD

*Symptom:* A synthesis doc grows over months. Sections start to look like *"What we should build."* By the end, it's structurally a PRD but living in `docs/research/`. Engineers start citing it as if it were ratified strategy.

*Cure:* if a research artifact contains *"What we should build"* or *"Acceptance criteria,"* it's a PRD candidate. Either promote it (move to `docs/prd/`, run through PRD review process, freeze after era ships) or strip the prescriptive content out and keep it as research.

### 5. Outdated research used as current truth

*Symptom:* A 2023 interview synthesis is cited in a 2027 PRD as if its findings still hold. Markets shift. Users shift. The world from 2023 isn't the world now.

*Cure:* the `Currency` column in `docs/research/README.md`. Annual sweep to mark anything older than 18 months as *"re-validate before relying on."* Stale research, like stale ADRs, is worse than absent research because it confers false confidence.

### 6. Lossy synthesis

*Symptom:* Themes are extracted but the original quotes that supported them are gone. Six months later, you can't reconstruct the evidence chain.

*Cure:* every theme includes 2-3 verbatim (anonymized) quotes as evidence. Treat them as the audit trail.

### 7. Research that nobody reads

*Symptom:* `docs/research/` has 20 artifacts but no recent PRD cites any of them. Researchers feel ignored; PRD writers feel they don't have time to read.

*Cure:* the `docs/research/README.md` index. Make it scannable. *"Used for"* column tells PRD writers *"this is the synthesis you want."* Without an index, research becomes a graveyard.

### 8. Research that bypasses PRD

*Symptom:* Engineer sees a research finding, says *"that sounds important,"* and just builds it. No PRD, no spec, no review.

*Cure:* the agent's guardrails (see [Working with AI Coding Agents § "Stay in scope"](working-with-agents-guide.md)) plus team discipline. Research is *upstream of strategy*. Engineering is *downstream of strategy*. Findings flow through PRDs, never directly into code.

### 9. Per-interview files instead of synthesis

*Symptom:* `docs/research/interviews/` has 47 individual interview files, one per session, each lightly anonymized. The agent and humans both struggle to find patterns.

*Cure:* synthesize in batches. After every 5-10 interviews, produce a synthesis doc. Archive the individual notes (still anonymized) under a `notes/` subfolder if you must keep them at all — but the *synthesis* is the canonical artifact.

### 10. PII linter disabled or absent

*Symptom:* No mechanical check for PII in research files. Discipline relies entirely on every contributor remembering. Eventually someone slips.

*Cure:* the pre-commit hook (see § PII). Even an imperfect linter (false positives included) catches the worst cases.

---

## Golden rules

1. **Anonymize before commit, not after.** Git history makes anonymization-after-the-fact disruptive at best, impossible at worst.

2. **Synthesized artifacts in the repo; raw sources outside it.** Dovetail / Grain / Notion / your research-ops tool keeps the raw; the repo gets the distilled output.

3. **Every claim has evidence frequency.** *"M of N interviews mentioned X"* — not *"users said."*

4. **Quote attribution is *role + segment*, never *name + company.*** *"Controller at mid-market SaaS company"* is the right form.

5. **Research is upstream of PRD, never a substitute for it.** Findings feed PRDs. PRDs feed specs. Engineering does not consume research directly.

6. **Date everything; track currency.** Research is time-stamped truth. The `docs/research/README.md` index tracks what's still current vs needs re-validation.

7. **The agent assists; the human judges.** AI can extract themes from anonymized notes; AI cannot decide which themes matter.

8. **When in doubt about PII, leave it out.** The cost of over-anonymizing is small; the cost of under-anonymizing is enormous.

9. **A research artifact that prescribes "what to build" has become a PRD by accident.** Promote it or strip it.

10. **`docs/research/README.md` is the index that keeps research alive.** Without it, the folder rots into a graveyard.

---

*This guide complements [`spec-driven-development-guide.md`](spec-driven-development-guide.md) § "Before the PRD: Research and Discovery" (the principle) and [`prd-guide.md`](prd-guide.md) (the next downstream artifact). Research is where strategy starts; everything downstream — PRDs, specs, plans, tasks, code — flows through this source through human-gated synthesis along the way.*
