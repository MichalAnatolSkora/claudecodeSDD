# specs/ — the per-feature trios

> One trio per feature from the [PRD](../docs/prd/2026-01-order-export.md), written **when the feature is picked up** — not before. Illustrative.

| PRD feature | Spec | State |
|-------------|------|-------|
| 1 — walking skeleton (XML + SFTP) | [`2026-01-export-skeleton.md`](2026-01-export-skeleton.md) — one-file trio | Shipped |
| 2 — CSV format option | [`2026-02-csv-format.md`](2026-02-csv-format.md) — one-file trio | Shipped |
| 3 — retry failed deliveries | [`2026-02-delivery-retry/`](2026-02-delivery-retry/) — full three-file trio | Active |
| 4 — per-partner schedule config | — | not started (backlog) |
| 5 — operator batch-status view | [`2026-03-batch-status-view.md`](2026-03-batch-status-view.md) — discovery one-file trio | Shipped |

**Why 4 has no spec yet.** SDD is forward-only: you write a spec **when you pick the work up**, not speculatively. Feature 4 is a PRD backlog row; its trio gets created the day someone starts it. This is also why **live status lives here, not in the PRD** — the PRD froze its slice list, and what's shipped vs in flight is visible from `specs/`.

**Two trio shapes on display.** Small, shipped features (1, 2, 5) use the **one-file trio** — spec / plan / tasks as three sections in a single file. The richer feature (3) uses the **full three-file trio** plus an ADR. Same discipline, sized to the change — see [`spec-plan-tasks-guide.md`](../../../guides/spec-plan-tasks-guide.md).

**Discovery and re-slicing on display.** Feature 5 is the **discovery** case — *"a view of batch status"* had no testable AC until an operator saw real data, so it was [spiked first, and its first AC was wrong and fixed mid-flight](2026-03-batch-status-view.md) (the struck line is kept on purpose). It also shows **re-slicing**: it was pulled ahead of feature 4 (per-partner scheduling) once the first weeks of live exports had operators repeatedly asking *"did my batch actually go out?"* — visibility became more urgent than scheduling. The slice list isn't frozen, only the PRD is. (It builds on feature 1 only, so it could ship while feature 3 was still in flight.)

---
*Illustrative index. A real `specs/` rarely needs a README — it's here to make the PRD ↔ specs mapping explicit.*
