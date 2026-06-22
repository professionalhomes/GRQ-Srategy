# GRQ Solar — Development & Token Strategy Document

**Project:** Tokyo branch of an overseas solar-panel sales company
**Product:** Online sales of solar panels, payable in fiat or in GRQ tokens
**Document version:** 1.0 · 2026-06-22
**Status:** Strategy draft (for internal review)

---

## 1. Executive Summary

GRQ Solar sells solar-panel systems through a website. Customers can pay with
**fiat** (Stripe / credit card / PayPal) or with **GRQ tokens** (an on-chain
SPL token on Solana) / **GRQ Credits** (the off-chain equivalent held inside the
system).

The strategic lever is a **loyalty-token reward loop**:

1. Every fiat purchase pays the buyer a **GRQ bonus** (a % of the purchase
   value, converted at the *current* GRQ price).
2. The GRQ price **rises monotonically** as more tokens are distributed.
3. Therefore **early customers accumulate cheap GRQ that gains purchasing power
   over time**, which they can only ever spend on *more solar panels*.
4. A **referral program** pays inviters 1.5–2.5 % of an invitee's purchase in
   GRQ, driving organic growth.

The token is deliberately a **closed-loop store of purchasing power, not a
tradeable financial instrument**: it cannot be redeemed for cash or swapped for
other currencies *through the system*. This is both a product decision and a
regulatory-risk decision (see §9).

> ⚠️ **Important framing note.** Because the price formula guarantees increases,
> it is tempting to market GRQ as an "investment that profits later." Doing so in
> Japan risks classifying GRQ as a regulated crypto-asset or even a security.
> The recommended position is to market GRQ as **discount/loyalty purchasing
> power for solar panels**, never as an investment. See §9.

---

## 2. Goals & Non-Goals

### Goals
- Launch a compliant Japanese e-commerce storefront for solar panels.
- Implement dual payment rails: fiat and GRQ/GRQ Credits.
- Implement a transparent, deterministic GRQ pricing engine.
- Implement bonus issuance, referral attribution, and on-chain/off-chain
  custody.
- Make the on-chain/off-chain distinction invisible to non-crypto users.

### Non-Goals (v1)
- No fiat *withdrawal* or cash-out of GRQ. GRQ is spend-only.
- No exchange listing, no AMM/liquidity pool, no GRQ↔other-token swaps.
- No secondary marketplace operated by GRQ Solar.

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
                         │  Auth · Orders · Pricing Engine · Ledger    │
                         └───┬───────────┬──────────────┬──────────────┘
                             │           │              │
              ┌──────────────▼──┐  ┌─────▼──────┐  ┌────▼─────────────┐
              │  Fiat Payments  │  │  Off-chain │  │   On-chain       │
              │  Stripe/CC/     │  │  Ledger    │  │   Solana service │
              │  PayPal         │  │ (GRQ Credit│  │ (GRQ SPL token,  │
              │                 │  │  balances) │  │  company wallet) │
              └─────────────────┘  └────────────┘  └──────────────────┘
                                         │              │
                                         └──────┬───────┘
                                          1:1 Credit→Token
                                          conversion bridge
