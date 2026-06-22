# GRQ Solar — Real-World Flow with Numbers

**Companion to:** `01_Development_Strategy.md`
**Purpose:** Walk through concrete scenarios using realistic, internally
consistent figures so the token economics can be sanity-checked.
**All figures in JPY (¥).** Numbers are illustrative, not financial projections.

---

## 0. Parameters used in this walkthrough

| Symbol | Meaning | Value |
|---|---|---|
| `P₀` | Initial GRQ price | ¥10.00 |
| `N₀` | Initial issuance constant | 1,000,000,000 GRQ |
| `α`  | Price-growth coefficient | 9 |
| `B`  | Bonus rate (fiat purchases) | 5 % |
| `R`  | Referral rate | 2 % |
| `f`  | GRQ-Credit transfer fee | 0.5 % |
| — | Average panel system price | ¥1,000,000 |

**Pricing formula:**  `P(D) = ¥10 · (1 + 9 · D / 1,000,000,000)`

where `D` = cumulative GRQ distributed to users.

- At `D = 0`: **¥10**
- At `D = N₀` (fully distributed): **¥100** (a 10× lifecycle increase)
- Beyond `N₀` (over-issuance): price keeps rising linearly (e.g. `D = 1.5·N₀` → ¥145)

---

## 1. Day-one customer: Aiko (fiat, no crypto knowledge)

Aiko signs up and buys one ¥1,000,000 panel system with her **credit card**.
The system is brand new, so `D ≈ 0` and the price is `P = ¥10.00`.

| Step | Detail |
|---|---|
| Pays | ¥1,000,000 by credit card (Stripe) |
| Bonus | 5 % × ¥1,000,000 = **¥50,000 worth** of GRQ |
| GRQ received | ¥50,000 ÷ ¥10.00 = **5,000 GRQ** |
| Custody | Aiko has no wallet → **5,000 GRQ Credits** held off-chain |

Aiko now holds 5,000 GRQ Credits. She paid full price for her panel; the bonus
is a *prepaid, appreciating discount* on her **next** panel.

---

## 2. The price rises as the company grows

As more customers buy panels with fiat, `D` grows and the price climbs. Using
the formula (assuming ~40 % of sales also trigger a 2 % referral payout):

| Cumulative fiat sales | GRQ price `P(D)` | GRQ minted *per* new sale (bonus) | Total distributed `D` | `D` as % of `N₀` | Cumulative revenue |
|---:|---:|---:|---:|---:|---:|
| 1 | ¥10.00 | 5,000 | 5,800 | 0.0 % | ¥1.0 M |
| 10,000 | ¥14.30 | 3,497 | 47.7 M | 4.8 % | ¥10 B |
| 50,000 | ¥24.94 | 2,005 | 166.0 M | 16.6 % | ¥50 B |
| 100,000 | ¥33.82 | 1,478 | 264.7 M | 26.5 % | ¥100 B |
| 150,000 | ¥40.82 | 1,225 | 342.4 M | 34.2 % | ¥150 B |
| 200,000 | ¥46.78 | 1,069 | 408.6 M | 40.9 % | ¥200 B |
| 250,000 | ¥52.06 | 960 | 467.3 M | 46.7 % | ¥250 B |
| 300,000 | ¥56.85 | 879 | 520.6 M | 52.1 % | ¥300 B |

**Read this table as:** the *earlier* you buy, the *more* GRQ you get per ¥1M
spent (5,000 → 879 GRQ), **and** each of those GRQ is worth more later. Both
effects favor early adopters — exactly the intended incentive.

---

## 3. Aiko comes back later — her bonus pays off

Time passes; the company has grown and the price is now **¥50.00**
(roughly the ~225,000-sales mark). Aiko wants a second ¥1,000,000 panel.

She still holds her **5,000 GRQ Credits** from day one.

| | Value |
|---|---|
| Her 5,000 GRQ Credits are now worth | 5,000 × ¥50 = **¥250,000** |
| Panel price | ¥1,000,000 |
| Pays with Credits | ¥250,000 (5,000 GRQ Credits) |
| Remaining due | ¥750,000 (by fiat) |
| Bonus on the fiat portion | 5 % × ¥750,000 ÷ ¥50 = **750 GRQ** |
| Bonus on the GRQ-paid portion | **0** (no bonus on token/Credit payments) |

Aiko's free day-one bonus covered **25 %** of her second panel. Her purchasing
power grew 5× (¥50,000 → ¥250,000) **without GRQ ever being cashable for
money** — it only ever bought more panels.

> **Early-buyer purchasing power, generalized** (5,000 GRQ from a day-one ¥1M buy):
>
> | When price reaches | Those 5,000 GRQ buy | Multiple |
> |---:|---:|---:|
> | ¥20 | ¥100,000 of panels | ×2 |
> | ¥50 | ¥250,000 of panels | ×5 |
> | ¥100 | ¥500,000 of panels | ×10 |

---

## 4. Wallet-savvy customer: Ken (on-chain GRQ)

Ken understands crypto. At sign-up he links his **Phantom wallet**.
He buys a ¥1,000,000 panel by fiat when the price is **¥20.00**.

