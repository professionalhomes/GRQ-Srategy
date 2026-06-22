# GRQ Solar — Real-World Flow with Numbers

**Companion to:** `01_Development_Strategy.md`
**Purpose:** Walk through concrete scenarios with realistic, internally
consistent figures.
**Currency: USD ($). Panel price: $500 fixed.** Numbers are illustrative, not
financial projections.

---

## 0. Parameters used in this walkthrough

| Symbol | Meaning | Value |
|---|---|---|
| `P₀` | Initial GRQ price | $0.005 |
| `N₀` | Initial issuance constant | 1,000,000,000 GRQ |
| `α`  | Price-growth coefficient | 9 |
| `B`  | Bonus rate (fiat purchases) | 5 % |
| `R`  | Referral rate | 2 % |
| `f`  | GRQ-Credit transfer fee | 0.5 % |
| — | Investor panel price | **$500 (fixed)** |

**Pricing formula:**  `P(D) = $0.005 · (1 + 9 · D / 1,000,000,000)`
where `D` = cumulative GRQ distributed to users.

- `D = 0` → **$0.005**
- `D = N₀` (fully distributed) → **$0.05** (a 10× lifecycle increase)
- Beyond `N₀` (over-issuance) → price keeps rising linearly (`D = 1.5·N₀` → $0.0725)

