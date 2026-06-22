# GRQ Solar — Development & Token Strategy Document

**Project:** Tokyo branch of an overseas solar-panel sales company
**Product:** Online platform where investors buy solar panels (USD 500 each) and
receive returns generated from the electricity produced by the underlying solar
projects. Panels are payable in fiat or in GRQ tokens / GRQ Credits.
**Document version:** 2.0 · 2026-06-22 · *(currency = USD; panel = $500 fixed)*
**Status:** Strategy draft (for internal review)

---

## 1. Executive Summary

GRQ Solar operates an **investment platform** for solar power. An investor buys
one or more solar panels at a **fixed price of USD 500 per panel** and receives a
**return paid from the revenue** of the underlying solar project (electricity
sales). The detailed project economics — hardware cost, installation, land, grid
connection, development and financing — **sit behind the platform and vary by
project and country**, and are *not* fixed at this stage. For system design we
use the working assumption:

- **Investor purchase price: USD 500 / panel**
- **Returns paid from project revenues**
- **Cost & profit margins vary by project; not yet fixed**

On top of this sits a **loyalty-token reward loop (GRQ)**:

1. Every **fiat** panel purchase pays the buyer a **GRQ bonus** (a % of the
   purchase value, converted at the *current* GRQ price).
2. The GRQ price **rises monotonically** as more GRQ is distributed.
3. So **early investors accumulate cheap GRQ that gains purchasing power over
   time**, spendable only on *more panels*.
4. A **referral program** pays inviters 1.5–2.5 % of an invitee's fiat purchase
   in GRQ, driving organic growth.

GRQ is a **closed-loop store of panel-purchasing-power, not a cash-out
instrument**: it can never be redeemed for cash or swapped for other currencies
*through the system*.

> ⚠️ **Two distinct things to keep separate, especially for compliance (§9):**
> 1. **The panel-with-returns offering** — this is an *investment product*. In
>    most jurisdictions (incl. Japan) selling an asset that pays returns from a
>    pooled/managed project is a **regulated securities / collective-investment
>    offering**. This is the primary regulatory item and exists independently of
>    GRQ.
> 2. **GRQ** — a loyalty/bonus layer. Keep it framed as *discount purchasing
>    power for panels*, never as a tradeable asset, to avoid adding a second
>    regulated instrument on top of the first.

---

## 2. Goals & Non-Goals

### Goals
- Launch a compliant online platform to sell $500 solar panels to investors.
- Track each panel's link to an underlying **project** and pay **returns from
  project revenue**.
- Dual payment rails: fiat and GRQ/GRQ Credits.
- Deterministic, auditable GRQ pricing engine + bonus/referral issuance.
- Make the on-chain/off-chain distinction invisible to non-crypto users.

### Non-Goals (v1)
- No fixed cost-per-watt or fixed profit-per-panel model (project-specific
  economics come **later**).
- No fiat *cash-out* or swap of GRQ. GRQ is spend-only on panels.
- No exchange listing / AMM / GRQ↔other-token swaps; no secondary marketplace
  operated by GRQ Solar.

---

## 3. System Architecture (high level)

```
                         ┌──────────────────────────────────────────┐
                         │                Web Frontend               │
                         │  (Next.js · wallet connect · checkout)     │
                         └───────────────┬────────────────────────────┘
                                         │ REST / GraphQL
                         ┌───────────────▼────────────────────────────┐
                         │            Application Backend              │
                         │ Auth · Orders · Projects · Pricing · Ledger │
                         │ Returns/Payout engine                       │
                         └──┬──────────┬──────────┬──────────┬─────────┘
                            │          │          │          │
              ┌─────────────▼┐ ┌───────▼────┐ ┌───▼──────┐ ┌─▼──────────────┐
              │ Fiat Payments│ │ Off-chain  │ │ On-chain │ │ Project Registry│
              │ Stripe/CC/   │ │ Ledger     │ │ Solana   │ │ + Returns ledger│
              │ PayPal       │ │ (GRQ Credit│ │ (GRQ SPL,│ │ (revenue in →   │
              │              │ │  balances) │ │  wallet) │ │  payouts out)   │
              └──────────────┘ └─────┬──────┘ └────┬─────┘ └─────────────────┘
                                     └──────┬──────┘
                                      1:1 Credit→Token bridge
```

