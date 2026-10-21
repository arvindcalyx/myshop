# MyShop — Constraints Card

**Always loaded. Defers to `DOCTRINE.md` v0.1 (draft).** States rules, not proof: no implementation exists or has been tested against these yet, and launch compliance is not established (see blockers below).

## Constraints (Doctrine §3)

| | Constraint | Where it bites |
| --- | --- | --- |
| **C1** | No supplier data on any customer surface; storefront never imports `fulfilment` | `catalog`, `checkout`, `notify` |
| **C2** | All money through the `finance` ledger; integer paise | `finance` |
| **C3** | State changes only via transition functions + event record | all stateful modules |
| **C4** | Order snapshots, invoices, credit notes write-once | DB triggers, `finance` |
| **C5** | No hardcoded prices, fees, thresholds, zones, timers — versioned config | `config`, `pricing` |
| **C6** | Live gate re-checked at placement and invoicing; edits revoke checklist items | `config`, `checkout`, `finance` |
| **C7** | Payment state only from verified idempotent webhook or recorded COD | `finance` |
| **C8** | Reversal ≤ unreversed charge, unique source ref; refund ≤ owed limit AND ≤ paid limit | `finance`, `returns` |
| **C9** | One active assignment per line; shipment source fixed; original/replacement qty checked separately | DB constraints |
| **C10** | Delivered only with OTP, or proof + Owner override | `delivery` |
| **C11** | Unanswered design question → STOP for a decision | every session |
| **C12** | No new infra or service split without spec §3.4 evidence + ADR | architecture |

## Invariants (Doctrine §2)

| | Invariant |
| --- | --- |
| **I-1** | Supplier identity/commercial data never customer-facing |
| **I-2** | Price, cost, fees, delivery charge, delivery cost kept separate |
| **I-3** | Placed snapshots and issued documents immutable |
| **I-4** | ≤1 active assignment per line; all transitions logged |
| **I-5** | Integer paise; integer quantities |
| **I-6** | Versioned config; orders record versions used |
| **I-7** | No live transaction or invoice without verified config + approved invoice policy |
| **I-8** | Append-only ledger; close only at zero balance with nothing unresolved |
| **I-9** | Payment status only from verified event or recorded COD |

## Launch blockers — LIVE mode blocked until resolved (Doctrine §5)

D1 seller of record/GST (CA) · D13 invoice policy and tax treatment (CA) · D16 privacy/retention (lawyer) · D10 return/refund terms (lawyer) · D17 supplier terms (lawyer). Test-mode build may proceed.

---

**If a change appears to violate any C or I line, stop** — that is a doctrine-level question, not an inline decision. Open the full `DOCTRINE.md` §2/§3 and route per the validation chain.
