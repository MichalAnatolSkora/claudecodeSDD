# SDD in Teams (1–10 People)

> What changes when spec-driven development goes from a solo memory aid to a small team's shared source of truth — kept deliberately light. This is the manual for **teams of 1–10**. Heavier org machinery (a formal RACI, multi-team monorepo precedence, regulated-industry sign-off gates) is named at the end under "When you outgrow this," not documented here — by design.

---

## Table of Contents

1. [What changes when you add people](#what-changes-when-you-add-people)
2. [Who owns what — one name per artifact](#who-owns-what--one-name-per-artifact)
3. [How review works](#how-review-works)
4. [Spec lifecycle](#spec-lifecycle)
5. [ADRs: the shared decision log](#adrs-the-shared-decision-log)
6. [The solo case (the floor)](#the-solo-case-the-floor)
7. [Onboarding someone new](#onboarding-someone-new)
8. [Failure modes that actually hit small teams](#failure-modes-that-actually-hit-small-teams)
9. [When you outgrow this](#when-you-outgrow-this)
10. [Golden rules](#golden-rules)

---

## What changes when you add people

Solo SDD has one nice property: there's no handoff. You write the spec, you implement it, you update the doc. The discipline is self-imposed, and the only victim of slacking is future-you.

Add even one more person and the docs stop being a memory aid and become a **shared source of truth** — the thing that lets two people work on the same system without stepping on each other or relitigating settled decisions.

Three things change once there's more than one of you:

1. **Ownership becomes the question.** Who writes vs. reviews each artifact? Discipline rots fastest where ownership is "the team's" — i.e., nobody's.
2. **Timing matters.** Solo, you can write the spec whenever. On a team, the spec goes *before* the PR — otherwise review has nothing to check against.
3. **Drift compounds.** One head can track what's stale. Two-to-ten heads can't; docs drift unless updating them rides along with the code.

Everything below is the lightest thing that keeps those three from biting. The test for any rule here: *would it help a two-person team?* If not, skip it.

---

## Who owns what — one name per artifact

In a team of 1–10, everyone wears several hats. Job titles don't matter; one thing does: **each artifact has one named owner, never "the team."** A team-owned doc is a dead doc within two quarters.

You don't need nine roles. In practice a small team has three hats, and one person often wears two:

- **Driver** — whoever's making the change. Writes the spec/plan/tasks, implements, marks it shipped.
- **Reviewer** — someone *other than* the driver. Checks scope ("is this what we meant to build?") and sanity ("does the plan fight an ADR?").
- **Keeper** — the most senior engineer or the founder. Owns the stable layer (`CLAUDE.md`, `ARCHITECTURE.md`, the active-ADR list) and breaks ties.

A rough owner per artifact:

| Artifact | Owner |
|----------|-------|
| `spec.md` / `plan.md` / `tasks.md` | the driver of that change |
| an ADR | the engineer who proposed it (until accepted) |
| `CLAUDE.md`, `ARCHITECTURE.md`, active-ADR list | the keeper |
| `PRD.md` | the founder or PM (frozen after v1) |
| `DOMAIN.md` / glossary | whoever knows the domain best |
| a runbook entry | whoever resolved the incident |

That's the whole ownership model. Put the owner's name in the file header if it isn't obvious — but below ~5 people it usually is, and `git blame` settles the rest (see the trio guide § "Optional: status and owner").

---

## How review works

Keep it boring and async:

- **PR-based.** The spec/plan/tasks are files in the PR; the reviewer comments inline. No meeting required.
- **Author + one reviewer.** You don't need a committee. One other person is enough to catch a missing acceptance criterion or an ADR conflict.
- **Timebox it.** A spec waiting on review is the most common stall. Rule of thumb: *24 hours to review, or it ships as-is.* A small review queue beats a perfect one nobody gets to.

The one ritual worth enforcing: a **docs checklist in the PR template**, so updates ride *with* the code instead of becoming a someday-follow-up.

```markdown
## Documentation
- [ ] Spec at `specs/YYYY-MM-slug/`
- [ ] Spec status set to `shipped` (post-merge)
- [ ] ARCHITECTURE.md updated (or N/A)
- [ ] CLAUDE.md updated if a new convention emerged (or N/A)
- [ ] Active ADR list updated if an ADR changed (or N/A)
```

The `(or N/A)` matters — people won't tick boxes that don't apply, so make the no-op explicit. This single checkbox is what stops "stale by Tuesday."

---

## Spec lifecycle

A spec moves through a few states. Who's involved at each:

1. **Draft** — the driver writes `spec.md` (plus `plan.md` / `tasks.md` if the change earns them); open questions listed at the end.
2. **Reviewed** — one other person checks scope and sanity; open questions get answered or pushed into an ADR.
3. **In implementation** — the driver works the tasks; the PR links the spec folder.
4. **Shipped** — PR merged; the driver appends `STATUS: shipped (PR #N, YYYY-MM-DD)` to the spec.
5. **Archived** — frozen history in `specs/<slug>/`. If the feature changes later, write a *new* spec; never edit a shipped one.

Where it leaks: the **Reviewed** step (busy reviewers, specs piling up). The 24-hour timebox above is the fix.

For *how* to actually write each of `spec.md` / `plan.md` / `tasks.md` — with the AI-authoring prompts and the consistency check — see [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md), the centerpiece guide.

---

## ADRs: the shared decision log

The biggest team payoff of ADRs is that you stop **relitigating**. A decision made once and written down doesn't get reopened every time someone new looks at the code.

The flow stays light:

- **Engineer proposes** — drafts the ADR, status `Proposed`.
- **The keeper ratifies** — flips it to `Accepted` with a date after a quick PR discussion. In a tiny team this can be you, a day later, confirming what you decided.
- **Supersede, don't edit** — a new decision is a new ADR with `Supersedes: ADR-NNN`; the old one's status changes, its body doesn't.

The only rule that really matters at this size: **don't let `Proposed` ADRs rot.** One sitting undecided for more than a week or two is worse than no ADR — people start working around it. Decide it or drop it.

Full how-to (format, worked examples, the supersede pattern): [`adr-guide.md`](adr-guide.md).

---

## The solo case (the floor)

A team of one still benefits — the role collapse is total (you're driver, reviewer, and keeper), and the discipline shifts from *"protect my teammates"* to *"protect future-me."*

What stays worth doing:

- **Specs** — even 10–20 lines. They keep you focused and give you a checkpoint when you return after weeks away.
- **ADRs** — for any decision you'd struggle to reconstruct in six months. Ten minutes now saves an afternoon later.
- **`CLAUDE.md`** — the agent leans on it every session; even 100 lines saves hours across a quarter.

What to skip solo: the PRD (intent lives in one head and doesn't drift), and anything with "meeting" or "cross-team" in the name. Use the agent as your second pair of eyes — *"based on today's work, what conventions did I correct you on?"*

Solo SDD is basically **`CLAUDE.md` + specs + a handful of ADRs.** That's it.

---

## Onboarding someone new

Keep it to a day, not a reading week:

1. `README.md` → `CLAUDE.md` → `ARCHITECTURE.md` — what it is, the conventions, the shape.
2. Skim the active ADRs; read the 2–3 most relevant in full.
3. Browse the last few shipped specs in `specs/` for real examples.
4. Ship a small first PR *with* a real spec — even 20 lines — so they run the spec → PR → docs-update loop once.

Treat their confusion as data: **if a new person gets stuck, the docs have a gap — fix the doc, don't blame the hire.** If the day-1 reading list has crept past a couple of hours, the stable layer needs trimming.

---

## Failure modes that actually hit small teams

1. **"The team owns it."** No named owner → `CLAUDE.md` untouched for months. *Fix:* one name per artifact.
2. **Spec as theater.** Spec written *after* the PR to satisfy process — that's reverse-engineered description, not a spec. *Fix:* the reviewer checks the spec was committed before the implementation commits. Two bounced PRs and the habit sticks.
3. **Stale by Tuesday.** Docs updated Monday, code changes Tuesday, docs lie by Wednesday. *Fix:* the PR docs checklist — updates ride with the code.
4. **The documentation hero.** One person writes every doc; the rest never learn the discipline; they leave; docs die in a quarter. *Fix:* the driver of a change writes its docs — the hero doesn't get to absorb the work.
5. **Founder won't delegate.** Year 3, still writing every ADR and owning `CLAUDE.md`. *Fix:* hand the keeper hat to someone; pair for a quarter, then step off.

---

## When you outgrow this

Past ~10 people, some of the ceremony this guide deliberately skips starts to earn its place:

- an explicit **RACI matrix**, once "who's on the hook?" stops being obvious from context;
- **multi-team / monorepo** rules — a per-service `CLAUDE.md`, an explicit precedence order, a cross-team ADR process;
- **regulated-industry** gates — compliance/security sign-off on certain ADRs, audit trails of shipped changes;
- a **formal onboarding** track and a rotation of doc-audit ownership so it doesn't fall on one person.

None of that is documented here on purpose. Adding it for a five-person team is exactly how SDD turns into the bureaucracy it was meant to replace. When you genuinely need one of these, lift that single piece — not the whole apparatus.

---

## Golden rules

1. **One name per artifact, never "the team."** The fastest way team SDD dies.
2. **The driver writes the docs.** Whoever makes the change writes its spec and updates what it touched — not a dedicated scribe.
3. **Docs ride with the PR.** Spec status, ADR list, `CLAUDE.md` — same PR as the code.
4. **Engineer proposes, the keeper ratifies.** Keeps decisions consistent without bottlenecking on one person.
5. **24 hours to review, or it ships.** A timebox beats a perfect review nobody reaches.
6. **Onboarding is an audit.** A new person's confusion points at the doc that needs work.
7. **Stay light on purpose.** If a rule wouldn't help a two-person team, you probably don't need it yet.

---

*Companion to [`spec-driven-development-guide.md`](spec-driven-development-guide.md) (the overall method), [`spec-plan-tasks-guide.md`](spec-plan-tasks-guide.md) (how to write the trio), [`adr-guide.md`](adr-guide.md) (the decision log), and [`claude-md-guide.md`](claude-md-guide.md) (the stable-layer file the keeper owns).*