**Two money flows, kept separate:**
- **GRQ loyalty loop** — bonus tokens on fiat purchases (this document's focus).
- **Investment returns** — paid to panel owners from project electricity revenue
  (see §9; rate is project-specific and not yet fixed).

---

## 1. Day-one investor: Aiko (fiat, no crypto knowledge)

Aiko signs up and buys **one $500 panel** with her **credit card**. The platform
is brand new, so `D ≈ 0` and the GRQ price is `$0.005`.

| Step | Detail |
|---|---|
| Pays | $500 by credit card (Stripe) |
| Owns | 1 panel, linked to a solar project (earns returns — §9) |
| GRQ bonus | 5 % × $500 = **$25 worth** of GRQ |
| GRQ received | $25 ÷ $0.005 = **5,000 GRQ** |
| Custody | No wallet → **5,000 GRQ Credits** held off-chain |

Aiko paid full price for her panel. The 5,000 GRQ is a *prepaid, appreciating
discount* on her **next** panel — separate from the returns her panel earns.

---

## 2. The GRQ price rises as the platform grows

As more investors buy panels with fiat, `D` grows and the price climbs (table
assumes ~40 % of sales also carry a 2 % referral payout):

| Panels sold (fiat) | GRQ price `P(D)` | GRQ minted *per* new sale (bonus) | Total distributed `D` | `D` as % of `N₀` | Cumulative revenue |
|---:|---:|---:|---:|---:|---:|
| 1 | $0.0050 | 5,000 | 5,800 | 0.0 % | $0.0 M |
| 10,000 | $0.0071 | 3,497 | 47.7 M | 4.8 % | $5 M |
| 50,000 | $0.0125 | 2,005 | 166.0 M | 16.6 % | $25 M |
| 100,000 | $0.0169 | 1,478 | 264.7 M | 26.5 % | $50 M |
| 250,000 | $0.0260 | 960 | 467.3 M | 46.7 % | $125 M |
| 500,000 | $0.0365 | 686 | 699.3 M | 69.9 % | $250 M |
| 750,000 | $0.0445 | 561 | 878.3 M | 87.8 % | $375 M |
| ~948,000 | $0.0500 | 500 | 1,000.0 M | 100 % | $474 M |

**Read this as:** the *earlier* you buy, the *more* GRQ you get per $500 (5,000 →
500 GRQ), **and** each GRQ is worth more later. Both effects favor early
adopters — exactly the intended incentive.

---

## 3. Aiko comes back later — her GRQ bonus pays off

Time passes; the platform has grown and the GRQ price is now **$0.025** (around
the ~250k-panel mark). Aiko still holds her **5,000 GRQ Credits** from day one
and wants more panels.

| | Value |
|---|---|
| Her 5,000 GRQ Credits are now worth | 5,000 × $0.025 = **$125** |
| Buys | panels with those Credits |
| GRQ needed per panel | $500 ÷ $0.025 = 20,000 GRQ |
| Panels her Credits cover | $125 ÷ $500 = **0.25 of a panel** |
| Bonus on GRQ-paid portion | **0** (no bonus on token/Credit payments) |

Her free day-one bonus grew **5×** in purchasing power ($25 → $125) — and that
purchasing power only ever buys panels, never cash.

> **Early-buyer purchasing power, generalized** (5,000 GRQ from a day-one $500 buy):
>
> | When GRQ price reaches | Those 5,000 GRQ buy | Multiple |
> |---:|---:|---:|
> | $0.010 | $50 of panels | ×2 |
> | $0.025 | $125 of panels | ×5 |
> | $0.050 | $250 of panels (½ a panel) | ×10 |

*(In practice investors typically combine accumulated GRQ with fiat to buy whole
panels — the GRQ simply discounts the fiat needed.)*

---

## 4. Wallet-savvy investor: Ken (on-chain GRQ)

Ken understands crypto and links his **Phantom wallet** at sign-up. He buys
**one $500 panel** by fiat when the GRQ price is **$0.010**.

| Step | Detail |
|---|---|
| Pays | $500 (PayPal) |
| GRQ bonus | 5 % × $500 = $25 → $25 ÷ $0.010 = **2,500 GRQ** |
| Custody | Sent **on-chain** to Ken's Phantom address |

Later Ken buys another panel **entirely in GRQ** when the price is **$0.020**:

| Step | Detail |
|---|---|
| GRQ needed | $500 ÷ $0.020 = **25,000 GRQ** |
| Action | Ken transfers 25,000 GRQ Phantom → company wallet |
| Verification | Backend confirms the on-chain tx signature & amount, then fulfills |
| Bonus | **0** (GRQ-paid purchase earns no bonus) |

---

## 5. Referral flow: Ken invites Mei

Ken shares his referral link (carries his referral ID). **Mei** signs up and buys
**one $500 panel** by **fiat** when the GRQ price is **$0.015**.

| Beneficiary | Calculation | Result |
|---|---|---|
| **Mei** (buyer bonus, 5 %) | $25 ÷ $0.015 | **1,667 GRQ** |
| **Ken** (referral, 2 %) | $10 ÷ $0.015 | **667 GRQ** (to his wallet) |

Both payouts increase `D`, nudging the price up for the next buyer. Referral pays
**only on fiat purchases** (an invitee paying in GRQ triggers none) — this blocks
circular farming.

---

## 6. Credit → Token conversion: Aiko "graduates"

Aiko learns about wallets and installs Phantom, then converts her off-chain
balance to on-chain tokens, **1:1, value-neutral**.

| Before | After |
|---|---|
| 5,000 GRQ **Credits** (off-chain) | 5,000 GRQ **Tokens** (on-chain, Phantom) |

The bridge mints/sends 5,000 GRQ on-chain and burns the 5,000 off-chain Credits
atomically. The price formula is identical for both, so no value is created or
destroyed. The invariant still holds:

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

| Milestone | Panels sold (fiat) | GRQ price | `D` vs `N₀` |
|---|---:|---:|---:|
| ~Half distributed | ~290,000 | $0.0275 | 50 % |
| ~70 % distributed | 500,000 | $0.0365 | 69.9 % |
| **Full distribution (`D = N₀`)** | **~948,000** | **$0.0500** | **100 %** |
| Over-issuance | 1,000,000 | $0.0513 | 103.0 % |
| Over-issuance | 1,500,000 | $0.0628 | 128.4 % |

**Over-issuance is well-defined.** Because `N₀` is a fixed constant in the
denominator (not live supply), once the original 1 B GRQ are distributed the
platform can keep issuing; `D` simply exceeds `N₀`, the ratio passes 1, and the
price continues up the same straight line — no discontinuity, no divide-by-zero.

---

## 9. The investment side — returns from project revenue

Separate from the GRQ loop, each $500 panel earns the investor a **return paid
from its project's electricity revenue**. Per the brief, the **return rate is not
fixed** — it varies by project and country (hardware, install, land, grid,
development and financing costs all differ). So this section is **structure, not a
promised yield.**

```
Project electricity revenue ──► Returns engine ──► Investor payouts (per panel owned)
   (varies by project)            (records in,        (USD; cadence per project)
                                   allocates out)
```

**Illustrative only — placeholder, not a commitment.** *If* a given project paid,
say, a 10 % annual return, one $500 panel would pay ~$50/year; the actual figure
is set per project once economics are introduced. Returns are:

- funded **only from genuine project revenue** (never from new panel sales — §9.1
  of the strategy doc);
- tracked in a **ledger separate** from GRQ;
- in v1, **paid in fiat (USD)**; optionally GRQ Credits or auto-reinvest (open
  question).

This is the dimension that makes the panel an **investment product** and drives
the primary regulatory work (securities/licensing) before launch.

---

## 10. One-screen summary of the loop

```
  Investor pays FIAT for a $500 panel ──────►  GRQ Solar revenue
        │                    │                       │
        │ 5% bonus           │ owns panel            │ GRQ price = $0.005·(1+9·D/N0)
        │ @ current price    │ linked to a project   │  rises as D grows
        ▼                    ▼                       │
  Gets GRQ / GRQ Credit   Earns RETURNS  ◄───────────┘
        │                  (from project
        │                   electricity revenue,
        │                   paid in fiat — §9)
        │ (spend-only on panels — never cash)
        ▼
  Buys MORE panels with GRQ  (no new bonus on GRQ-paid orders)
        │
        ▼
   GRQ returns to GRQ Solar ──► loop closes
```

**The flywheel:** fiat sales → cheap early GRQ + rising price → early investors'
purchasing power grows → they buy more panels → more sales; meanwhile each panel
pays returns from real project revenue. Referrals widen the funnel. Guardrails:
**GRQ is loyalty purchasing power, never cashable; returns come only from project
revenue, never from new inflows.**

---

*Reproduce the GRQ numbers from the formula in §0 (cumulative table assumes ~40 %
of sales carry a 2 % referral). See `01_Development_Strategy.md` §5 for the
formula and §9 for the two-dimensional regulatory posture.*