```

### Core components
| Component | Responsibility |
|---|---|
| **Auth & Accounts** | Sign-up, KYC tier, wallet linking (Phantom / Connect Wallet). |
| **Catalog & Orders** | Panel SKUs, quantities, order lifecycle. |
| **Payments (fiat)** | Stripe, credit card, PayPal; idempotent webhooks. |
| **Payments (token)** | Verify inbound GRQ transfer to company wallet; debit GRQ Credits. |
| **Pricing Engine** | Single source of truth for current GRQ price (see §5). |
| **Issuance & Ledger** | Mint/allocate bonus GRQ, referral GRQ; double-entry ledger. |
| **Custody Bridge** | On-chain wallet ops + off-chain Credit ledger + 1:1 conversion. |
| **Admin / Treasury** | Parameter config, reserve monitoring, reporting. |

---

## 4. Token Model: GRQ Token vs GRQ Credit

| | **GRQ Token** | **GRQ Credit** |
|---|---|---|
| Form | On-chain SPL token (Solana) | Off-chain balance in system DB |
| For whom | Users with wallet knowledge | Users without wallet knowledge |
| Custody | User's own wallet (Phantom) | Custodied by GRQ Solar |
| Created when | User has linked a wallet | User has no linked wallet |
| Spend on panels | ✅ | ✅ |
| Transfer | Wallet→wallet (on-chain) | Account→account (with small fee) |
| Convertible | — | **GRQ Credit → GRQ Token, 1:1, user-initiated** |
| Cash-out | ❌ never | ❌ never |

**Key rule:** A purchase paid with GRQ **or** GRQ Credit earns **no** new GRQ
bonus. Bonuses are only minted against **fiat** purchases. This prevents a
self-referential inflation loop.

The 1:1 Credit→Token conversion is one-directional in practice for UX (a user
who "graduates" to having a wallet pulls their off-chain balance on-chain). The
price formula is identical for both, so conversion is value-neutral.

---

## 5. GRQ Pricing Formula

### 5.1 Design requirements
- **Deterministic & auditable** — anyone can recompute the price from public
  state.
- **Monotonically non-decreasing** — price only goes up as distribution grows.
- **Bounded growth, not a blow-up** — avoid a `1/(1-r)` singularity that would
  make late panels unaffordable in GRQ terms.
- **Well-defined past full distribution** — the formula must keep working when
  cumulative distribution exceeds the initial issuance constant.

### 5.2 Parameters (constants)

| Symbol | Meaning | Initial value |
|---|---|---|
| `P₀` | Initial GRQ price | **¥10.00** |
| `N₀` | Initial issuance constant | **1,000,000,000 GRQ** |
| `α`  | Price-growth coefficient | **9** |
| `B`  | Bonus rate on fiat purchases | **5 %** |
| `R`  | Referral rate | **1.5 – 2.5 %** (default 2 %) |
| `f`  | GRQ-Credit transfer fee | **0.5 %** |

`D` = cumulative GRQ distributed to users (bonus + referral), the only
*variable* in the formula.

### 5.3 The formula (recommended: linear bonding curve)

```
P(D) = P₀ · ( 1 + α · D / N₀ )
```

Equivalently, with `r = D / N₀` (the distribution ratio):

```
P(r) = P₀ · (1 + α · r)
```

**Why linear?**
- At `D = 0` → `P = P₀ = ¥10`.
- At `D = N₀` (full distribution, `r = 1`) → `P = P₀·(1+α) = ¥100` — a clean
  **10×** over the life of the initial issuance.
- It is the easiest curve to explain to customers and regulators.
- It never blows up: past full distribution (`r > 1`) it simply keeps rising
  linearly, which is exactly the desired over-issuance behavior.

**Over-issuance behavior.** When the entire `N₀` has been distributed, GRQ Solar
may continue issuing. Because `N₀` is a *constant in the denominator* (not the
live supply), `D` keeps growing past `N₀`, `r > 1`, and the price keeps climbing
along the same line. Example: at `r = 1.5` → `P = ¥10·(1+9·1.5) = ¥145`.

### 5.4 Optional convex variant (for later consideration)

If a steeper "early-bird" reward is desired, a mild power curve can be used
instead:

```
P(D) = P₀ · ( 1 + α · (D / N₀)^β ),   with β > 1  (e.g. β = 1.3)
```

This makes early tokens even cheaper relative to later ones. **Recommendation:
ship the linear curve first** — it is predictable and defensible. Treat `β` as a
governance-controlled future option, never changed retroactively.

### 5.5 Price-update mechanics
- The price is recomputed **on every issuance event** and **read at the moment
  of each transaction**.
- A purchase uses the price *as of order confirmation* (price-lock for the
  duration of checkout, e.g. 10 minutes, to avoid mid-payment drift).
- All price changes are written to an **append-only price-history log** for
  audit and customer transparency.

---

## 6. Earning & Spending Flows

### 6.1 Earning GRQ
1. **Purchase bonus (fiat only).** Buyer pays ¥X by fiat → receives
   `(B · X) / P(D)` GRQ, credited on-chain (if wallet linked) or as GRQ Credit.
2. **Referral.** Invitee's link carries a referral ID. When the invitee makes a
   *fiat* purchase of ¥X, the inviter receives `(R · X) / P(D)` GRQ/Credit.

Both events increase `D`, nudging the price up.

### 6.2 Spending GRQ
- Buyer chooses "Pay with GRQ" / "Pay with GRQ Credit" at checkout.
- System quotes the panel price in GRQ at the current `P(D)`:
  `GRQ_needed = panel_price_JPY / P(D)`.
- On-chain: user transfers GRQ from Phantom → company wallet; backend verifies
  the transaction signature & amount before fulfilling.
- Off-chain: system debits the GRQ Credit balance.
- **No bonus** is issued on GRQ/Credit-paid purchases (§4).

### 6.3 Transfers
- **GRQ token → GRQ token:** standard on-chain transfer (network fee only).
- **GRQ Credit → GRQ Credit:** internal ledger move, **fee `f` = 0.5 %**
  deducted from the sender.

---

## 7. Custody, Wallets & the On/Off-chain Bridge

- **Wallet-savvy users** link a Phantom/Connect wallet; the system records the
  public address and sends bonus GRQ directly on-chain.
- **Non-savvy users** receive **GRQ Credits** held off-chain. The system runs an
  **omnibus company wallet** backing all outstanding Credits 1:1.
- **Graduation:** when a non-savvy user later links a wallet, they may convert
  Credits → Tokens at 1:1; the bridge mints/transfers the on-chain amount and
  burns the off-chain balance atomically (saga / two-phase with reconciliation).
- **Invariant (must always hold):**
  `on-chain GRQ held for users + outstanding GRQ Credits = total distributed D`.
  A nightly reconciliation job asserts this and alerts on drift.

---

## 8. Treasury & Reserve Policy

GRQ is spend-only on panels, so every GRQ eventually returns to GRQ Solar as
payment for a panel — at a *higher* price than it was issued. The economic cost
to the company is the **bonus discount**: it gives away `B` (5 %) of fiat
revenue as future purchasing power.

- The 5 % bonus is effectively a **prepaid, appreciating discount** on future
  panel sales. Model it as a **deferred-revenue / loyalty liability** in
  accounting.
- Maintain a **panel-supply reserve** sized to the outstanding GRQ liability so
  that GRQ holders can always actually buy panels (avoid a "points you can't
  spend" failure).
- Monitor the **redemption ratio** (GRQ-paid sales ÷ total sales). If it climbs
  too fast, margins compress; adjust `B`, `α`, or panel pricing prospectively.

---

## 9. Regulatory & Compliance Strategy (Japan) — **read before launch**

> This section flags risk areas. It is **not legal advice.** Engage Japanese
> counsel (fintech/crypto) before go-live.

- **Crypto-Asset classification (Payment Services Act, 資金決済法).** A token that
  is transferable and used as payment can be a "Crypto Asset," requiring the
  operator to register as a **Crypto-Asset Exchange Service Provider** with the
  **FSA/JFSA**. The fact GRQ is on-chain and transferable wallet-to-wallet is a
  risk factor here.
- **Prepaid Payment Instrument (前払式支払手段).** GRQ Credits that are spend-only
  inside GRQ Solar look much more like a **prepaid instrument / store points**,
  a lighter regime. Keeping Credits non-transferable-for-cash supports this.
- **Investment/Securities framing.** *Never* market guaranteed appreciation or
  "profit." Position GRQ as **loyalty purchasing power**, denominated in
  panels, with a transparent discount mechanic.
- **No cash-out, by design.** The hard "cannot exchange for other currencies"
  rule is your single strongest compliance argument — preserve it.
- **AML/KYC.** Tiered KYC: light for fiat e-commerce; stricter for on-chain
  wallet linking and large GRQ transfers.
- **Consumer protection / 特定商取引法.** Clear disclosure of pricing mechanics,
  bonus terms, and that GRQ has no cash value.
- **Tax.** Determine VAT/consumption-tax treatment of bonuses and the timing of
  revenue recognition with an accountant.

**Recommended compliance posture:** launch with **GRQ Credits (off-chain,
prepaid-instrument-like) as the default**, and treat the on-chain GRQ token as
an *opt-in advanced feature* gated behind stricter KYC and a legal green-light.

---

## 10. Technology Stack (proposed)

| Layer | Choice | Rationale |
|---|---|---|
| Frontend | Next.js + TypeScript | SSR storefront, wallet adapters. |
| Wallet | Solana Wallet Adapter (Phantom) | Matches the stated wallet. |
| Backend | Node.js/NestJS or Go | Strong typing, webhook handling. |
| DB | PostgreSQL | Double-entry ledger, ACID, audit. |
| On-chain | Solana + SPL Token | Low fees, fast finality. |
| Fiat | Stripe (+ PayPal SDK) | Cards, wallets, JP support. |
| Infra | Containerized, IaC | Reproducible, auditable. |

**Ledger principle:** every GRQ/Credit movement is a **double-entry** record.
The on-chain state is mirrored; the DB ledger is the source of truth for
balances, reconciled against chain nightly.

---

## 11. Build Phases & Milestones

| Phase | Scope | Exit criteria |
|---|---|---|
| **P0 — Foundations** | Accounts, catalog, fiat checkout (Stripe), order lifecycle. | Can sell a panel for fiat end-to-end. |
| **P1 — Off-chain GRQ** | Pricing engine, bonus issuance as **GRQ Credits**, ledger, referral attribution, Credit transfers + fee. | Bonus & referral Credits issued & spendable; price log audited. |
| **P2 — On-chain GRQ** | SPL token, company wallet, Phantom linking, on-chain bonus payouts, GRQ payment verification. | A wallet user earns & spends on-chain GRQ. |
| **P3 — Bridge** | Credit→Token 1:1 conversion, reconciliation jobs, treasury dashboard. | Invariant (§7) holds under load test. |
| **P4 — Hardening** | Security review, KYC/AML, compliance sign-off, monitoring, fraud controls on referrals. | Legal green-light; pen-test passed. |

---

## 12. Key Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Regulatory reclassification of GRQ as crypto-asset/security | Off-chain-first launch; no cash-out; legal review; loyalty framing. |
| "Points you can't spend" (supply can't meet GRQ demand) | Panel-supply reserve sized to GRQ liability (§8). |
| Referral abuse / self-referral fraud | KYC, fiat-only referral trigger, velocity limits, manual review thresholds. |
| Margin erosion from rising redemption | Monitor redemption ratio; tune `B`/`α` prospectively (never retroactively). |
| On/off-chain balance drift | Nightly reconciliation + alerting; atomic bridge ops. |
| Price-manipulation perception | Fully public, deterministic formula + append-only price log. |
| Mid-checkout price drift | Price-lock window at order confirmation. |

---

## 13. Open Questions for Stakeholders

1. Currency of denomination — JPY assumed; confirm.
2. Final values for `P₀`, `α`, `B`, `R`, `f` (defaults proposed above).
3. Is over-issuance past `N₀` a real plan, or a contingency only?
4. Does GRQ Solar want the on-chain token at all in v1, or Credits-only?
5. Target average panel price and expected monthly sales volume (drives
   treasury sizing).

---

*See `02_Realworld_Flow.md` for a worked numeric walkthrough using the values
proposed here.*
