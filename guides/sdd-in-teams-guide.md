# SDD in Teams: Roles, Ownership, and Handoffs

> The full how-to for running spec-driven development with more than one engineer. Per-role responsibilities, artifact lifecycles, cadence patterns, multi-team monorepos, the OSS case, regulated-industry overlays, onboarding, failure modes, and the discipline that keeps it from collapsing under organizational gravity.

---

## Table of Contents

1. [Framing — what changes when SDD becomes a team protocol](#framing--what-changes-when-sdd-becomes-a-team-protocol)
2. [Role profiles](#role-profiles)
3. [Artifact ownership in depth](#artifact-ownership-in-depth)
4. [Lifecycle handoffs](#lifecycle-handoffs)
5. [Cadence patterns](#cadence-patterns)
6. [Multi-team and monorepo specifics](#multi-team-and-monorepo-specifics)
7. [The solo-developer case](#the-solo-developer-case)
8. [Open-source projects](#open-source-projects)
9. [Regulated industries](#regulated-industries)
10. [Onboarding new contributors](#onboarding-new-contributors)
11. [Team failure modes](#team-failure-modes)
12. [A RACI for SDD](#a-raci-for-sdd)
13. [Maintaining discipline at scale](#maintaining-discipline-at-scale)
14. [Golden rules](#golden-rules)

---

## Framing — what changes when SDD becomes a team protocol

Solo SDD has one nice property: there is no handoff. You write the spec; you implement it; you update the doc. The discipline is entirely self-imposed; the cost of slacking is paid by your future self.

Team SDD adds **interfaces between people**. Specs are handed from author to reviewer to implementer; ADRs are proposed by one person and ratified by another; runbooks are written by whoever ran the incident but read by whoever's on call next. Each interface is a place the discipline can leak.

Three things change once there's more than one person:

1. **Ownership becomes the central question.** Who writes vs reviews vs approves each artifact? The discipline rots fastest where ownership is diffuse.
2. **Cadence matters.** Solo: write the doc when you feel like it. Team: write it before the PR, or the PR review blocks. The rhythm has to be enforced somewhere.
3. **Doc drift compounds.** With one person, drift is bounded by what you can hold in your head. With ten people, drift is the *normal* state unless the team works actively to prevent it.

The patterns in this guide are about making those three problems tractable.

---

## Role profiles

Roles overlap; titles vary. What matters is *who, by default, is on the hook* for each piece of the SDD discipline. Below: the roles I'll reference, what they do in an SDD context, and how to collapse them for smaller teams.

### Engineer (Individual Contributor)

The default actor in the SDD workflow. For most projects, ICs are the people who:

- Write **specs** for the features they're implementing
- Implement the spec, task by task
- Update **spec status** to `shipped` after merge
- **Propose ADRs** when they hit a decision worth recording
- Update **CLAUDE.md** with new conventions (proposed; tech lead approves)
- Write **runbook entries** if they were on-call for the incident
- Update **DOMAIN.md** when they introduce a new term in code

If an engineer touches a piece of code more than twice and a convention isn't documented yet, *they* are the one to add it to CLAUDE.md — not the tech lead.

### Tech Lead / Staff Engineer

The default maintainer of the *stable layer* — the documents the agent loads on every session and the team consults whenever a question is bigger than one feature. Tech leads:

- Own **CLAUDE.md** — review every change; run the quarterly audit
- Own **ARCHITECTURE.md** — write/update when boundaries change
- **Ratify ADRs** — engineers propose; the tech lead (or architecture group) decides when one moves from `Proposed` to `Accepted`
- Maintain the **Active ADR list** in CLAUDE.md
- Resolve **convention precedence** disputes (CLAUDE.md vs ADR vs ARCHITECTURE.md)

In a small team without a designated tech lead, this role rotates or sits with the most senior engineer.

### Engineering Manager

Process owner, not artifact owner. The EM doesn't write specs or ADRs; they enforce the discipline that makes them happen:

- Spec exists *before* code in the PR
- Spec status updated *with* the merge, not as a follow-up
- ADRs reviewed in a timely manner (no `Proposed` ADRs older than 2 weeks)
- Quarterly doc audits actually happen
- New hires are pointed at the onboarding flow

If your EM is also writing code, this role collapses into Tech Lead. If your EM is purely organizational, the discipline lives with them.

### Product Manager / Product Owner

Owns the **PRD** (which freezes after v1) and reviews specs for scope alignment. PMs:

- Write the **PRD** during the v0 → v1 stretch
- Review **specs** for *what* the feature delivers — does it match the PRD's intent?
- Don't usually touch ADRs, ARCHITECTURE, CLAUDE.md (those are engineering's)
- May contribute to **DOMAIN.md** if the domain is heavy in business terminology

PMs are the most common bottleneck on spec review. A PM who blocks 5 specs at once is the most common reason teams drift to "code first, spec second."

### Founder / CTO (small teams)

Wears multiple hats. Until the team grows past ~5 engineers, the founder usually:

- Writes the initial **PRD**
- Drafts the first 3–5 **ADRs** that capture the founding-era decisions
- Owns **CLAUDE.md** until a tech lead is hired
- Is the de-facto **PM** and **Tech Lead** for the first year

The collapse is fine; the trap is *forgetting to delegate* once others arrive. A founder still writing every ADR in year 3 is a sign of role-collapse becoming a bottleneck.

### Designer

In code-heavy SDD, designers usually don't write SDD artifacts. They do:

- Provide **specs**' "what the user sees" content for UX-heavy features
- Review **specs** for design-system alignment
- Occasionally own a `docs/design-system.md` or equivalent (one detail file)

If your project is design-heavy (consumer app, marketing site), designers may co-author specs more substantially. In B2B / infrastructure / data systems, the design role is light or absent.

### QA Lead / SET / SRE

On the hook for **TESTING.md**, sometimes **RUNBOOK.md**, and any test-related conventions. They:

- Write/own **TESTING.md** — test strategy, conventions, coverage expectations
- Review **specs** for testability — does the spec describe testable acceptance criteria?
- Co-own **runbooks** with SRE / DevOps
- Pull testing-related ADRs when the discipline changes

QA at SDD-mature teams is light — much of "QA" becomes "specs the agent can verify against."

### Security Engineer

On the hook for **SECURITY.md** and review of ADRs that touch:

- Authentication, authorization
- Data handling (PII, encryption, retention)
- Third-party integrations (new attack surface)
- Secrets management

In small teams this role collapses into Tech Lead. In larger orgs, security has a *block veto* on certain ADR categories.

### DevOps / SRE

Owns **OPERATIONS.md**, deployment runbooks, and infrastructure-related ADRs. They:

- Write **OPERATIONS.md** and most of `docs/runbooks/`
- Review specs for **deployment implications** (new background job? new dependency? new persistent state?)
- Approve ADRs touching scheduling, infrastructure, observability
- Maintain `.claude/settings.json` hooks that enforce operational invariants (regen PDF on doc edit, block dangerous commands, etc.)

### External Contributors (OSS)

Read **CLAUDE.md** and **CONTRIBUTING.md**; usually don't have write access to anything else. The relationship is:

- They produce **specs and PRs**; maintainers review and merge
- They don't propose ADRs (or if they do, maintainers ratify slowly)
- They don't update **CLAUDE.md** directly (maintainers do, based on PR feedback)

OSS governance is a topic of its own; see [Open-source projects](#open-source-projects) below.

---

## Artifact ownership in depth

The compact table in the main guide expanded with more nuance.

### `PRD.md` — Product / founder

Written once, during v0 → v1. Frozen after v1 ships. Engineering reviews for technical feasibility but doesn't write or edit. After freeze, the PRD lives in `docs/prd/` as historical context — new features get specs, not PRD edits.

**Common failure:** treating PRD as living. The PRD becomes a Frankenstein doc of every change request; original intent dilutes.

### `CLAUDE.md` — Tech Lead (or designated maintainer)

One named owner. Engineers propose changes via PR; the owner reviews and merges. The owner also runs the quarterly trim (remove anything older than 6 months without a current rationale).

**Common failure:** "the team owns it." With no named owner, drift goes unmanaged. A team-owned CLAUDE.md is dead within two quarters.

### `ARCHITECTURE.md` — Tech Lead / Staff Engineer

Updated when system boundaries change (new service, new layer, new module). Most updates come *via* an ADR that justifies the change — the ARCHITECTURE.md update is a downstream effect.

**Common failure:** ARCHITECTURE.md updated as a *replacement* for an ADR. The why is lost; only the what remains.

### `DOMAIN.md` (or `GLOSSARY.md`) — Domain expert

Often the PM, sometimes a senior engineer with domain background. Engineers contribute new terms as they encounter them; the domain expert approves to keep definitions tight.

**Common failure:** definitions drift toward technical jargon over time. *"Order"* starts as a business concept and ends as the SQL table. Periodic re-grounding with non-engineers helps.

### `docs/adr/*.md` — Engineer who proposed the decision

Each ADR has the proposing engineer's name in `References` or a footer. They're responsible for shepherding it from `Proposed` to `Accepted`. After Accepted, the ADR is immutable; ownership of *the content* effectively transfers to the team.

**Common failure:** ADRs proposed and forgotten. A stalled `Proposed` ADR > 1 month old rots the catalog. Tech lead's job to close or accept.

### `specs/<date-slug>/spec.md` — Engineer driving the change

The engineer owns the spec for its lifetime — propose, implement, mark `shipped`, link the PR. If they hand off mid-flight (illness, reassignment), the spec gets a new named owner in its status line.

**Common failure:** specs written *after* the PR ("documentation of what got built"). At that point it's not a spec; it's reverse-engineered description.

### `docs/runbooks/<incident-slug>.md` — On-call team

The engineer who resolved the incident writes the runbook entry within 24 hours. The on-call rotation as a whole owns the runbook collection — quarterly audit for staleness.

**Common failure:** runbook written by the incident commander but never reviewed; commands stop working a month later; next on-call follows them off a cliff.

### `RUNBOOK.md` / `OPERATIONS.md` — SRE / DevOps

For larger systems. The team that owns the deploy pipeline owns the operations manual.

### `TESTING.md` — Tech Lead or QA Lead

Updated when test conventions change. New engineers read it during onboarding; if they have to ask basic questions ("where do integration tests live?"), the doc is incomplete.

---

## Lifecycle handoffs

Each artifact has its own life cycle. Knowing where the handoffs are tells you where to look when the discipline leaks.

### Spec lifecycle

```
[draft] → [reviewed] → [in implementation] → [shipped] → [archived]
   ↑          ↑              ↑                    ↑
engineer     PM (scope)   engineer            engineer
            tech lead    (task by task)      (update status,
            (design)                          link PR)
```

- **Draft** — engineer writes spec.md + plan.md + tasks.md. Open questions listed at end.
- **Reviewed** — PM checks scope (does this match what we agreed to build?). Tech lead checks design (is the plan plausible? does it conflict with ADRs?). Open questions get answered or moved to ADRs.
- **In implementation** — engineer works through tasks.md, with checkpoints. PR references the spec folder.
- **Shipped** — PR merged. Engineer appends `STATUS: shipped (PR #N, YYYY-MM-DD)` to spec.md. Done.
- **Archived** — frozen. Lives in `specs/<date-slug>/` as historical record.

**Where it leaks:** the *Reviewed* step. PMs are busy; tech leads are busy; specs sit in `Proposed` for days. The fix is timeboxing — *"24h to review or it ships as-is"* — combined with a small specs queue, so reviewers aren't drowning.

For the *authoring* side — how engineers actually draft each of `spec.md` / `plan.md` / `tasks.md`, the AI-assisted prompts that speed up each step, and the trio consistency check that catches contradictions before implementation — see [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md).

### ADR lifecycle

```
[proposed] → [reviewed] → [accepted] → [(maybe) superseded]
    ↑           ↑              ↑              ↑
 engineer    tech lead    tech lead      new ADR by
            + relevant    (date the      whoever proposes
            team          status)        the change
```

- **Proposed** — engineer drafts the ADR. Status: `Proposed`. Anyone can comment on content.
- **Reviewed** — tech lead leads discussion (PR-based or in an architecture meeting). Open questions in Consequences/Alternatives get resolved.
- **Accepted** — tech lead marks Status: `Accepted` with today's date. Body freezes from here.
- **Superseded** — a new ADR (proposed by anyone) replaces this one. Only the Status header of the old ADR changes; new ADR has `Supersedes: ADR-NNN`.

**Where it leaks:** `Proposed → Accepted` taking weeks. A stalled ADR is worse than no ADR; team starts working around it. Cure: a *Proposed* SLA (1–2 weeks max).

### PRD lifecycle

```
[draft] → [accepted] → [v1 ships] → [frozen]
   ↑          ↑             ↑           ↑
founder/   founding         (no doc    archived
 PM        team review      change)    in docs/prd/
```

The PRD is the one artifact that *doesn't* evolve. New direction = new PRD or roadmap entry, not edits to the old one. Engineering involvement is at draft (technical feasibility) and accepted (commitment).

**Where it leaks:** the PRD becomes living. Six months later the document reads like a press release for a system that no longer matches reality. Cure: a hard freeze rule, and a `Frozen: <date>` line in the PRD header.

### Runbook lifecycle

```
[incident] → [draft] → [reviewed] → [in rotation] → [(quarterly) verified] → [archived]
    ↑           ↑           ↑              ↑                  ↑                    ↑
on-call    on-call     senior         on-call          on-call rotation       SRE if
engineer   engineer    engineer       team             walks through            deprecated
(within   (24h after)  (~next day)   (used in         once per quarter
24h)                                  prod)
```

The 24h rule is the key one — the incident commander writes the runbook entry while details are fresh. A week later, the recovery steps are mythology.

**Where it leaks:** the runbook never gets written. *"We can do it next sprint."* Next sprint, the entry that would have saved the next on-call is lost. Cure: post-incident review includes "write the runbook entry" as a blocking task.

---

## Cadence patterns

### Spec review

- **PR-based** (most teams) — spec is a file in the PR; reviewers comment inline. Async-friendly.
- **Standup-based** — spec is presented in 5 minutes at a standup; review is real-time. Faster decisions; bad for remote-distributed teams.
- **Spec hours** — a weekly 90-minute slot where specs queued during the week get reviewed together. Good for batching; bad for urgent work.

### ADR review

- **PR-based** — ADR is committed as `Proposed`; reviewers comment; eventual flip-to-Accepted PR. Standard tooling, async, leaves a trail.
- **Architecture meeting** — biweekly or monthly; ADRs queued; discussed in person/video; decision made and committed during the meeting. Faster but remote-unfriendly.
- **Hybrid** — small ADRs go PR-based; ADRs affecting multiple teams escalate to architecture meeting.

Pick one and document it in `CLAUDE.md` or a `docs/process.md`. The format matters less than the consistency.

### Doc audits

- **Quarterly walkthrough** — once per quarter, designated owner reads CLAUDE.md, runs the ADR audit, checks DOMAIN.md against codebase. Half a day max.
- **Rotation** — different person each quarter so no single person is the choke point.
- **Automation-assisted** — a subagent runs the staleness check (see `working-with-agents-guide.md § Subagents`) before the human reads.

### Spec → PR coupling

Most teams converge on: **the PR template includes a docs checklist**, and the PR doesn't merge with unchecked items. Sample:

```markdown
## Documentation

- [ ] Spec at `specs/YYYY-MM-slug/`
- [ ] Spec status updated to `shipped (PR #X, YYYY-MM-DD)` (post-merge)
- [ ] ARCHITECTURE.md updated if boundaries changed
- [ ] DOMAIN.md updated if new terms introduced
- [ ] CLAUDE.md updated if a new convention emerged
- [ ] Active ADR list updated if an ADR was added/superseded
```

The checklist is what keeps "stale by Tuesday" from happening.

---

## Multi-team and monorepo specifics

When more than one team contributes to the same repo, two new questions emerge: *whose CLAUDE.md wins?* and *who owns cross-team ADRs?*

### Sub-`CLAUDE.md` per service (or per team)

Most monorepos benefit from one `CLAUDE.md` at the root (cross-service rules, shared stack, global conventions) plus a `CLAUDE.md` per service (local stack, service-specific patterns).

```
/
├── CLAUDE.md                          # cross-service: logging, observability, deploys
├── services/
│   ├── orders/
│   │   ├── CLAUDE.md                  # orders-specific: stack version, repos
│   │   └── src/...
│   ├── payments/
│   │   ├── CLAUDE.md                  # payments-specific
│   │   └── src/...
```

When the agent works in `services/orders/`, it loads both the root CLAUDE.md and the local one. Conventions in the local file win for that subtree.

### Cross-team ADRs

Two patterns:

- **Global ADR folder** (`docs/adr/`) — for decisions affecting more than one team. Ratified by an architecture group with representatives from each team.
- **Per-team ADR folders** (`services/<name>/docs/adr/`) — for decisions internal to one service. Ratified by that team's tech lead.

A common rule: if the decision would change another team's behavior, it's global. If it changes only your service, it's local.

### Convention precedence in monorepos

Document the precedence explicitly:

> When root `CLAUDE.md`, service `CLAUDE.md`, ADRs, and ARCHITECTURE.md disagree:
> 1. Service `CLAUDE.md` wins for that service's code
> 2. Cross-cutting ADRs (security, observability) win globally
> 3. Root `CLAUDE.md` is the fallback
> 4. ARCHITECTURE.md is descriptive, not prescriptive

If your team can't articulate the precedence in 3 lines, drift is inevitable.

### Cross-service tech-lead syncs

A 30-minute biweekly sync between service tech leads, focused only on:

- ADRs proposed this cycle that might affect other services
- Stale ADRs ripe for supersede
- New cross-cutting conventions to add to root CLAUDE.md

This is the single highest-leverage organizational pattern for multi-team SDD.

---

## The solo-developer case

A team of one still benefits from SDD, but the role collapse is dramatic — you play every role above. The discipline shifts from *"protect my teammates from confusion"* to *"protect future-me from forgetfulness."*

What collapses:

- **Spec review = you-now reviewing what you-yesterday wrote.** The 24-hour cool-off helps; sleep on a spec before implementing.
- **ADR ratification = you-now confirming what you-yesterday decided.** Still useful — forces you to articulate the decision in writing instead of holding it in your head.
- **CLAUDE.md ownership = you alone.** No bottleneck, but also no second opinion. The agent itself can act as the second pair of eyes (*"based on our work today, what conventions did I correct you on?"*).
- **Doc audit = a 15-minute quarterly review you do solo.** Easier than for a team; easier to skip.

What stays useful:

- **Specs** — even short ones (10-20 lines) keep you focused and produce a checkpoint when you return after weeks away.
- **ADRs** — for any decision you'd struggle to reconstruct six months later. The 10-minute investment pays off the first time you forget why.
- **CLAUDE.md** — the agent uses it heavily; even a 100-line CLAUDE.md saves hours over a quarter of agent sessions.

What you can skip:

- PRD (you remember why you're building it; project-wide intent doesn't drift in a single head)
- Cross-team ADRs (n/a)
- Architecture meetings (n/a)
- Roadmap as a separate doc (lives in your todo list)

Solo SDD is mostly *CLAUDE.md + specs + a handful of ADRs*. That's it.

---

## Open-source projects

OSS adds two roles to the matrix: **maintainer** and **external contributor**. The discipline shifts accordingly.

### Maintainers

- Own all repo-level artifacts (CLAUDE.md, ARCHITECTURE.md, DOMAIN.md, ADRs)
- Review external contributions for spec compliance
- Are the only ones with merge authority
- Adapt CLAUDE.md to *help external contributors' agents* behave well, not just internal team agents

### External contributors

- Read CLAUDE.md and CONTRIBUTING.md before opening a PR
- Submit specs alongside non-trivial changes
- Don't propose ADRs (or do, but rarely get ratified quickly)
- Don't directly edit CLAUDE.md (maintainers do, based on PR feedback)

### OSS-specific docs

In addition to the SDD core, OSS projects need:

- **CONTRIBUTING.md** — how to contribute; complements CLAUDE.md
- **CODE_OF_CONDUCT.md** — community norms
- **CODEOWNERS** — who reviews what
- **GOVERNANCE.md** (for larger projects) — how decisions get made, voting rules, maintainer rotation

### CLAUDE.md for OSS

Write it with external contributors' agents in mind:

- **Project goals** — so the agent doesn't propose out-of-scope features
- **What we don't do** — features explicitly out of scope; libraries we won't depend on; architectural taboos
- **License-aware** — *"all dependencies must be MIT-compatible; do not propose GPL libraries"*
- **Style strict** — external code volume is high; lenient style rules cause review fatigue

The CLAUDE.md trigger for OSS migration is usually *"first low-quality AI-driven PR."* That PR is the signal to write the doc; subsequent PRs improve in quality measurably.

### ADRs in OSS

ADRs are mostly maintainer-driven. External contributors *can* propose ADRs in PR form, but the ratification process is slower (maintainers may want to discuss async with no SLA). Document this in CONTRIBUTING.md so contributors aren't surprised.

For high-traffic OSS projects, an **RFC process** (separate from ADRs, with its own folder and template) handles community-driven decisions. RFCs are public-facing; ADRs are decision-record-after-the-fact.

---

## Regulated industries

Finance, healthcare, defense, regulated SaaS — SDD overlays cleanly with compliance requirements; you just add roles and gates.

### Additional roles

- **Compliance officer** — reviews ADRs touching data handling, retention, audit logging
- **Security architect** — sign-off on ADRs touching auth, encryption, secrets
- **Auditor (external)** — periodically reviews ADRs and runbook history for evidence of controls

### Additional artifacts

- `docs/compliance/` — control mappings (SOC 2, HIPAA, PCI, etc. ↔ ADRs and code)
- `docs/audit-trail/` — records of significant changes with sign-off
- Stricter `RUNBOOK.md` — recovery procedures may need approvals before execution

### Workflow adjustments

- ADRs touching regulated areas need *additional* sign-offs (security, compliance) before `Accepted`
- All ADRs and specs are append-only with full history (most VCS already does this; just be deliberate)
- Audit log of `STATUS: shipped` events — easy via `git log` on `specs/`

### CLAUDE.md additions for regulated work

```markdown
## Regulatory constraints
- All PII must be encrypted at rest (see ADR-018)
- All API access logs retained for 7 years (see ADR-022)
- Production data NEVER copied to non-prod environments
- Any change touching `auth/`, `pii/`, or `audit/` requires Security ADR sign-off

Do NOT propose changes in these areas without explicit instruction to do so;
flag them as "requires compliance review" and stop.
```

The agent obeying these constraints is one of the highest-value applications of SDD in regulated work.

---

## Onboarding new contributors

A new hire (or new OSS contributor) goes through a roughly fixed sequence. Document it; reduce the variance.

### Day 1: read the stable layer (~2 hours)

1. **README.md** (10 min) — what is this project
2. **CLAUDE.md** (15 min) — conventions, stack, what NOT to do
3. **ARCHITECTURE.md** (30 min) — components, boundaries
4. **DOMAIN.md** (15 min) — terminology
5. **Active ADRs** — skim the Active list, read 2–3 most relevant in full (30 min)

### Day 2–3: orient on real artifacts

- Browse `specs/` — pick the last 3 shipped specs to see real examples
- Browse `docs/adr/` — see how ADRs are structured in this project
- Read `docs/runbooks/` if relevant to their role

### Day 4–5: a small first PR

Ideal first PR is something small with a real spec — even a 20-line spec. Goals:

- Practice the spec → PR → docs-update loop
- Get a code review from someone senior
- Learn where the team's review queue lives

If a new contributor can ship the small first PR following the SDD workflow, they've internalized the discipline. If they get stuck, the SDD docs (not just the code docs) have a gap — surface and fix it.

### Ongoing: pair on the first ADR

If a new contributor's work prompts an ADR within the first month, pair with a senior engineer on writing it. ADRs have more invisible conventions than other artifacts; pairing once teaches them.

---

## Team failure modes

The compact list in the main guide expanded with concrete remedies.

### 1. Diffused ownership

*Symptom:* CLAUDE.md updated by no one for 3 months because *"the team owns it."*

*Remedy:* one name per artifact. Put the name in the file header. Rotate annually if it's a burden.

### 2. The documentation hero

*Symptom:* one engineer writes everything; the rest never internalize the discipline; that engineer leaves; docs die within a quarter.

*Remedy:* mandatory rotation of audit ownership. Every IC writes at least one ADR per quarter. PR template forces docs updates; the hero doesn't get to absorb the work.

### 3. Stale-by-Tuesday

*Symptom:* docs updated Monday; codebase changes Tuesday; docs lie by Wednesday.

*Remedy:* updates ride *with* the PR. PR template checklist. Review blocks if items are unchecked.

### 4. Spec as theater

*Symptom:* specs written *after* the PR — reverse-engineered from code to satisfy the process.

*Remedy:* PR description includes the spec path; reviewer verifies the spec was committed *before* the implementation commits. If not, send the PR back. Two cycles of this and the team learns.

### 5. ADR sprawl

*Symptom:* 40 ADRs after 6 months; half obsolete; nobody references them.

*Remedy:* the quarterly audit. Aggressively supersede stale ADRs. Aim for ~10-20 Active ADRs at any time in a mid-size system.

### 6. Cross-team ADR collisions

*Symptom:* service A's ADR-007 contradicts service B's ADR-003; both teams point at the other's doc as the problem.

*Remedy:* the cross-team tech-lead sync (biweekly, 30 min) catches these early. When collision is found, *one* ADR supersedes the other — never two co-equal contradictions.

### 7. The bottleneck-PM

*Symptom:* PM is reviewer on every spec; queue of 8 specs awaiting their review; engineers start working without PM sign-off.

*Remedy:* either reduce the PM's review scope (skip trivial specs), parallelize (each spec routed to one named PM, not all PMs), or accept async (24h-or-it-ships).

### 8. Hero on-call writes nothing

*Symptom:* the senior engineer who handles every incident never writes the runbook entries; the rest of the on-call rotation gets paged on the same incidents repeatedly.

*Remedy:* the 24h rule, enforced. Incident review meeting includes "the runbook entry exists" as a yes/no checkbox.

### 9. Founder-still-writing-everything

*Symptom:* year 3, team of 10, founder writes every ADR and owns CLAUDE.md.

*Remedy:* explicit transition. Founder picks a tech lead; for 1 quarter, tech lead writes alongside; then founder steps off.

### 10. New hire gets a 60-page reading list

*Symptom:* new hire's first week is reading 30 docs; they never write anything; they leave by month 6.

*Remedy:* the onboarding flow above (2-hour day 1, small PR by day 5). If the reading list has grown past day 1, it's a sign the stable layer needs trimming, not that the hire is slow.

---

## A RACI for SDD

For teams that like explicit matrices. RACI = Responsible / Accountable / Consulted / Informed.

| Activity | Engineer | Tech Lead | EM | PM | SRE | Security |
|----------|----------|-----------|----|----|-----|----------|
| Write a spec | **R/A** | C | I | C | I | I |
| Review a spec | C | C | I | **R/A** (scope) | C (if ops impact) | C (if security impact) |
| Implement a spec | **R/A** | C | I | I | I | I |
| Write an ADR | **R/A** (propose) | C | I | I | C | C (if relevant) |
| Ratify an ADR | C | **R/A** | I | I | C | C (if relevant) |
| Write CLAUDE.md | C (propose) | **R/A** | I | I | C | C |
| Run quarterly audit | C | **R/A** | C | I | C | C |
| Write a runbook entry | **R/A** (if on-call) | C | I | I | **R/A** | I |
| Update DOMAIN.md | C | C | I | **R/A** (or domain expert) | I | I |
| Operations setup | I | C | I | I | **R/A** | C |

(R = Responsible; A = Accountable; C = Consulted; I = Informed. Where R and A diverge — Engineer "R" + Tech Lead "A" — the engineer does the work; the tech lead is on the hook for whether it gets done.)

**Adapt freely.** This matrix is a starting point. For your team, you might collapse two roles into one or move some cells based on actual ownership. The point isn't the specific cells; it's that each cell has a name in it.

---

## Maintaining discipline at scale

Three patterns keep team SDD from collapsing as the team grows.

### 1. Ownership rotation

Every quarter, rotate ownership of:

- CLAUDE.md maintenance (one engineer for a quarter)
- The quarterly doc audit (different engineer each time)
- The runbook freshness review

The rotation prevents both burnout and hero-dependency. Bonus: each engineer learns the SDD discipline more deeply during their rotation.

### 2. Onboarding-as-audit

Every new hire is an opportunity to audit the docs. If they're stuck, the docs have a gap; fix it. Track which doc each new hire's first three questions point at; the most-questioned docs are the ones that need work.

### 3. The "PR docs checkbox" ritual

The PR template includes a docs section. Review blocks if items are unchecked. After 3 months of consistent enforcement, the discipline becomes automatic.

```markdown
## Documentation
- [ ] Spec at `specs/YYYY-MM-slug/`
- [ ] Spec status updated post-merge
- [ ] ARCHITECTURE.md updated (or N/A)
- [ ] DOMAIN.md updated (or N/A)
- [ ] CLAUDE.md updated (or N/A)
- [ ] Active ADR list updated (or N/A)
```

The `(or N/A)` matters — engineers won't check items that don't apply, so make the no-op explicit.

---

## Golden rules

1. **One name per artifact, not "the team."** Diffuse ownership is the failure mode that kills team SDD fastest.
2. **Engineers propose; tech leads ratify.** ADRs especially. The flow keeps decisions consistent without bottlenecking on one person.
3. **Docs ride with the PR.** Spec status, ADR list, CLAUDE.md, ARCHITECTURE.md — all updated in the same PR that changes the code.
4. **Rotate ownership annually.** Hero-dependency is invisible until the hero leaves.
5. **PMs review scope, not implementation.** *"Does this match what we agreed to build?"* not *"is this the right way to build it?"*
6. **Tech leads own the stable layer.** CLAUDE.md, ARCHITECTURE.md, the Active ADR list. Authority + accountability.
7. **The 24-hour runbook rule.** Incident → runbook entry within a day. Later = lost.
8. **Onboarding is an audit.** New hires' confusion is data; treat it as a signal, not a personal failing.
9. **Quarterly doc trim.** 30 minutes per quarter beats a multi-day cleanup once a year.
10. **Solo SDD protects future-you; team SDD protects everyone else.** The artifacts serve different purposes; the discipline is the same.

---

*This guide complements [`spec-driven-development-guide.md`](spec-driven-development-guide.md) (the overall methodology), [`claude-md-guide.md`](claude-md-guide.md) (the file maintained by the tech lead), [`adr-guide.md`](adr-guide.md) (the artifact engineers propose and tech leads ratify), and [`working-with-agents-guide.md`](working-with-agents-guide.md) (how multi-engineer teams keep agent context consistent across sessions and tools).*