| Step | Detail |
|---|---|
| Pays | ¥1,000,000 (PayPal) |
| Bonus | 5 % × ¥1,000,000 = ¥50,000 worth |
| GRQ received | ¥50,000 ÷ ¥20 = **2,500 GRQ** |
| Custody | Sent **on-chain** to Ken's Phantom address |

Later, Ken pays for a ¥1,000,000 panel **entirely in GRQ** when the price is **¥40**:

| Step | Detail |
|---|---|
| GRQ needed | ¥1,000,000 ÷ ¥40 = **25,000 GRQ** |
| Action | Ken transfers 25,000 GRQ from Phantom → company wallet |
| Verification | Backend confirms the on-chain tx signature & amount, then fulfills |
| Bonus | **0** (GRQ-paid purchase earns no bonus) |

---

## 5. Referral flow: Ken invites Aiko's friend Mei

Ken shares his referral link (carries his referral ID). **Mei** clicks it, signs
up, and buys a ¥1,000,000 panel by **fiat** when the price is **¥30**.

| Beneficiary | Calculation | Result |
|---|---|---|
| **Mei** (buyer bonus, 5 %) | ¥50,000 ÷ ¥30 | **1,667 GRQ** |
| **Ken** (referral, 2 %) | ¥20,000 ÷ ¥30 | **667 GRQ** (to his wallet) |

Both payouts increase `D`, nudging the price up for the next buyer. Referral is
paid **only on fiat purchases** (an invitee paying in GRQ generates no referral),
which blocks circular farming.

---

## 6. Credit → Token conversion: Aiko "graduates"

Aiko learns about wallets and installs Phantom. She converts her off-chain
balance to on-chain tokens, **1:1, value-neutral**.

| Before | After |
|---|---|
| 4,250 GRQ **Credits** (off-chain) | 4,250 GRQ **Tokens** (on-chain, in Phantom) |

The bridge mints/sends 4,250 GRQ on-chain and burns the 4,250 off-chain Credits
in one atomic operation. The price formula is identical for both, so no value is
created or destroyed. The system invariant still holds:

```
on-chain GRQ for users + outstanding GRQ Credits = total distributed D
```

---

## 7. Credit transfer (with fee)

Aiko sends 1,000 GRQ Credits to her sister's account.

| | Value |
|---|---|
| Sent | 1,000 GRQ Credits |
| Fee `f` (0.5 %) | 5 GRQ Credits |
| Sister receives | **995 GRQ Credits** |

(On-chain token→token transfers incur only the Solana network fee, not `f`.)

---

## 8. Reaching — and passing — full distribution

| Milestone | Cumulative fiat sales | GRQ price | `D` vs `N₀` |
|---|---:|---:|---:|
| Half distributed | ~290,000 | ¥55 | 50 % |
| ~70 % distributed | 500,000 | ¥72.94 | 69.9 % |
| **Full distribution (`D = N₀`)** | **~948,000** | **¥100.00** | **100 %** |
| Over-issuance | 1,000,000 | ¥102.66 | 103.0 % |
| Over-issuance | 1,500,000 | ¥125.54 | 128.4 % |
| Over-issuance | 2,000,000 | ¥144.84 | 149.8 % |

**Over-issuance is well-defined.** Because `N₀` is a fixed constant in the
denominator (not the live supply), once the original 1 B GRQ are distributed the
company can keep issuing; `D` simply exceeds `N₀`, the ratio `D/N₀` passes 1, and
the price continues up the same straight line. No discontinuity, no division by
zero.

---

## 9. Company-side economics (intuition)

GRQ is **spend-only on panels**, so every GRQ eventually comes back to GRQ Solar
as payment — at a higher price than it was issued.

- The real cost of the program is the **5 % bonus**: GRQ Solar forgoes 5 % of
  fiat revenue as future, appreciating panel-purchasing power.
- Treat outstanding GRQ/Credits as a **loyalty liability / deferred revenue**,
  and hold a **panel-supply reserve** sized to it so holders can always actually
  redeem (§8 of the strategy doc).
- **Redemption ratio** (GRQ-paid sales ÷ total sales) is the key margin dial —
  monitor it and tune `B`/`α` **prospectively** if margins compress.

---

## 10. One-screen summary of the loop

```
  Customer pays FIAT  ─────────────►  GRQ Solar revenue
        │                                   │
        │ 5% bonus @ current price          │ price = ¥10·(1 + 9·D/N₀)
        ▼                                   │  rises as D grows
   Customer gets GRQ / GRQ Credit  ◄────────┘
        │
        │ (spend-only on panels — never cash)
        ▼
  Customer buys MORE panels with GRQ
        │  (no new bonus on GRQ-paid orders)
        ▼
   GRQ returns to GRQ Solar  ──►  loop closes
```

**The flywheel:** fiat sales → cheap early GRQ + rising price → early customers'
purchasing power grows → they buy more panels → more sales. Referrals widen the
top of the funnel. Compliance guardrail throughout: **GRQ is loyalty purchasing
power, never an investment, never cashable.**

---

*Reproduce these numbers from the formula in §0; the cumulative table assumes
~40 % of sales also trigger a 2 % referral. See `01_Development_Strategy.md` §5
for formula derivation and §9 for the regulatory posture behind the "no cash-out"
rule.*
