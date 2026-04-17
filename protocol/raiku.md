# Raiku

A reference guide for enterprises evaluating **Raiku** — a Solana coordination layer that replaces probabilistic priority-fee bidding with deterministic block-space reservations.

{% hint style="warning" %}
**Status (April 2026)**: Raiku is **pre-mainnet**. v2 SDK rollout is expected late 2025, mainnet launch targeted for 2026. All pricing figures in this document are modeled from existing Solana priority-fee data and are **not observed Raiku rates**.
{% endhint %}

---

## What Raiku Is

Raiku is a **block-building coordination layer** for Solana. Rather than competing in Solana's per-block priority-fee auction (where higher bids probabilistically get faster inclusion), Raiku lets applications **pre-book or instantly purchase blockspace** from participating validators.

| | Traditional Solana | Raiku |
|---|---|---|
| **Inclusion model** | Probabilistic (priority fee bidding) | Deterministic (reserved or auction-won) |
| **Slot targeting** | No — "best effort" | Yes — specific future slot (AOT) |
| **Pre-confirmation** | No | Sub-50ms |
| **Claimed reliability** | Varies widely under load | 99.9%+ within Raiku-controlled slots |
| **Pricing** | Market-driven priority fee | Auction (AOT) or priority fee × 1.05 + routing (JIT) |

---

## How It Works

Raiku operates within slots led by **Raiku-opted-in validators**. Non-Raiku slots remain under the normal Solana / Jito flow — Raiku has no control there.

### Two Execution Models

**AOT — Ahead-of-Time**

* English-style sealed-bid auction for a **specific future slot**
* Auction runs roughly 36 seconds (~100 slots) before the target, closes ~90 slots prior
* Winners receive an **inclusion signal** — a pre-confirmation that the transaction will land
* Use cases: scheduled settlements, oracle updates, liquidations, arbitrage on known events

**JIT — Just-in-Time**

* First-price sealed-bid auction for the **current leader's block**
* **Minimum price = current priority fee × 1.05 + routing fee**
* Sub-second execution when the current leader is Raiku-enabled
* Falls back to Jito / priority fees when the current leader is non-Raiku
* Use cases: opportunistic MEV, HFT, reactive trades

### Raiku Stake Coverage Constraint

Raiku can only operate during slots led by opted-in validators. Because Solana's leader schedule is fixed per epoch, the percentage of network stake opted in to Raiku directly determines coverage.

| Raiku stake | Raiku slots / min | Avg gap between Raiku windows |
|---|---|---|
| 10% | ~15 | 14.4 s |
| 30% | ~45 | 4.7 s |
| 50% | ~75 | 1.6 s |

During gaps, JIT falls back to Jito / priority fees. AOT is only usable for slots a Raiku validator will lead.

---

## Cost Analysis

### Scenario-Based Pricing (modeled from Solana data)

Raiku's own simulator uses four scenarios, each derived from historical priority-fee + MEV-tip data (not Raiku bids).

| Scenario | Modeled fee/CU | Source methodology |
|---|---|---|
| **Current Market** | 0.10 lam/CU | 30-day p25 of active payers + 0.10 floor (n=83) |
| **Last 12 Months** | 1.50 lam/CU | 30-day CU-weighted non-base mean (n=117) |
| **Last 24M + Congestion** | 2.00 lam/CU | 30-day CU-weighted total-fee mean (n=117) |
| **Bull Case** | 3.50 lam/CU | 30-day p75 of active payers (n=83) |

### Per-Transaction Cost (200,000 CU typical)

At SOL = $100:

| Scenario | Fee/CU | Per-tx cost |
|---|---|---|
| Current Market | 0.10 lam | ~$0.002 |
| Last 12 Months | 1.50 lam | ~$0.030 |
| Elevated / Congestion | 2.00 lam | ~$0.040 |
| Bull Case (at $250 SOL) | 3.50 lam | ~$0.175 |

Traditional Solana priority fees typically land in **$0.00025–$0.001** during normal periods and can spike to **several dollars** during acute congestion. Raiku's predictability matters most during spikes.

### Key Pricing Insight

**JIT is always at least 5% more expensive than the prevailing priority fee** — Raiku JIT is never cheaper than Jito for the same slot.

**AOT pricing depends on auction dynamics** — can be lower than priority fees (if demand is thin), equal, or higher (if multiple bidders compete for the same slot). Raiku's "AOT discount" marketing is not guaranteed.

**Raiku's value is certainty, not raw cost reduction.** Enterprises paying the ~5% premium are buying inclusion guarantees, not cheaper execution.

---

## Execution Certainty

### Claimed Metrics

* **99.9%+ inclusion rate** within Raiku-controlled slots
* **Sub-50ms pre-confirmation** via the coordination engine
* **Atomic execution** — transactions succeed as a unit or not at all
* **No drops under congestion** for pre-reserved AOT transactions

### Realistic Limitations

