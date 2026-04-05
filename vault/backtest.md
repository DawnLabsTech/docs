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
| Multiply APY | 13% (fixed) |
| Lending APY | 5% (fixed) |
| dawnSOL Staking APY | 6.8% |
| DN Allocation Cap | 70% of NAV |
| FR Entry Threshold | > 10% annualized for 3 days |
| FR Exit Threshold | < 0% annualized for 3 days |
| FR Emergency Exit | < -10% annualized |

## Results: Current Strategy

Multiply-first + Lending overflow + conditional DN — the live production configuration.

| Metric | Value |
|--------|-------|
| **Final NAV** | $13,439.01 |
| **Total Return** | 34.39% |
| **Annualized Return** | 14.04% |
| **Sharpe Ratio** | 27.01 |
| **Max Drawdown** | 0.23% |
| Days in BASE_ONLY | 584 |
| Days in BASE_DN | 237 |
| DN Entries / Exits | 3 / 3 |

### Yield Breakdown

| Source | Amount | Share |
|--------|-------:|------:|
| Multiply Yield | $2,784.87 | 79.1% |
| Funding Rate Income | $535.38 | 15.2% |
| dawnSOL Staking | $200.22 | 5.7% |
| **Total Gross** | **$3,520.47** | **100%** |
| Fees (deducted) | -$69.86 | |
| **Net Profit** | **$3,439.01** | |

Multiply generates the vast majority of returns. The DN layer contributes a smaller but consistent positive amount across 3 entry/exit cycles ($736 gross from funding + staking).

### DN Active Periods

| # | Period | Days | Avg FR (annualized) |
|---|--------|-----:|--------------------:|
| 1 | 2024-01-03 — 2024-07-05 | 185 | 18.4% |
| 2 | 2024-11-11 — 2024-12-20 | 40 | 20.0% |
| 3 | 2025-07-22 — 2025-08-02 | 12 | 6.4% |
| **Total** | | **237 (28.9%)** | **18.0%** |

### Benchmarks

| Strategy | Total Return |
|----------|------------:|
| **Current (Multiply + Lending + DN)** | **34.39%** |
| Multiply Only | 31.64% |
| Lending Only | 11.60% |
| SOL Buy & Hold | -18.31% |

## Limitations

- **Fixed APY assumption**: Multiply and Lending APYs are held constant. Real rates fluctuate.
- **No depeg / liquidity modeling**: Multiply collateral depeg risk and exit liquidity constraints are not simulated.
- **Fee model**: Binance taker 0.04% + swap slippage 0.1% + Solana gas + $1 withdrawal fee.
- **Data source**: Binance SOL-PERP 8-hour funding rates only.
- **Sharpe Ratio caveat**: The high Sharpe (27.0) is partly an artifact of fixed-APY inputs. Expect lower values in live operation.

{% hint style="warning" %}
Backtest results do not guarantee future performance. Past market conditions may not repeat. See [Risk & Security](risk-and-security.md) and [Disclaimer](../legal/disclaimer.md).
{% endhint %}

## Run It Yourself

The backtest engine is open source. You can reproduce these results or test your own parameters.

### Setup

```bash
git clone https://github.com/DawnLabsTech/vault.git
cd vault/backtest
npm install
```

### Basic Usage

```bash
# Run with default parameters (matches live bot config)
npm run backtest

# Show all available options
npm run backtest -- --help
```

### CLI Options

| Flag | Description | Default |
|------|-------------|---------|
| `--start` | Start date | 2021-01-01 |
| `--end` | End date | 2026-03-01 |
| `--capital` | Initial capital (USDC) | 10000 |
| `--multiply-apy` | Fixed Multiply APY (%) | 13 |
| `--multiply-cap` | Max USDC in Multiply | unlimited |
| `--lending-apy` | Fixed Lending APY (%) | 5 |
| `--dawnsol-apy` | Fixed dawnSOL APY (%) | 6.8 |
| `--entry-fr` | FR entry threshold (%) | 10 |
| `--exit-fr` | FR exit threshold (%) | 0 |
| `--emergency-fr` | Emergency exit threshold (%) | -10 |
| `--confirm-days` | Confirmation days | 3 |
| `--dn-alloc` | DN allocation ratio | 0.7 |
| `--output` | Output format: `table` / `csv` / `json` | table |
| `--fetch-only` | Fetch data only, skip simulation | — |

### Examples

```bash
# Reproduce the published results
npm run backtest -- --start 2024-01-01 --end 2026-04-01

# Export as JSON for further analysis
npm run backtest -- --output json

# Fetch funding rate data without running simulation
npm run backtest -- --fetch-only --start 2020-01-01 --end 2026-04-01
```

Funding rate data is fetched from Binance on the first run and cached locally in SQLite. Subsequent runs reuse the cache.
