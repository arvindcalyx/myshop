# S1 Design Record — handover to Claude Code

**Date:** 2026-10-21 · **Owner:** Arvind · **Produced in:** a claude.ai chat (design work only; the operating model was **not** run there).
**Purpose:** everything Claude Code needs to close **S1 (Design)** under the ai-operating-model engine: what was produced, what was decided, what is still open, and drafts of the S1 wrap outputs for owner approval.

> **Limitations — read first.**
> - S1 is a **design and documentation milestone only**. No application code exists.
> - Nothing has been implemented or tested. The worked examples (spec §10.3) and tests T1–T7 (spec §10.6) are **test requirements for S10**, not passed tests. Their arithmetic was checked against a throwaway model during design; that says nothing about any implementation.
> - Tax, legal and supplier-term compliance is **not established**. Launch blockers D1, D10, D13, D16, D17 are unresolved.
> - The independent reviews below were of documents, not code.
> - The drafts in §6–§8 are **proposals awaiting owner approval**; they are not yet durable state.

This file is in `docs/archive/` (append-only, outside the ≈8 active-doc target). Do not edit it; supersede it with the S1 closed handoff.

---

## 1. Documents produced (on `main` and `design/s1-mvp-spec`)

| File | Version | Role in the operating model |
| --- | --- | --- |
| `docs/Platform_Design_v1.md` | v1.4 | Canonical design doc for the whole MVP until modules earn their own |
| `docs/DOCTRINE.md` | v0.1 draft | Doctrine — **awaiting independent cold-read validation and owner lock** |
| `docs/CONSTRAINTS_CARD.md` | matches doctrine v0.1 | Always-loaded constraint surface |
| `docs/archive/S1_Design_Record.md` | — | This handover record |

Not yet present (to be installed by Claude Code from the engine templates): `CLAUDE.md`, `docs/operating-model.md`, `docs/ACTIVE_ARCHITECTURE.md`, `docs/ACTIVE_RUNTIME_MAP.md`, `docs/DECISIONS_LOG.md`, `docs/WATCHPOINTS.md`, the active handoff, `.claude/skills/`.

## 2. Commits (branch `design/s1-mvp-spec`, fast-forwarded to `main`)

| Commit | Content |
| --- | --- |
| `67e6f83` | Initial repo commit by owner (`test` file) |
| `49b8aa1` | Spec v1.0 |
| `c096573` | Spec v1.1 — configurable business/tax settings, commercial rules, full lifecycle, ledger, capacity sizing |
| `0b629ed` | Spec v1.2 — second-review fixes |
| `20a2132` | Spec v1.3 — financial and lifecycle consistency |
| `0d45232` | Spec v1.3.1 — reversal idempotency, split refund limits, tests T1–T7 |
| `f4be7a5` | `DOCTRINE.md` v0.1 and `CONSTRAINTS_CARD.md` |
| `8d7bb30` | I-7 wording aligned across spec, doctrine and card |
| (commit adding this file) | Spec v1.4 owner decisions; card wording; this record — see `git log` |

## 3. Review and decision history

| Round | Reviewer | Main findings | Outcome |
| --- | --- | --- | --- |
| v1.0 | Independent (ChatGPT) | Seller of record; payment lifecycle; no-supplier-accepts path; DB-level invariants; GST lifecycle | Fixed in v1.1. Rejected with reasons: "too many modules" (folders, not frameworks), "too much marketplace design" (deferred table only), "too many payment paths" (COD expected; D5 decides) |
| v1.1 + owner requirements | Owner | Configurable tax/business settings; commercial rules; full delivery/returns lifecycle; 100k requests/day capacity; four-way scope split | Incorporated in v1.1 |
| v1.1 | Independent | Ledger sign convention; supplier failure after shipment; immutability vs adjustments; invoice timing; load-test profile; supplier privacy vs delivery contact; return window; live gate as a flag | Fixed in v1.2 |
| v1.2 | Independent | Supplier-driver COD netting; reversal vs refund; replacement quantities; two reconciliation checks; close guard | Fixed in v1.3. Advisor also found and fixed: abandoned prepaid checkouts left a balance (charges now written at Placed) |
| v1.3 | Owner | Idempotent reversal on replacement failure; separate refund limits | Fixed in v1.3.1; advisor added charge reinstatement (flagged) because the success path otherwise gave goods free |
| v1.3.1 | Owner | I-7 wording | Aligned in v1.3.2 |
| v1.3.2 | Owner | Stack; S2 split; ADR-004; card wording; live-gate scope | Done in v1.4 (this commit) |

