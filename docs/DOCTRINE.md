# MyShop (working name, brand = D2) — Product Doctrine

**Version:** v0.1 draft · 2026-10-21 · Owner: Arvind
**Status:** Draft pending independent validation and owner lock. Derived from `docs/Platform_Design_v1.md` v1.3.1 (§2).

**What this is.** The constitution. What the product **is / is not / refuses to do**, the non-negotiable invariants and constraints, and the success/failure criteria. Everything defers to it. If a design or a change conflicts with a line here, **the design is wrong, not the doctrine.** ADRs never override it.

**What this is not.** This document states rules the implementation must satisfy. It is **not** evidence that any implementation satisfies them — no code exists at this version — and it is **not** evidence of tax, legal or supplier-term compliance. Compliance depends on the launch blockers in §5, which are unresolved.

---

## 1. What the product is / is not

- **Is:** a branded, single-seller online store for construction materials in small quantities, starting in Gurgaon. The business sets every price and fee, sources each order line from a local supplier it assigns, and is accountable to the customer for delivery, returns and refunds.
- **Is not (v1):** a marketplace; a supplier-facing platform; an inventory or warehouse system; a logistics platform; a credit or financing product; a native mobile app.
- **Refuses to:**
  - show supplier identity or commercial data to customers;
  - change prices automatically;
  - accept orders outside serviced zones;
  - take live orders without a verified business configuration;
  - issue live tax invoices without both a verified business configuration and an approved invoice policy;
  - treat any test-mode document as a tax invoice;
  - mark goods delivered without a delivery OTP or recorded proof with an Owner override;
  - move money, or record money as moved, outside the ledger;
  - treat a design document, review or test-mode run as proof of compliance.

## 2. Invariants (tersest binding form)

| | Invariant |
| --- | --- |
| **I-1** | Supplier identity and commercial data never appear on a customer-facing surface; customers see only the delivery contact needed to receive goods. |
| **I-2** | Goods price, supplier cost, platform fees, delivery charge and delivery cost are separate financial components with separate owners. |
| **I-3** | A placed order's snapshot and every issued financial document are immutable; corrections are new records. |
| **I-4** | An order line has at most one active fulfilment assignment; a shipment line always references its assignment; a shipment's source never changes; every state change goes through a transition function and is logged with actor, time and reason. |
| **I-5** | Money is integer paise; quantities are positive integers in the product's selling unit. |
| **I-6** | Business rules are versioned configuration; an order records the rule versions and amounts it used. |
| **I-7** | Live transactions require a verified business configuration. Live tax-invoice issuance requires a verified business configuration **and** an approved invoice policy; invoice-policy approval never substitutes for business verification. Test-mode documents are never tax invoices. |
| **I-8** | Every money movement is an append-only ledger entry; the ledger customer account is the authoritative balance; an order closes only when it is zero and no return, replacement or refund is unresolved. |
| **I-9** | Payment status changes only from a verified, idempotent gateway event or a recorded COD collection. |

## 3. Constraints (where the invariants bite)

Module names refer to the planned modules in `Platform_Design_v1.md` §3.5. No code exists yet; "where it bites" names where enforcement must live.

