# SOL Staking

Dawn Labs provides multiple staking options built on our Solana validator infrastructure.

## Native Staking

Direct delegation to Dawn Labs' validator with **0% commission** — the simplest way to earn staking rewards on Solana.

| | |
|---|---|
| **Identity** | `4k6wgP5WPBKQpsFGtzuXNrjcTE2fKWLj17nDvFeG5zSF` |
| **Vote Account** | `8zuMRTXThoPTTPLLvaiKiJshLLCqGMt9BdRjjCL19xBc` |
| **Commission** | 0% |
| **MEV Commission** | 0% |
| **Uptime** | 99.9%+ |

- Top 10 performing validator on Solana
- Multi-level security including HSM key management, sentry nodes, and DDoS protection
- Multi-region redundancy with continuous monitoring

**Explorer Links:** [JPool](https://app.jpool.one/validators/8zuMRTXThoPTTPLLvaiKiJshLLCqGMt9BdRjjCL19xBc?epoch=943) | [Rated](https://explorer.rated.network/v/8zuMRTXThoPTTPLLvaiKiJshLLCqGMt9BdRjjCL19xBc?network=solana&timeWindow=30d) | [Validators.app](https://www.validators.app/validators/4k6wgP5WPBKQpsFGtzuXNrjcTE2fKWLj17nDvFeG5zSF?locale=en&network=mainnet)

## LST Staking (dawnSOL)

Liquid staking through **dawnSOL** on Sanctum with **0% commission** — use across DeFi while earning staking rewards.

- [Swap to dawnSOL on Sanctum](https://app.sanctum.so/stake/dawnSOL)

## VIP Kickback

Exclusive kickback plans for large-scale stakers. [Contact us](#contact) for details.

---

## Multiple Staking (Leveraged Looping)

Amplified staking yield through leveraged looping strategies. Coming soon.

### Jupiter Native Stake Multiple

A looping strategy using Jupiter Lend to borrow SOL against natively staked SOL, then restake the borrowed SOL.

**How it works:**
1. Stake SOL with Dawn Labs validator
2. Tokenize staked SOL into nsToken (via Solana's Single Pool Program)
3. Deposit nsToken as collateral on Jupiter Lend
4. Borrow SOL and restake — repeat to achieve up to ~7x leverage

| | |
|---|---|
| **Collateral** | Natively staked SOL (nsToken) |
| **Borrow Asset** | SOL |
| **Max Leverage** | ~7x |
| **LST Price Risk** | None (native stake, not LST) |
| **Operation** | Manual (Jupiter developing automated management) |

**Advantages:**
- No LST price deviation risk (native stake used directly)
- SOL-to-SOL borrow structure — no USD price-driven liquidation risk
- Utilization-based variable rate model dampens sudden rate spikes

**Risks:**
- **Borrow Rate Volatility** — If borrow rate exceeds staking yield for an extended period, could push collateral ratio below liquidation. Historically has never occurred.
- **Smart Contract Risk** — Jupiter audited by OtterSec (4 audits), Offside Labs, Zenith, Sec3, Code4rena ($107K scope, Feb-Mar 2026). [Full audit list](https://dev.jup.ag/resources/audits)
- **Liquidity Risk** — Unstaking requires ~2-3 day cooldown period
- **Manual Management** — Looping currently requires manual operations. Jupiter has planned automated features.

### Kamino LST Multiple

A looping strategy using Kamino (K-Lend) and Sanctum to leverage dawnSOL for amplified staking yield. One-click leveraged position via Kamino Multiply.

**How it works:**
1. Stake SOL into dawnSOL (via Sanctum)
2. Deposit dawnSOL as collateral on Kamino
3. Kamino Multiply uses Flash Loans to build the entire leveraged position in a single transaction

| | |
|---|---|
| **Collateral** | dawnSOL (LST) |
| **Borrow Asset** | SOL |
| **Max Leverage** | ~10x (with eMode) |
| **Operation** | One-click (Kamino Multiply) |

**eMode (Elevation Mode):** Raises LTV cap for correlated asset pairs (SOL and LST). Standard: 75% LTV (~4x), eMode: 90% LTV (~10x).

**Risks:**
- **Borrow Rate Volatility** — Same as Jupiter Native Stake Multiple. SOL-to-SOL structure mitigates.
- **Smart Contract Risk** — Kamino audited by Certora, OtterSec, Sec3, Ackee, OSec, RX Auditors. [Full audit list](https://github.com/Kamino-Finance/audits). Bug bounty: [$1.5M on Immunefi](https://immunefi.com/bug-bounty/kamino)
- **LST Depeg Risk** — If dawnSOL market price deviates from theoretical price, collateral value drops. Mitigated by Sanctum redemption guarantee and Kamino auto-deleverage.

**Kamino Risk Management:**
- Auto-Deleverage: Automatically unwinds when LTV deteriorates
- Partial Liquidation: Only ~2% of collateral liquidated at threshold (not full liquidation)
- KRAF: Comprehensive risk assessment with volatility measurement, stress testing, real-time monitoring