## 4. Decisions taken by the owner in S1

| Decision | Record |
| --- | --- |
| Product / Offer / FulfilmentAssignment / Shipment model | Spec §5, §9.3a — ADR-002 (draft §6) |
| Modular monolith, web + worker, one Postgres, Postgres-backed queue; no Redis or microservices | Spec §3 — ADR-003 (draft §6) |
| Append-only multi-account ledger with defined transaction boundaries | Spec §10, §10.7 — ADR-004 (draft §6) |
| **D7a stack:** Next.js App Router + TypeScript; Drizzle ORM with hand-written SQL migrations for triggers and DB permissions; Graphile Worker on Postgres, connected directly or via a session-mode pooler (not transaction-mode) | Spec §3.2, §14 |
| **Live gate is global:** any unverified checklist item L1–L10 blocks all live placement and all live invoicing | Spec §4.1, Doctrine §5, Card |
| I-7: live transactions need verified business configuration; live tax invoices need that **and** an approved invoice policy; test-mode documents are never tax invoices | Spec §2, Doctrine §2, Card |
| Reseller flow modelled as the **working assumption only** (D1 still open) | Spec §4.1 |
| S2 split into S2a and S2b, separate PRs, no business features or live credentials | Spec §15 |
| Placeholder values (confirmation deadline 4 working hours, offer freshness 7 days) are config defaults, never constants | Spec §4.2, Doctrine C5 |

## 5. Open decisions

**Launch blockers — LIVE mode blocked until resolved (Doctrine §5):**

| ID | Decision | Needs | Checklist |
| --- | --- | --- | --- |
| D1 | Seller of record, GST, invoicing and accounting | CA | L1, L3 |
| D13 | Invoice policy; GST on delivery charges and fees; credit-note rules incl. charge reinstatement; documents travelling with goods | CA | L4 |
| D16 | Privacy notice, retention, consent | Lawyer | L7 |
| D10 | Return/refund terms incl. delivery-charge treatment | Owner + lawyer | L7 |
| D17 | Supplier terms incl. liability, payment, COD handover/netting, paperwork | Owner + lawyer | L9 |

**Other open decisions (owner):** D2 brand/domain · D3 who delivers · D4 delivery charges and minimums (L8) · D5 COD policy (L8) · D6 launch zones (L8) · **D7b** hosting vendor and SMS/WhatsApp provider · D8 launch SKUs · D9 platform fees · D11 confirmation deadline (placeholder 4 h) · D12 offer freshness (placeholder 7 days) · D14 COD refund method · D15 delivery OTP mandatory. Full table: spec §14.

None of these blocks S2a or S2b (test-mode, no business features).

---

## 6. Draft ADRs for `docs/DECISIONS_LOG.md` (owner approval required)

### ADR-001 — Run this project under the ai-operating-model engine v1.0.0

- **Date:** S1 close
- **Status:** Proposed → Accepted on owner validation of the S1 wrap
- **Design source:** `docs/operating-model.md` (engine v1.0.0), `docs/DOCTRINE.md` v0.1.
- **Decision:**
  1. The project runs under ai-operating-model `MODEL_VERSION` 1.0.0, pinned in `CLAUDE.md`.
  2. Authority chain: `DOCTRINE` > `operating-model.md` > `CLAUDE.md`. A design conflicting with doctrine is wrong.
  3. Single gated writer: only `session-wrap` writes durable state, after human merge and validation (design-only sessions wrap at session end).
  4. Implementation reaching an unanswered design question stops for an explicit decision (Doctrine C11).
- **Rationale:** The platform handles money, tax documents and supplier commitments; drift through small inline decisions is the main risk. One ADR because these commitments only work together.
- **Consequences:** Sessions start from the active handoff ENTRY + `CONSTRAINTS_CARD.md`; code sessions STOP at Engineering Complete; High-tier paths get blind + hostile review. Engine `skills/` are never edited in this repo.
- **Supersedes / superseded by:** nothing / nothing.
- **References:** ai-operating-model `README.md`, `docs/DESIGN.md`, `docs/getting-started.md`.

