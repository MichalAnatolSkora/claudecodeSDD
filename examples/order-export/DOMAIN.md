# DOMAIN.md — Order Export platform

> Business vocabulary for the agent. Definitions come from how terms are used in code and with partners. Part of `examples/order-export/` (illustrative).

| Term | Meaning |
|------|---------|
| **Order** | A single customer order eligible for export to a partner. Stored in `app.order_log`. |
| **HDR / DTL** | Record types inside an export file. **HDR** = one header row per batch (partner, date, order count); **DTL** = one detail row per order. A file is one HDR followed by many DTL. |
| **RDY** | Order status meaning *ready to export* — picked up by the next batch. Order lifecycle: `NEW → RDY → SENT` (or `FAILED`). |
| **Batch** | The set of orders exported to one partner in one run, plus the file produced and the delivery record. Keyed by `batch_id`. |
| **Partner** | An external recipient (bank or logistics provider) with a format, a schedule, and SFTP credentials. Stored in `app.partner`. Example: `acme-bank`. |
| **Export** | Rendering orders into a partner's file format (XML or CSV) and writing the batch file. |
| **Delivery** | Transmitting a batch file to the partner over SFTP and recording the outcome in `app.delivery_attempt`. |
| **Dead-letter** | A delivery that exhausted its retries: the batch is marked `FAILED` and flagged for operator follow-up — never silently dropped. |

**Status values** — order: `NEW`, `RDY`, `SENT`, `FAILED`. delivery attempt: `PENDING`, `OK`, `FAILED`.

---
*Illustrative glossary. A DOMAIN.md earns its place here because HDR / DTL / RDY are non-obvious; skip the file entirely when your domain is generic CRUD.*