### Core components
| Component | Responsibility |
|---|---|
| **Auth & Accounts** | Sign-up, KYC tier, wallet linking (Phantom / Connect). |
| **Catalog & Orders** | Panels @ $500, quantities, order lifecycle. |
| **Project Registry** | Each project's metadata; maps owned panels → a project. |
| **Returns / Payout engine** | Records project revenue in, computes & pays investor returns out (see §6). |
| **Payments (fiat)** | Stripe, credit card, PayPal; idempotent webhooks. |
| **Payments (token)** | Verify inbound GRQ to company wallet; debit GRQ Credits. |
| **Pricing Engine** | Single source of truth for current GRQ price (§5). |
| **Issuance & Ledger** | Mint/allocate bonus & referral GRQ; double-entry ledger. |
| **Custody Bridge** | On-chain wallet ops + off-chain Credits + 1:1 conversion. |
| **Admin / Treasury** | Parameter config, reserve & payout monitoring, reporting. |

---

## 4. Token Model: GRQ Token vs GRQ Credit

| | **GRQ Token** | **GRQ Credit** |
|---|---|---|
| Form | On-chain SPL token (Solana) | Off-chain balance in system DB |
| For whom | Users with wallet knowledge | Users without wallet knowledge |
| Custody | User's own wallet (Phantom) | Custodied by GRQ Solar |
| Spend on panels | ✅ | ✅ |
| Transfer | Wallet→wallet (on-chain) | Account→account (small fee `f`) |
| Convertible | — | **GRQ Credit → GRQ Token, 1:1, user-initiated** |
| Cash-out | ❌ never | ❌ never |

**Key rule:** a purchase paid with GRQ **or** GRQ Credit earns **no** new GRQ
bonus. Bonuses are only minted against **fiat** purchases — this prevents a
self-referential inflation loop.

The 1:1 Credit→Token conversion lets a user who later "graduates" to a wallet
pull their off-chain balance on-chain. The price formula is identical for both,
so conversion is value-neutral.

---

## 5. GRQ Pricing Formula

### 5.1 Design requirements
- **Deterministic & auditable** — recomputable from public state.
- **Monotonically non-decreasing** — price only rises with distribution.
- **Bounded growth, no blow-up** — avoid a `1/(1-r)` singularity.
- **Well-defined past full distribution** — must keep working when cumulative
  distribution exceeds the initial issuance constant.

### 5.2 Parameters (constants)

| Symbol | Meaning | Working value |
|---|---|---|
| `P₀` | Initial GRQ price | **$0.005** |
| `N₀` | Initial issuance constant | **1,000,000,000 GRQ** |
| `α`  | Price-growth coefficient | **9** |
| `B`  | Bonus rate on fiat purchases | **5 %** |
| `R`  | Referral rate | **1.5 – 2.5 %** (default 2 %) |
| `f`  | GRQ-Credit transfer fee | **0.5 %** |
| — | Investor panel price | **$500 (fixed)** |

`D` = cumulative GRQ distributed to users (bonus + referral), the only
*variable* in the formula.

> **Calibration note.** The absolute GRQ start price is a free parameter; what
> matters economically is `P₀ · N₀` relative to revenue, which sets how fast the
> price moves. With `$500` panels and a 1 B supply, `P₀ = $0.005` makes the price
> rise meaningfully over a realistic early-stage horizon (≈10 % of supply by
> ~50k panels) and reach full distribution at ≈948k panels. If 1 B is later
> revised, re-pick `P₀` so `P₀·N₀ ≈ $5 M` to preserve this pacing.

### 5.3 The formula (recommended: linear bonding curve)

```
P(D) = P₀ · ( 1 + α · D / N₀ )          with r = D / N₀
```

**Why linear?**
- `D = 0` → `P = P₀ = $0.005`.
- `D = N₀` (full distribution, `r = 1`) → `P = P₀·(1+α) = $0.05` — a clean
  **10×** over the life of the initial issuance.
- Easiest curve to explain to investors and regulators.
- Never blows up: past full distribution (`r > 1`) it simply keeps rising along
  the same line — exactly the desired over-issuance behavior.

**Over-issuance.** When all of `N₀` is distributed, GRQ Solar may keep issuing.
Because `N₀` is a *constant in the denominator* (not live supply), `D` grows past
`N₀`, `r > 1`, and price keeps climbing. Example: `r = 1.5` → `P = $0.005·(1+9·1.5)
= $0.0725`.

### 5.4 Optional convex variant (later)

```
P(D) = P₀ · ( 1 + α · (D / N₀)^β ),   β > 1  (e.g. 1.3)
```

Steeper early-bird reward. **Recommendation: ship linear first**; treat `β` as a
governance option, never changed retroactively.