### ADR-002 — Separate what is sold from who supplies it: Product / Offer / FulfilmentAssignment / Shipment

- **Date:** S1 close
- **Status:** Proposed
- **Design source:** `docs/Platform_Design_v1.md` v1.4 §5, §9.3–9.3a, §13.
- **Decision:**
  1. Product is the platform's catalogue item and never references a supplier.
  2. Offer is one supplier's ability to supply one product.
  3. FulfilmentAssignment links one order line to one source; at most one active per line; `source_type` extensible (v1 `SUPPLIER_SOURCED`; later `OWN_STOCK`).
  4. Shipment is the unit of delivery (one source, address, slot); its source never changes; supplier failure cancels and supersedes, never edits.
  5. Only `fulfilment` and `delivery` see suppliers; customer surfaces never do (I-1, C1).
- **Rationale:** Rejected (a) supplier-specific listings and (b) `supplier_id` on Product: both put the supplier on the customer's path and make a supplier change a catalogue change. This framing keeps the customer contract stable while sourcing changes underneath, so own stock, supplier portals and delivery partners arrive as new source or executor types.
- **Consequences:** I-4/C9 DB constraints become load-bearing. Line splitting across suppliers deferred. Marketplace seller-of-record is not reachable by configuration (spec §13).
- **Supersedes / superseded by:** nothing / nothing.
- **References:** spec §2, §5, §9.3a, §13; Doctrine I-1, I-4, C1, C9.

### ADR-003 — Modular monolith: web + worker processes, one Postgres, Postgres-backed job queue

- **Date:** S1 close
- **Status:** Proposed
- **Design source:** `docs/Platform_Design_v1.md` v1.4 §3 (incl. §3.4 thresholds, §3.5 module boundaries, §3.7 load test).
- **Decision:**
  1. One codebase of modules that own their tables and expose typed internal APIs.
  2. Two process types: stateless web (horizontal, minimum 2 instances) and worker (Graphile Worker: queue, schedules, webhook processing).
  3. One managed Postgres; no Redis, replicas or separate services until a §3.4 threshold is met and recorded in an ADR (C12).
  4. Serverless-only hosting ruled out (worker needs a long-running process).
- **Rationale:** 100,000 requests/day ≈ 2 req/s average (designed for 50 req/s) is well within one monolith and database; the ceiling is manual operations. Microservices would add cost and make transactional rules (placement, ledger writes with transitions) harder to keep atomic. The framing defended is "scale by instances, split only on measured evidence".
- **Consequences:** Module boundaries enforced in code (lint, no cross-module table access). Not called proven until the §3.7 load test passes (S12). The approved stack (D7a: Next.js, Drizzle, Graphile Worker) is a tooling choice recorded in the spec and handoff, not in this ADR; hosting and messaging vendors remain D7b.
- **Supersedes / superseded by:** nothing / nothing.
- **References:** spec §3; Doctrine C3, C12.

### ADR-004 — Append-only multi-account financial ledger with defined transaction boundaries

- **Date:** S1 close
- **Status:** Proposed
- **Design source:** `docs/Platform_Design_v1.md` v1.4 **§10.7** (transaction boundaries and idempotency), with §10.1–§10.6 (record layers, accounts and sign rule, invariants, worked examples, invoices and reconciliation, S10 tests).
- **Decision:**
  1. Every money movement is an insert-only ledger entry on one counterparty account (customer per order, gateway, executor, supplier), with one sign rule: + they owe the business, − the business owes them.
  2. The customer account is the authoritative customer balance; tax documents are reconciled against it, not the reverse.
  3. Operational changes that alter what is owed write their ledger entries in the same transaction, along the ten boundaries TX-1…TX-10 in §10.7; no external calls inside a transaction; jobs enqueued transactionally.
  4. Charge reversals and refunds are distinct events; reversals carry unique source references and never exceed the unreversed charge; refunds must satisfy both the owed limit and the paid limit, net of completed refunds and pending commitments.
  5. Two reconciliation checks (tax documents; customer account) are part of the order close guard.
