# Postmortem: [one-line incident summary]

> Copy this file to `docs/postmortems/YYYY-MM-DD-<incident-slug>.md` and fill in.
> Postmortems are **blameless** — accuse systems and decisions, not individuals.
> Otherwise the next one isn't written honestly, and the discipline collapses.

**Date:** YYYY-MM-DD
**Severity:** [P0 / P1 / P2 / P3]
**Duration:** [HH:MM start] → [HH:MM end] (X minutes total)
**Author:** [name]
**Status:** [draft / reviewed / final]

## Summary

[1-2 paragraphs in plain English: what happened, who was affected, how it was resolved. Written so a non-technical reader can follow.]

## Timeline

All times in [timezone, e.g. UTC].

- **HH:MM** — [event: alert fired / engineer paged / something noticed]
- **HH:MM** — [diagnosis step]
- **HH:MM** — [decision made]
- **HH:MM** — [fix applied]
- **HH:MM** — [verified / incident closed]

## Root cause

[The actual underlying cause, not the symptom. Use 5 Whys if needed:
"Symptom was X. Why? Y. Why Y? Z. Why Z? …"
Stop when you reach a structural / process / design cause, not a person.]

## What went well

- [Something that worked — runbook entry that helped, alerting that fired correctly, recovery procedure that succeeded]
- ...

## What went wrong

- [Failures or gaps — missing alert, slow detection, runbook entry that didn't apply, wrong assumption]
- ...

## Lessons learned

- [Generalizable insights — what this incident teaches about our system, processes, or assumptions]
- ...

## Action items

Each action item has an owner and a due date. Track in a ticket; reference the ticket here.

- [ ] **[Owner]** — [action] — due YYYY-MM-DD — [TICKET-XXXX]
- [ ] **[Owner]** — [action] — due YYYY-MM-DD — [TICKET-XXXX]

## Related

- Runbook entry: `docs/runbooks/<slug>.md` (created or updated as a result of this incident)
- Related ADRs (if the incident exposes a decision worth revisiting): ADR-NNN
- Prior incidents with similar shape: [list]
