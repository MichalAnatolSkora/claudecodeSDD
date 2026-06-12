# specs/ — the per-feature trios

> One trio per feature from the [PRD](../docs/prd/2026-01-order-export.md), written **when the feature is picked up** — not before. Illustrative.

| PRD feature | Spec | State |
|-------------|------|-------|
| 1 — walking skeleton (XML + SFTP) | [`2026-01-export-skeleton.md`](2026-01-export-skeleton.md) — one-file trio | Shipped |
| 2 — CSV format option | [`2026-02-csv-format.md`](2026-02-csv-format.md) — one-file trio | Shipped |
| 3 — retry failed deliveries | [`2026-02-delivery-retry/`](2026-02-delivery-retry/) — full three-file trio | Active |
| 4 — per-partner schedule config | — | not started (backlog) |
| 5 — operator batch-status view | — | not started (backlog) |

**Why 4 and 5 have no spec yet.** SDD is forward-only: you write a spec **when you pick the work up**, not speculatively. Features 4–5 are PRD backlog rows; their trio gets created the day someone starts them. This is also why **live status lives here, not in the PRD** — the PRD froze its slice list, and what's shipped vs in flight is visible from `specs/`.

**Two trio shapes on display.** Small, shipped features (1, 2) use the **one-file trio** — spec / plan / tasks as three sections in a single file. The current, richer feature (3) uses the **full three-file trio** plus an ADR. Same discipline, sized to the change — see [`spec-plan-tasks-guide.md`](../../../guides/spec-plan-tasks-guide.md).

---
*Illustrative index. A real `specs/` rarely needs a README — it's here to make the PRD ↔ specs mapping explicit.*