- **Rationale:** Orders change after placement (partial deliveries, returns, replacements, refusals, late payments) while placed orders and issued documents must stay immutable. A single-balance or mutable-order model would force edits to history to stay correct. Recording this as one ADR because the sign rule, transaction boundaries and limits only guarantee correctness together. The detail lives in §10; this ADR records acceptance of that framing.
- **Consequences:** DB role permissions and triggers enforce insert-only tables (S2b). S7 (payments) and S10 (returns, refunds, documents) must implement §10.7 exactly; §10.3 examples and §10.6 T1–T7 become automated tests in S10. No payments or refunds are implemented in S2a/S2b. Invoice timing remains D13.
- **Supersedes / superseded by:** nothing / nothing.
- **References:** spec §2 (I-3, I-8, I-9), §10; Doctrine C2, C4, C7, C8.

---

## 7. Draft `CLAUDE.md` (repo root; owner approval required)

```markdown
# MyShop (working name) — Session Bootstrap

operating-model: v1.0.0

This file **POINTS; it does not RESTATE.** The operating model lives in `docs/` and the engine in
`.claude/skills/`. This file is in context every turn, so it stays minimal.

## Start every session
1. State one line: `Initiating S<N> <Intent>`  (Intent ∈ Bug | Investigation | Design | Implementation).
2. Run the `session-start` skill. It loads the Tier-0 set:
   - **Always:** the single active `docs/HANDOFF_Session_<NNN>_Active.md` **ENTRY** + `docs/CONSTRAINTS_CARD.md`.
   - **+ intent index:** `docs/ACTIVE_RUNTIME_MAP.md` and/or `docs/ACTIVE_ARCHITECTURE.md` (see the skill).
   - Read each whole file — keep them small.
3. **Startup safeguard:** check for an open PR from the previous session. If one exists, surface it and
   ask whether we're resuming it (validate → wrap) or starting new work.
4. Emit the orientation line and wait for direction.

Pull everything else only on its trigger during the session: the full `docs/DOCTRINE.md`, the canonical
design doc `docs/Platform_Design_v1.md` (by section), `docs/WATCHPOINTS.md`, `docs/DECISIONS_LOG.md`.

**Authority:** `DOCTRINE` > `docs/operating-model.md` > this pointer.
**Single Gated Writer:** only `session-wrap` writes durable state, and only after merge + human validation.
**Unanswered design question → STOP** for an explicit decision (Doctrine C11).
```

---

## 8. Draft S2a handoff — `docs/HANDOFF_Session_002a_Active.md` (owner approval required)

Only one active handoff may exist. S2b's handoff is created at the S2a wrap; S2b's scope is in spec §15.

