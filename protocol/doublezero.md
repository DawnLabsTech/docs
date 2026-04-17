# DoubleZero

A reference guide for enterprises evaluating **DoubleZero Edge** — a dedicated high-performance network that delivers Solana raw market data to latency-sensitive trading operations.

{% hint style="warning" %}
**Status (April 2026)**: DoubleZero Edge is in **public beta**. Performance figures below reflect DoubleZero's published claims; no third-party audited SLA is currently available.
{% endhint %}

---

## What DoubleZero Edge Is

DoubleZero is a permissionless high-performance network built on otherwise-unused private fiber capacity. **DoubleZero Edge** is its first productized service: it distributes **raw Solana block data (shreds)** from validators to subscribers over a dedicated multicast network — bypassing the public internet and traditional relay trees.

The mental model is **traditional finance market data distribution**, applied to Solana:

* Publisher (validator) sends once → network fans out to all subscribers simultaneously
* No gossip propagation, no CDN caching, no peer-to-peer relay hops
* Deterministic path over dedicated fiber, not best-effort internet

| | Public Solana Infrastructure | DoubleZero Edge |
|---|---|---|
| **Delivery network** | Public internet, gossip propagation | Dedicated fiber, multicast |
| **Data type** | Structured RPC / streaming responses | Raw shreds (pre-parse) |
| **Fanout model** | Peer-to-peer relay | Publisher → network → subscribers |
| **Pricing** | Subscription to managed API | Per-epoch USDC seat, regional |
| **Target user** | Broad developer base | Latency-sensitive trading |

---

## Business Positioning

DoubleZero Edge is **not a general-purpose infrastructure product**. It is narrowly engineered for use cases where **milliseconds of read-path advantage translate into measurable P&L**.

### What DoubleZero Edge is NOT

* **Not an RPC replacement** — cannot read wallet balances, program state, or account data
* **Not a transaction submission service** — optimizes data arrival, not transaction landing (Jito / Helius Sender still required for write-path)
* **Not a managed SaaS** — no API-key-to-endpoint onboarding
* **Not a dApp developer product** — general application teams gain little from raw shred access

### What it augments

* Sits **alongside**, not instead of, QuickNode / Alchemy / Helius / Triton — firms retain those for RPC and structured streaming, and add DoubleZero as the primary low-latency read path
* Most direct comparison in the Solana ecosystem is Jito ShredStream (raw shred feed for trading)

---

## Use Case Fit

### Strong fit — where Edge delivers material value

| Use case | Why Edge matters |
|---|---|
| **High-frequency trading on Solana** | Earlier access to leader-produced shreds translates into fill rate and slippage reduction |
| **Market makers / proprietary AMMs** | Earlier view of order flow tightens quoting and reduces adverse selection |
| **Quantitative / algorithmic trading firms** | Deterministic, low-jitter feed improves backtest-to-production parity |
| **MEV searchers** | Earlier mempool-equivalent data widens the extractable opportunity window |
| **Cross-exchange arbitrage desks** | Reduces Solana-side latency inside a multi-venue latency budget |

Launch subscribers announced by DoubleZero include Jito Labs, Harmonic, Staking Facilities, and Triton One — all latency-sensitive infrastructure operators.

### Poor fit — where Edge is misaligned or overkill

* **Wallet apps, consumer dApps, marketplaces** — public RPC is sufficient; raw shred handling is operational overhead without benefit
* **Indexers and analytics platforms** — structured streaming APIs (Helius LaserStream, QuickNode Yellowstone) are purpose-built for this and more cost-efficient
* **Transaction submission optimization** — Edge does not improve write-path; Jito / Helius Sender / premium RPC remain the answer
* **Thin-margin transaction flow** — seat pricing is calibrated to trading economics, not retail or payments volume

### Evaluation heuristic

Ask: **"Would 10–30 milliseconds of earlier market data change a trade decision we make today?"**

* If yes → Edge is worth evaluating
* If no → incremental latency will not justify the integration burden

---

## What Adopters Get

### Published performance claims

* **~28 ms faster shred delivery at p95**, measured across 50,000 slots against a public-internet baseline
* **~90 % win rate** in slots where a DoubleZero-connected validator is the leader (official figure)
* **30+ metros globally**, regionally priced
* **~45 % of Solana stake** publishing to Edge; **~52 %** connected to DoubleZero network overall