* These guarantees apply **only to slots led by Raiku validators**
* During non-Raiku slot gaps, applications still depend on Jito or priority fees
* Network-wide coverage scales with Raiku validator adoption

---

## Who Should Consider Raiku

### Raiku's Stated Target Segments

Raiku's public positioning targets use cases that demand **predictable, low-latency execution**:

* **High-frequency trading platforms**
* **AI / ML coordination systems**
* **Decentralized physical infrastructure (DePIN)**
* **Gaming applications requiring real-time state updates**

### Dawn Labs' Analysis — Practical Fit

Layering Raiku's positioning against observed Solana fee data, we see three tiers of practical fit:

**Strong fit** — execution failure is expensive:

* MEV / arbitrage — timing-critical, high fee tolerance
* Perpetuals liquidations — failed liquidations cascade into protocol risk
* Oracle updates — specific-slot delivery required
* NFT launches — launch-time inclusion is the product
* Institutional settlement — predictable execution windows for accounting

**Marginal fit**:

* DEX / AMM swaps — useful during volatility, priority fees fine in normal conditions
* Lending routine operations — only liquidation paths benefit materially

**Poor fit**:

* Retail payments / transfers — Raiku premium not economically justified
* Bulk batch processing with loose timing — priority fees remain cheaper

### Customer Category Positioning

Historical Solana data shows clear stratification in fee-per-CU willingness. AOT auctions favor high-fee-tolerant categories; lower-tolerance categories risk being outbid.

| Category (from Solana data) | Typical non-base fee/CU | AOT positioning |
|---|---|---|
| Proprietary AMM / MEV | 7.25 lam/CU | Top of auction — reliably wins slots |
| Lending (with liquidations) | ~2.0 lam/CU | Competitive |
| Payments | ~1.08 lam/CU | Mid-tier — risks being outbid in competitive blocks |
| DEX / Perps / Bridge / NFT / Gaming | < 1 lam/CU | Bottom — likely outbid during congestion |

Enterprises should benchmark their current priority-fee spend against this distribution before committing to AOT strategy.

---

## Decision Framework for Enterprises

Before engaging with Raiku, answer these questions:

1. **What is the cost of one failed transaction?** If low ($1–10), Raiku's premium is hard to justify. If high (lost MEV, cascading liquidation, broken SLA), Raiku may be cheap insurance.
2. **Do you need specific-slot execution?** Only AOT can deliver this. If "soon" is fine, JIT or regular priority fees suffice.
3. **How congestion-sensitive are you?** Raiku's biggest advantage appears during spikes — when priority fees 10x, AOT locked at auction price stays fixed.
4. **Can you absorb a 5–15% fee premium?** If margins are thin (payments, retail), no. If margins are fat (MEV, trading), yes.
5. **Where does your fee profile sit in the distribution?** Check Category Explorer or your own on-chain data. If your typical fee/CU is below 1 lamport, AOT auctions may consistently outbid you.

---

## Integration Path

| Phase | Status | Action |
|---|---|---|
| **Testnet / devnet** | Ongoing | Build against published SDK, benchmark in simulator |
| **Mainnet** | 2026 target | Production deployment |

---

## Limitations & Caveats

* **All pricing is pre-launch.** There are no observed AOT bid prices — every figure is a proxy from Solana priority-fee history.
* **The simulator is a scenario tool, not a price guarantee.** Run your own sensitivity analysis with conservative assumptions.
* **Non-Raiku slot fallback remains necessary.** Any production integration needs a Jito / priority-fee path for gaps.
* **JIT pricing floor (priority fee × 1.05) means Raiku is never cheaper than Jito for the same slot.** Cost advantages come from AOT discounts or avoided congestion-spike exposure.

---

## Resources

| Resource | URL | Purpose |
|---|---|---|
| Raiku main site | [raiku.com](https://www.raiku.com/) | Product overview |
| Documentation | [docs.raiku.com](https://docs.raiku.com/) | Technical specs, SDK docs |
| Revenue Simulator | [raiku-protocol.github.io/raiku-simulator](https://raiku-protocol.github.io/raiku-simulator/index.html) | Scenario modeling |
| Simulator guide | [raiku-simulator/guide.html](https://raiku-protocol.github.io/raiku-simulator/guide.html) | Methodology, data sources |
| Slot Marketplace overview | [raiku.com/technology/slot-marketplace](https://www.raiku.com/technology/slot-marketplace) | Auction mechanics |

---

## How Dawn Labs Can Help

For enterprises evaluating Raiku, Dawn Labs can:

* **Collaboratively assess Raiku's fit for your use case** — working through your execution requirements, fee profile, and volume together to determine whether AOT, JIT, or traditional priority fees best serve each workflow
* **Facilitate Raiku BD engagement** and early-access discussions
* **Integrate Raiku SDK paths** into existing Solana operations once mainnet lands

---

{% hint style="info" %}
**Last reviewed**: April 2026. Figures and Raiku status may change as mainnet approaches — re-verify with Raiku's official channels before making commitments.
{% endhint %}