```markdown
# MyShop — Active Handoff (S2a)

**Purpose.** Operational truth the next session reads on entry. ENTRY at start; HISTORY only on a
"what happened / why" trigger.

**Wrap invariant.** Exactly one active handoff at `docs/` root. At S2a close, archive verbatim →
`docs/archive/Handoff_Session2a_Closed.md` and create `HANDOFF_Session_002b_Active.md`.

---

# ENTRY

## Standing platform facts
- Repo `arvindcalyx/myshop`. No application code before S2a.
- Canonical design doc: `docs/Platform_Design_v1.md` v1.4. Doctrine v0.1 (draft until validated). Always-loaded: `docs/CONSTRAINTS_CARD.md`.
- Architecture: modular monolith, web + worker, one Postgres (ADR-003). Domain core per ADR-002. Ledger per ADR-004.
- Stack (D7a, approved): Next.js App Router + TypeScript; Drizzle ORM (hand-written SQL migrations for triggers and DB permissions); Graphile Worker on Postgres — worker connects directly or via a session-mode pooler, never transaction-mode.
- Hosting vendor and messaging provider not chosen (D7b). No deployment target is required for S2a.
- System runs in TEST mode only. No live credentials in any environment. Live gate is global.

## Next-session scope — S2a Implementation (spec §15)
- Next.js App Router + TypeScript scaffold; module folders per spec §3.5; lint rules enforcing module boundaries (C1: storefront modules cannot import `fulfilment`).
- Operating-model installation already done at S1 close (verify present; do not re-install).
- Web entry point and worker entry point (Graphile Worker) with health checks for both.
- Structured JSON logging with request id; error-monitoring hook (provider-agnostic interface).
- CI: lint, type-check, unit tests; a test proving the boundary lint rule fails on a forbidden import.
- Graphile Worker may install its own schema; **no application tables** (that is S2b).
- Risk tier: Medium (no persistence, money or auth paths). Independent verification + blind and hostile review before merge.

## Anti-scope
- No application database schema or migrations (S2b).
- No business features: catalogue, cart, checkout, payments, refunds, admin, notifications.
- No live credentials, no production deployment, no Redis, replicas or extra services (C12).
- Do not reopen ADR-001–004, invariants I-1–I-9, refund limits, invoice-policy options, live-gate scope, or any D-decision.

## Active carry-forward ledger
- Launch blockers open: D1, D13 (CA); D16 (lawyer); D10, D17 (owner + lawyer). Do not block test-mode build.
- Other open decisions: D2–D6, D7b, D8, D9, D11, D12, D14, D15 (spec §14).
- Doctrine v0.1 awaiting independent cold-read validation before lock.
- Spec §10.3 examples and §10.6 T1–T7 are S10 test requirements, not passed tests.
- **Deferred design seeds:** watchpoint candidates — ledger bypass via convenience writes; typed config drifting into a generic rules engine; supplier fields leaking through shared DTOs; status changes bypassing transition functions. Live-tracking pings to a separate ingest store (spec §9.6).

## Known placeholders
- D11 confirmation deadline 4 working hours; D12 offer freshness 7 days (config defaults, C5).
- D4 delivery charges and minimums: example values only. Gateway fee 2%: examples only.
- "MyShop" is a working name until D2.

## OS backlog
- None.

## Doctrine-discipline check
- Active core docs: 8 (DOCTRINE, operating-model, CONSTRAINTS_CARD, ACTIVE_ARCHITECTURE, ACTIVE_RUNTIME_MAP, DECISIONS_LOG, HANDOFF, Platform_Design_v1). ADRs: 4 (target ≈25).
- Validation: spec reviewed independently three times and accepted by the owner; doctrine not yet independently validated; no implementation exists.

---

# HISTORY
- See `docs/archive/Handoff_Session1_Closed.md` and `docs/archive/S1_Design_Record.md`.
```

---

## 9. Steps for Claude Code to close S1

1. Read the engine: `https://github.com/arvindcalyx/ai-operating-model` — `README.md`, `docs/getting-started.md`, `docs/operating-model.md`, `skills/`, `templates/`.
2. Install per `getting-started.md`, **without overwriting** the filled `docs/DOCTRINE.md` and `docs/CONSTRAINTS_CARD.md`: copy the remaining templates into `docs/`; create `CLAUDE.md` from §7 (after owner approval); copy `skills/` into `.claude/skills/`; leave `ACTIVE_RUNTIME_MAP.md` and `WATCHPOINTS.md` as stubs; add one `ACTIVE_ARCHITECTURE.md` row per spec §3.5 module, each pointing at `docs/Platform_Design_v1.md`.
3. Get owner approval of the §6 ADRs, §7 `CLAUDE.md` and §8 handoff. Do not mark doctrine as locked unless the owner confirms independent validation is done.
4. Run `session-wrap` for S1 (design session: no merge gate): write `DECISIONS_LOG.md` with ADR-001–004; fill the S1 handoff from this record and archive it to `docs/archive/Handoff_Session1_Closed.md`; create the single active `HANDOFF_Session_002a_Active.md`; verify exactly one active handoff.
5. Do not change any decision in spec v1.4 or doctrine v0.1 during the wrap. Any conflict found → stop and ask the owner.
6. Do not start S2a in the same session.

**Sequencing note for the owner:** spec §15 lists "operating-model installation" under S2a, but `session-wrap` must exist to close S1, so step 2 installs the engine at S1 close. The §8 handoff therefore has S2a *verify* the installation rather than perform it. If you prefer to keep installation in S2a, S1 must be closed manually (write the decisions log and handoff without the skill) — confirm which before step 2.