### Economic model

* **Per-seat, per-epoch, per-device** pricing in USDC
* **Dynamic and regional** — prices vary by metro and device, recalculated each epoch
* **Permissionless onboarding, no minimum commitment**
* **Seat tenure depends on escrow** — funding monitoring is an operational requirement

### What's in the feed

* Raw Solana shreds — pre-parse, pre-dedup
* Subscribers run on-prem infrastructure to receive, deduplicate, and decode
* Feed is license-restricted to internal use; retransmission or resale is not permitted

---

## Integration Reality

DoubleZero Edge is **infrastructure-grade, not application-grade**. Adoption requires:

* **Network engineering capability** — public IPv4, dedicated tunnels, routing configuration, firewall exceptions
* **Linux host operations** — daemon process integrated into kernel networking
* **On-chain wallet operations** — seat purchase, renewal, and tenure monitoring
* **Custom receive pipeline** — organizations build their own shred ingestion, dedup, and decode stack; reference examples exist but there is no turnkey SDK

Organizations with existing low-latency trading infrastructure (co-located servers, dedicated network ops, packet-level processing) will find Edge a natural extension. Organizations accustomed to managed cloud APIs face a significant operational lift.

**Recommended deployment pattern**: Add Edge as a **parallel read-path** alongside existing RPC / streaming. Existing infrastructure continues to serve RPC queries and transaction submission; Edge delivers the latency-critical market data feed into the strategy engine.

---

## Maturity & Risk Considerations

Decision-makers should weigh the following before committing:

* **Beta product status** — subscriber-side public beta, early adoption cycle
* **No published SLA** — operational dashboards exist, but no formal uptime guarantee, service credits, or remediation terms
* **Limited public audit trail** — no formal third-party security audit of Edge-specific components currently published
* **Roadmap vs. shipping features** — the DoubleZero whitepaper describes programmable edge filtration, DDoS mitigation, state sync, and MEV support; Edge today ships only the raw shred market data feed. Evaluate against what is live, not what is planned
* **New vendor dependency** — a dedicated network provider on a critical data path should be reflected in business continuity planning

### Signaled expansion

DoubleZero has indicated intent to extend Edge beyond Solana shreds to:

* Other blockchain feeds
* Centralized exchange (CEX) market data
* Prediction market data
* Traditional exchange order-by-order feeds

If realized, Edge repositions itself as a **cross-market data distribution platform** rather than a Solana-specific product. Enterprises with multi-venue trading operations should track this trajectory.

---

## Decision Framework for Enterprises

1. **Is sub-100 ms Solana data latency a revenue-linked constraint?** If no, Edge is not the product
2. **Do we have in-house network engineering?** If no, integration cost likely exceeds benefit
3. **Can we absorb a read-path-only product?** Edge solves one layer — RPC procurement and transaction submission infrastructure are still required
4. **What is our Solana leader / stake exposure?** Higher exposure amplifies Edge's value on the publisher side
5. **Are we comfortable deploying beta-phase infrastructure?** Mission-critical use typically wants formal SLAs; Edge does not yet offer them

---

## Resources

| Resource | URL | Purpose |
|---|---|---|
| DoubleZero Edge | [doublezero.xyz/dz-edge](https://doublezero.xyz/dz-edge) | Official Edge product page |

---

## How Dawn Labs Can Help

For enterprises evaluating DoubleZero Edge, Dawn Labs can:

* **Assess use case fit** — working through your trading workflow, latency sensitivity, and read-path architecture to determine whether Edge delivers material value
* **Benchmark against existing infrastructure** — measure current data-path latency and model Edge's delta in the context of your strategy P&L
* **Facilitate DoubleZero engagement** — subscriber onboarding, seat procurement, and validator publisher setup
* **Operate the receive stack** — deploy and monitor the DoubleZero daemon, tunnel, and shred ingestion pipeline alongside existing Solana infrastructure

---

{% hint style="info" %}
**Last reviewed**: April 2026. DoubleZero Edge is in active development; re-verify status and figures with official channels before making commitments.
{% endhint %}