| | Constraint | Where it bites |
| --- | --- | --- |
| **C1** | Storefront modules never import `fulfilment`; customer DTOs and notification templates carry no supplier fields. (I-1) | `catalog`, `checkout`, `notify`; lint rule; storefront contract test |
| **C2** | Every money movement goes through `finance` as a ledger entry; amounts are integer paise. (I-2, I-5, I-8) | `finance`; DB column types; insert-only ledger table |
| **C3** | State changes only through each entity's transition function, in one transaction with its event record; no direct status writes. (I-4) | `orders`, `fulfilment`, `delivery`, `returns`, `finance`; DB role permissions |
| **C4** | Order snapshots, invoices and credit notes are write-once; corrections are reversals, adjustments, credit notes or refunds. (I-3) | DB triggers; `finance` |
| **C5** | No business value is hardcoded: prices, costs, fees, thresholds, zones, slots and timers live in versioned config. Placeholder values (e.g. confirmation deadline, offer freshness) are config defaults, never constants. (I-6) | `config`, `pricing`; code review |
| **C6** | The live-mode gate is re-checked at execution time: live placement checks verified business configuration; live invoice issuance checks verified business configuration **and** an approved invoice policy; test-mode documents use a test series and can never become tax invoices; editing a gated setting revokes its checklist item. (I-7) | `config`, `checkout`, `finance` |
| **C7** | Payment state changes only via a signature-verified, idempotent webhook or a recorded COD collection; never from a browser redirect. (I-9) | `finance` webhook handler; unique gateway event id |
| **C8** | Reversals carry a unique source reference and never exceed the unreversed charge; refunds never exceed the owed limit or the paid limit, each net of completed refunds and pending commitments. (I-8) | `finance`, `returns`; order lock; unique constraints |
| **C9** | One active assignment per line; a shipment line's assignment source equals the shipment's source; original and replacement quantities are checked separately. (I-4) | DB partial unique index, FK and check constraints |
| **C10** | "Delivered" only from Out for delivery, with a delivery OTP or proof plus an Owner override. | `delivery` |
| **C11** | Implementation that reaches a question the spec does not answer **stops** for an explicit decision; it does not decide inline. | Every implementation session (operating model §1) |
| **C12** | No new infrastructure or service split (Redis, replicas, queues, microservices) without the threshold evidence in spec §3.4 and an ADR. | Architecture changes |

## 4. Success / failure criteria

Checkable statements; each should be verifiable from the database, logs or tests months after launch.

**Success means all of these stay true:**
- No storefront response, notification or customer document contained supplier identity or commercial data.
- Every Closed order has customer balance 0 and passes both reconciliation checks (tax documents; customer account).
- No ledger entry, invoice or credit note was ever updated or deleted.
- No payment changed state without a verified gateway event or a recorded COD collection.
- No live order was placed while the business profile was not verified.
- No live tax invoice was issued without both a verified business profile and an approved invoice policy.
- No test-mode document was ever issued or numbered as a tax invoice.
- No order line ever had two active assignments; no shipment ever changed source.
- No refund exceeded either limit; no system reversal was written twice for the same source.
- No shipment reached Delivered without an OTP or recorded proof and override.
- No business value used in pricing or timers came from a code constant.
- Every infrastructure addition has a linked ADR with threshold evidence.

**Failure signals (each triggers a doctrine-level review, not a local fix):**
- A reconciliation mismatch that cannot be traced to a specific record.
- A fix that requires editing a ledger row or an issued document.
- A feature request that needs supplier data on the storefront.
- A "temporary" hardcoded price, fee or threshold.

## 5. Launch blockers (unresolved)

These decisions are **not made**. They do not block building and testing in `TEST` mode; each **blocks switching to `LIVE` mode** through the go-live checklist (spec §4.1). The gate is **global**: any unverified checklist item blocks all live order placement and all live invoice issuance. Nothing in this doctrine, the spec or any review resolves them.

| Blocker | Decision | Requires | Blocks (checklist item) |
| --- | --- | --- | --- |
| **D1** | Seller of record, GST treatment, invoicing and accounting model (working assumption: reseller under the business GSTIN) | **Chartered Accountant** | L1, L3 |
| **D13** | Invoice policy (at placement / dispatch / delivery); GST on delivery charges and platform fees; credit-note rules, including charge reinstatement; documents that travel with goods (delivery challan / e-way bill where applicable) | **Chartered Accountant** | L4 |
| **D16** | Privacy notice, data retention, consent text (including any future location tracking) | **Lawyer** | L7 |
| **D10** | Return and refund policy terms: window, change-of-mind, redelivery and restocking charges, delivery-charge treatment on partial delivery or refusal | Owner + **lawyer** for customer terms | L7 |
| **D17** | Supplier terms: liability for damaged or incorrect goods, payment terms, return-to-supplier costs, COD handover and netting, no supplier paperwork or branding to customers | Owner + **lawyer** | L9 |

Commercial values the Owner must also set before live (not legal or tax decisions, but still live-mode prerequisites): D4 delivery charges and minimums, D5 COD policy, D6 launch zones (checklist L8).

---

**Overrides.** If a change must violate a C or I line, that is a doctrine-level decision, not an inline call. Stop, route it through the validation chain, and record the override with its justification in `DECISIONS_LOG.md`. Silent drift across many small exceptions is the failure mode this file prevents.
