# Strategies

The USDC Vault uses three strategies in a two-layer architecture: **Base Layer** (Kamino Multiply + Lending) and **Alpha Layer** (SOL Delta-Neutral).

## Base Layer — Kamino Multiply (Primary)

Leveraged stablecoin loops via Kamino to earn native collateral yield + borrow rewards.

### Mechanism

1. Deposit a yield-bearing stablecoin as collateral on Kamino
2. Borrow USDC against the collateral
3. Swap borrowed USDC back to collateral and re-deposit
4. Repeat to reach the target leverage multiple
5. Net yield = (Collateral native yield × leverage) − borrow cost

### Active Pools

| Pool | Market | Effective APY | Leverage | Notes |
|------|--------|---------------|----------|-------|
| **ONyc/USDC** (Primary) | RWA Market | ~16% @ 2.5x | 2.5x | ONyc native yield via Onre |
| **USDG/PYUSD** (Backup) | Main Market | ~9.5% @ 5.75x | 5.75x | Fallback when ONyc/USDC degrades |

### Market Scanner & Pool Switching

The Market Scanner continuously monitors candidate pools and recommends switching only when:

- 24h moving-average APY advantage is large enough to repay estimated switch cost within the configured payback window (7 days)
- The destination candidate clears the live risk gate
- Minimum holding period (3 days) has elapsed

### Multiply Risk Scoring

The **Multiply Risk Scorer** evaluates each pool on **4 dimensions** as a separate risk axis. Risk does not reduce displayed APY — it **gates allocation**, trims oversized positions, and forces exits at explicit score thresholds.

| Dimension | Weight | What It Measures |
|-----------|--------|------------------|
| **Depeg Risk** | 30% | Peg deviation + 24h volatility + 7d tail risk |
| **Liquidation Proximity** | 30% | Distance to liquidation at target leverage + stress scenarios |
| **Exit Liquidity** | 20% | Swap slippage on emergency exit (via Jupiter) |
| **Reserve Pressure** | 20% | Reserve utilization + capacity ratios + TVL safety checks |

**Risk thresholds (live):**

| Score | Action |
|-------|--------|
| < 75 | Normal operation |
| ≥ 75 | Stop new deposits + trim position to dynamic `maxPositionCap` |
| ≥ 90 | Full exit / emergency deleverage |

Candidates with score ≥ 75 are excluded from switch targets.

### Deleverage Protection

Multi-stage health rate protection prevents liquidation:

| Health Rate | Action |
|-------------|--------|
| Target: 1.15 | Normal operation |
| < 1.10 | Soft deleverage — reduce 20% of position |
| < 1.05 | Emergency full deleverage |

### Excluded Pools

| Pool | Reason |
|------|--------|
| ONyc/USDG | Only 0.18% APY advantage over ONyc/USDC with worse USDG liquidity |

---

## Base Layer — Lending (Supplementary)

USDC lending on Kamino / Jupiter Lend (3-8%) absorbs capital that cannot be added to Multiply, and enforces diversification.

### Supported Protocols

| Protocol | Rate Range | Notes |
|----------|------------|-------|
| **Kamino Lend** | 3-8% | Established protocol with deep liquidity |
| **Jupiter Lend** | 3-8% | Growing protocol with competitive rates |

> **Note:** Drift has been excluded due to the 2025 hack.

### Lending Risk Scoring

The **Lending Risk Scorer** evaluates each protocol on **5 dimensions** with APY penalty adjustment:

| Dimension | Weight | What It Measures |
|-----------|--------|------------------|
| **TVL Scale** | 30% | Protocol total value locked — larger TVL indicates stability |
| **Protocol Maturity** | 20% | Time since launch and operational track record |
| **Reserve Utilization** | 25% | Borrow utilization rate — high utilization signals withdrawal risk |
| **Deposit Concentration** | 15% | Depositor concentration — high concentration increases whale risk |
| **Incident History** | 10% | History of hacks, exploits, or operational failures |

### Protocol Circuit Breaker

Autonomous safety mechanism triggering auto-exit:

| Trigger | Action |
|---------|--------|
| TVL crash (-20% in 1 hour) | Immediate withdrawal |
| Oracle drift detected | Pause new deposits + alert |
| Withdrawal failure | Immediate withdrawal attempt + alert |

### Protocol Diversification

Maximum **60% allocation cap** per single protocol.

---

## Alpha Layer — SOL Delta-Neutral

A market-neutral strategy that captures SOL funding rate payments while maintaining zero directional exposure.

> **Current Status (2026-04):** SOL-PERP funding rates have been negative for an extended period. DN strategy is dormant; 100% allocation to Base Layer.

### Mechanism

1. Split USDC 50/50: half for spot, half for margin
2. Buy SOL spot → convert to **dawnSOL** (Dawn Labs' LST, ~7% staking yield)
3. Open equal-sized SOL-PERP short on Binance (1x leverage — margin = position size)
4. Net SOL exposure = 0: spot long cancels out perp short
5. Collect funding rate payments (when positive) + staking rewards

**No leverage is used.** For a $20,000 allocation: $10,000 buys SOL spot → dawnSOL, $10,000 serves as margin for SOL-PERP short. Liquidation risk is effectively zero.

### Entry / Exit Logic (Live Config)

| Signal | Condition | Action |
|--------|-----------|--------|
| **Entry** | SOL FR > 10% annualized for 3 days | Allocate up to 70% to DN |
| **Exit** | SOL FR < 0% annualized for 3 days | Close positions |
| **Emergency Exit** | SOL FR < -10% annualized | Immediate full close |

A "day" means one complete UTC day with all 3 expected 8-hour funding samples recorded. Partial days are ignored. Emergency exit keys off the latest funding print and does not wait for a full day.

### Yield Composition

| Source | Estimated APY | Condition |
|--------|--------------|-----------|
| Funding Rate (net) | 8-23% | When FR is positive |
| dawnSOL Staking | ~7% | Always-on while position is held |
| **Combined** | **15-30%** | During favorable FR periods |

### Historical Reference (Not Live Config)

A prior 5.5-year Walk-Forward study produced:

- Entry: FR > 15% annualized for 2 days
- Exit: FR < -2% annualized for 1 day
- DN Allocation: 50%
- Result: 8.57% annualized return, Sharpe 13.41, max drawdown -0.07%

These are research context, **not** the current runtime parameters.

---

## Dynamic Allocation

The vault automatically adjusts the split between Base and Alpha layers using a state machine:

```
BASE_ONLY ←→ BASE_DN
```

| From | To | Condition |
|------|----|-----------|
| BASE_ONLY | BASE_DN | SOL FR > 10% annualized for 3 consecutive days |
| BASE_DN | BASE_ONLY | SOL FR < 0% annualized for 3 days |
| BASE_DN | EMERGENCY_EXIT | SOL FR < -10% annualized (no time condition) |

### Base Layer Internal Allocation

Within the Base Layer, capital flows in a **Multiply-first / Lending-second** mode:

1. `CapitalAllocator` sends deployable USDC to Kamino Multiply first
2. `BaseAllocator` manages lending across Kamino / Jupiter for overflow, diversification, and withdrawal buffer
3. Wallet buffer (5%) is preserved before deployment

### Excluded Strategies

| Strategy | Reason |
|---|---|
| Drift | Hack — unusable, code deprecated |
| USDC/USDT Leverage Loop | Borrow rate spike (Drift fallout) makes spread negative |
| JLP / LP / Insurance Pools | Principal loss risk — incompatible with Vault mandate |
| PRIME (Hastra Finance) | Insufficient track record |
| CASH (Perena Finance) | Insufficient track record, minimal TVL |
