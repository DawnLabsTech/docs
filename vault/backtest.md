# Backtest

Backtesting validates the USDC Vault's strategy using historical market data before deploying real capital.

## Methodology

- **Data**: Binance SOL-PERP 8-hour funding rates + OHLCV
- **Period**: January 2024 — April 2026 (821 days)
- **Initial Capital**: $10,000
- **Assumptions**: Multiply and Lending APYs are held constant (see [Limitations](#limitations)). DN entry/exit uses the same parameters as the live bot.

### Parameters

| Parameter | Value |
|-----------|-------|
| Multiply APY | 16% (fixed) |
| Lending APY | 5% (fixed) |
| dawnSOL Staking APY | 7% |
| DN Allocation Cap | 70% of NAV |
| FR Entry Threshold | > 10% annualized for 3 days |
| FR Exit Threshold | < 0% annualized for 3 days |
| FR Emergency Exit | < -10% annualized |

## Results: Current Strategy

Multiply-first + Lending overflow + conditional DN — the live production configuration.

| Metric | Value |
|--------|-------|
| **Final NAV** | $14,189.68 |
| **Total Return** | 41.90% |
| **Annualized Return** | 16.83% |
| **Sharpe Ratio** | 31.50 |
| **Max Drawdown** | 0.22% |
| Days in BASE_ONLY | 584 |
| Days in BASE_DN | 237 |
| DN Entries / Exits | 3 / 3 |

### Yield Breakdown

| Source | Amount | Share |
|--------|-------:|------:|
| Multiply Yield | $3,528.52 | 82.6% |
| Funding Rate Income | $537.20 | 12.6% |
| dawnSOL Staking | $207.09 | 4.8% |
| **Total Gross** | **$4,272.81** | **100%** |
| Fees (deducted) | -$71.06 | |
| **Net Profit** | **$4,189.68** | |

Multiply generates the vast majority of returns. The DN layer contributes a smaller but consistent positive amount across 3 entry/exit cycles ($744 gross from funding + staking).

### Benchmarks

| Strategy | Total Return | Annualized |
|----------|------------:|----------:|
| **Current (Multiply + Lending + DN)** | **41.90%** | **16.83%** |
| Multiply Only | 39.63% | — |
| Lending Only | 11.60% | — |
| SOL Buy & Hold | -18.31% | — |

## Scenario: Multiply Capacity Cap ($5,000)

When Multiply capacity is limited to $5,000, excess capital overflows to Lending. This reduces return but improves protocol diversification.

| Metric | Value |
|--------|-------|
| **Final NAV** | $12,819.77 |
| **Total Return** | 28.20% |
| **Annualized Return** | 11.68% |
| **Sharpe Ratio** | 21.68 |
| **Max Drawdown** | 0.22% |
| Multiply Yield | $1,612.76 |
| Lending Interest | $547.73 |
| Funding Rate Income | $534.58 |
| dawnSOL Staking | $205.77 |

## Scenario: Legacy Strategy (Lending + DN Only)

No Multiply — all base capital goes to Lending. This was the pre-Multiply strategy.

| Metric | Value |
|--------|-------|
| **Final NAV** | $11,650.49 |
| **Total Return** | 16.50% |
| **Annualized Return** | 7.03% |
| **Sharpe Ratio** | 12.99 |
| **Max Drawdown** | 0.26% |
| Lending Interest | $994.24 |
| Funding Rate Income | $530.85 |
| dawnSOL Staking | $204.02 |

## Strategy Comparison

| Metric | Current | Capacity Cap | Legacy (Lending+DN) |
|--------|--------:|------------:|-------------------:|
| Total Return | 41.90% | 28.20% | 16.50% |
| Annualized Return | 16.83% | 11.68% | 7.03% |
| Sharpe Ratio | 31.50 | 21.68 | 12.99 |
| Max Drawdown | 0.22% | 0.22% | 0.26% |
| Multiply Yield | $3,529 | $1,613 | $0 |
| Lending Interest | $0 | $548 | $994 |
| DN Income (FR + Staking) | $744 | $740 | $735 |

Adding Multiply improved annualized return by +9.8 percentage points (7.03% → 16.83%) with better risk-adjusted metrics.

## Limitations

- **Fixed APY assumption**: Multiply and Lending APYs are held constant. Real rates fluctuate.
- **No depeg / liquidity modeling**: Multiply collateral depeg risk and exit liquidity constraints are not simulated.
- **Fee model**: Binance taker 0.04% + swap slippage 0.1% + Solana gas + $1 withdrawal fee.
- **Data source**: Binance SOL-PERP 8-hour funding rates only.
- **Sharpe Ratio caveat**: The high Sharpe (31.5) is partly an artifact of fixed-APY inputs. Expect lower values in live operation.

{% hint style="warning" %}
Backtest results do not guarantee future performance. Past market conditions may not repeat. See [Risk & Security](risk-and-security.md) and [Disclaimer](../legal/disclaimer.md).
{% endhint %}