### 5.5 Price-update mechanics
- Recomputed on **every issuance event**; read at the moment of each transaction.
- A purchase uses the price **as of order confirmation** (price-lock ~10 min to
  avoid mid-payment drift).
- All changes written to an **append-only price-history log** for audit.

---

## 6. Flows

### 6.1 Earning GRQ
1. **Purchase bonus (fiat only).** Buyer pays `$X` by fiat → receives
   `(B · X) / P(D)` GRQ (on-chain if wallet linked, else GRQ Credit).
2. **Referral.** Invitee's link carries a referral ID. On the invitee's *fiat*
   purchase of `$X`, the inviter receives `(R · X) / P(D)` GRQ/Credit.

Both increase `D`, nudging the price up.

### 6.2 Spending GRQ on panels
- `GRQ_needed = 500 / P(D)` per panel.
- On-chain: user transfers GRQ Phantom → company wallet; backend verifies the
  tx signature & amount before fulfilling.
- Off-chain: system debits GRQ Credit balance.
- **No bonus** on GRQ/Credit-paid purchases.

### 6.3 Returns / payouts (the investment side)
- Each owned panel is linked to a **project** in the registry.
- Project revenue (electricity sales) flows in; the **Returns engine** records it
  and allocates **returns to panel owners** per that project's rules.
- **v1 working assumption:** returns are computed per project (rate not fixed)
  and **paid in fiat (USD)** to the investor's payout method. *Open question:*
  optionally allow payout as GRQ Credits or auto-reinvest into new panels.
- The returns ledger is **separate** from the GRQ ledger — different money, different
  accounting, different regulatory treatment.

### 6.4 Transfers
- **GRQ token → token:** on-chain transfer (network fee only).
- **GRQ Credit → Credit:** internal ledger move, **fee `f` = 0.5 %** from sender.

---

## 7. Custody, Wallets & the On/Off-chain Bridge

- **Wallet-savvy users** link Phantom/Connect; bonus GRQ sent on-chain.
- **Non-savvy users** receive **GRQ Credits** off-chain; an **omnibus company
  wallet** backs all outstanding Credits 1:1.
- **Graduation:** linking a wallet later lets a user convert Credits → Tokens
  1:1; the bridge mints/transfers on-chain and burns the off-chain balance
  atomically (saga + reconciliation).
- **Invariant (must always hold):**
  `on-chain GRQ held for users + outstanding GRQ Credits = total distributed D`.
  Nightly reconciliation asserts this and alerts on drift.

---

## 8. Treasury, Reserves & Returns Funding

Two separate liabilities to manage:

1. **GRQ loyalty liability.** GRQ is spend-only on panels, so every GRQ returns
   to GRQ Solar as panel payment — at a higher price than issued. The cost is the
   **5 % bonus**: a *prepaid, appreciating discount* on future panels. Book it as
   **deferred revenue / loyalty liability**. Hold a **panel-supply reserve** sized
   to the outstanding GRQ liability so holders can always actually buy panels.
2. **Returns liability.** Returns to investors are funded from **project
   revenues**, not from new panel sales. Funding returns out of new sales would be
   a red flag (see §9). Track per-project: revenue in vs returns paid out.

Monitor the **GRQ redemption ratio** (GRQ-paid sales ÷ total sales); if it climbs
too fast margins compress — tune `B`/`α` **prospectively**, never retroactively.

---

## 9. Regulatory & Compliance Strategy — **read before launch**

> Flags risk areas; **not legal advice.** Engage Japanese (and host-country)
> counsel before go-live. This platform now has **two** regulated dimensions.

### 9.1 The panel-with-returns offering (PRIMARY risk)
- Selling an asset that pays a **return from a pooled/managed project** is, in
  most jurisdictions, a **securities offering or collective investment scheme**.
  In Japan this likely touches the **Financial Instruments and Exchange Act
  (金融商品取引法)** and may require registration/licensing and disclosure.
- **Cross-border:** projects in other countries add each host country's
  securities, energy and tax regimes.
- **Do not** promise or imply guaranteed/fixed returns; returns must be presented
  as variable and project-dependent (which matches the stated model).
- **Fund returns only from genuine project revenue** — never from new investor
  inflows. This separation is both an accounting and a legal necessity.

### 9.2 GRQ (SECONDARY risk)
- **Payment Services Act (資金決済法).** A transferable on-chain token used for
  payment can be a "Crypto Asset," triggering **FSA** exchange-provider
  registration. GRQ Credits (off-chain, spend-only) look more like a lighter
  **prepaid payment instrument (前払式支払手段)**.
- **No cash-out, by design** — the single strongest mitigation; preserve it.
- Never market GRQ as an investment or guaranteed appreciation; frame it as
  loyalty purchasing power for panels.

### 9.3 Cross-cutting
- **AML/KYC:** tiered — light for browsing, stricter for investing, wallet
  linking and large GRQ transfers.
- **Consumer protection / 特定商取引法:** clear disclosure of pricing mechanics,
  bonus terms, returns variability, and that GRQ has no cash value.
- **Tax:** treatment of returns, bonuses, and revenue-recognition timing — with
  an accountant, per jurisdiction.

**Recommended posture:** (a) get securities/licensing clarity on the
panel-returns product *before* taking investor money; (b) launch GRQ as
**off-chain Credits first** (prepaid-instrument-like), with the on-chain token as
an opt-in advanced feature behind stricter KYC and a legal green-light.

---

## 10. Technology Stack (proposed)

| Layer | Choice | Rationale |
|---|---|---|
| Frontend | Next.js + TypeScript | SSR storefront, wallet adapters. |
| Wallet | Solana Wallet Adapter (Phantom) | Matches stated wallet. |
| Backend | Node.js/NestJS or Go | Strong typing, webhooks. |
| DB | PostgreSQL | Double-entry ledgers (GRQ + returns), ACID, audit. |
| On-chain | Solana + SPL Token | Low fees, fast finality. |
| Fiat | Stripe (+ PayPal SDK) | Cards, wallets. |
| Infra | Containerized, IaC | Reproducible, auditable. |

**Ledger principle:** every GRQ/Credit movement **and** every returns transaction
is a **double-entry** record. On-chain state is mirrored and reconciled nightly.

---

## 11. Build Phases & Milestones

| Phase | Scope | Exit criteria |
|---|---|---|
| **P0 — Foundations** | Accounts, catalog ($500 panels), fiat checkout (Stripe), order lifecycle, **project registry**. | Sell a panel for fiat, linked to a project. |
| **P1 — Off-chain GRQ** | Pricing engine, bonus issuance as **GRQ Credits**, ledger, referral attribution, Credit transfers + fee. | Bonus & referral Credits issued & spendable; price log audited. |
| **P2 — Returns** | Project-revenue intake, **returns/payout engine** (fiat), investor statements. | Returns paid from real project revenue, per project. |
| **P3 — On-chain GRQ** | SPL token, company wallet, Phantom linking, on-chain payouts, GRQ payment verification. | Wallet user earns & spends on-chain GRQ. |
| **P4 — Bridge** | Credit→Token 1:1 conversion, reconciliation jobs, treasury dashboard. | Invariant (§7) holds under load test. |
| **P5 — Hardening** | Security review, KYC/AML, **securities/licensing sign-off**, monitoring, referral-fraud controls. | Legal green-light; pen-test passed. |

---

## 12. Key Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Panel-returns product = unregistered securities offering | Securities/licensing review before taking funds; variable (not guaranteed) returns; revenue-funded payouts only. |
| Returns funded from new sales (Ponzi pattern) | Hard separation of returns ledger from sales; per-project revenue-in vs payout-out monitoring. |
| GRQ reclassified as crypto-asset/security | Off-chain-first; no cash-out; legal review; loyalty framing. |
| "Points you can't spend" | Panel-supply reserve sized to GRQ liability (§8). |
| Referral abuse / self-referral | KYC, fiat-only referral trigger, velocity limits, manual review. |
| Margin erosion from GRQ redemption | Monitor redemption ratio; tune `B`/`α` prospectively. |
| On/off-chain balance drift | Nightly reconciliation + alerting; atomic bridge ops. |
| Cross-border legal/tax complexity | Per-project legal & tax review as projects are added. |

---

## 13. Open Questions for Stakeholders

1. **Returns:** paid in fiat only, or optionally GRQ Credits / auto-reinvest?
   What determines each project's return rate and cadence?
2. Confirm GRQ parameters: `P₀=$0.005`, `α=9`, `B=5%`, `R=2%`, `f=0.5%`.
3. Is over-issuance past `N₀` a real plan or a contingency only?
4. On-chain GRQ token in v1, or Credits-only at launch?
5. First target markets/projects (drives securities & tax scoping).
6. Expected panel sales volume & cadence (drives treasury & returns sizing).

---

*See `02_Realworld_Flow.md` for a worked numeric walkthrough using these values.*
