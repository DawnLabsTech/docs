# FAQ & Glossary

## FAQ

### General

**What is Dawn Vault?**
A yield-generating vault on Solana built by Dawn Labs, an active Solana validator operator. It combines Kamino Multiply, lending aggregation, and delta-neutral strategies for optimized risk-adjusted returns.

**What makes Dawn Vault different?**
(1) **Validator-native alpha** — dawnSOL adds staking rewards on top of strategy returns; (2) **Full transparency** — every source of yield is decomposed and disclosed; (3) **Yield Smoothing Reserve** — validator commission revenue stabilizes returns.

**Is Dawn Vault custodial?**
No. All assets are held in on-chain PDA accounts controlled by the Vault Program smart contract. No individual or multisig can arbitrarily withdraw depositor funds.

**Who operates Dawn Vault?**
Dawn Labs, a Solana operations partner based in APAC. We run validator infrastructure and operate the vault with our own capital alongside depositors.

### Yield

**Where does the yield come from?**
- **Kamino Multiply** — Leveraged stablecoin loops (ONyc/USDC ~16%, USDG/PYUSD ~9.5%)
- **Lending interest** — USDC lent to Kamino and Jupiter
- **Funding rate payments** — SOL-PERP perpetual futures (when active)
- **Staking rewards** — dawnSOL ~7% APY (when DN is active)

See [Transparency](transparency.md) for full details.

**What APY can I expect?**
9-16%+ from the Base Layer (Kamino Multiply + Lending). During high SOL funding rate periods, 25-30% when the Alpha Layer is active. Current mode (2026-04): Base Layer only.

**Is the APY guaranteed?**
No. APY depends on market conditions and is variable. The Yield Smoothing Reserve helps stabilize but does not guarantee any minimum.

**How is yield distributed?**
Auto-compounded into the vault. No claiming or harvesting. LP token share price increases as the vault earns yield.

### Deposits & Withdrawals

**What assets can I deposit?**
USDC (SPL token on Solana).

**Is there a lock-up period?**
No. Withdraw at any time.

**What are the fees?**
Performance: 20% (HWM-based), Management: 1%/year, Deposit: 0%, Withdrawal: 0.1%. See [Fees & How to Use](fees-and-usage.md).

### Risk

**Can I lose money?**
Yes. Strategies minimize risk, but loss is possible due to smart contract bugs, market events, or other factors. See [Risk & Security](risk-and-security.md).

**How is risk managed?**
Multi-layered system: Multiply Risk Scorer (4 dimensions: depeg risk, liquidation proximity, exit liquidity, reserve pressure), Lending Risk Scorer (5 dimensions), Protocol Circuit Breaker, staged deleverage protection, DN risk manager, guardrails and kill switch.

**Is the smart contract audited?**
Voltr framework and Kamino have been audited. A dedicated third-party audit of Dawn Vault's deployment is planned.

**Why was Drift excluded?**
Following the 2025 hack. All Drift code paths are deprecated.

---

## Glossary

| Term | Definition |
|------|-----------|
| **APY** | Annual Percentage Yield — annualized return including compounding |
| **Base Layer** | Always-on yield strategy (Kamino Multiply + Lending) |
| **Alpha Layer** | Conditional yield strategy (delta-neutral) — activates only during favorable conditions |
| **Circuit Breaker** | Autonomous safety mechanism: auto-exit on TVL crash, oracle drift, or withdrawal failure |
| **dawnSOL** | Dawn Labs' Liquid Staking Token — represents staked SOL with Dawn Labs' validator |
| **Delta-Neutral (DN)** | Equal long spot + short perp positions, eliminating directional exposure while earning funding rates |
| **Funding Rate (FR)** | Periodic payment between long/short traders in perpetual futures to keep perp prices aligned with spot |
| **High Water Mark (HWM)** | Highest-ever share price — performance fees only charged on new profits above HWM |
| **Kamino Multiply** | Leveraged stablecoin loop strategy on Kamino — deposit collateral, borrow, swap, re-deposit |
| **LP Token** | Liquidity Provider Token — represents a depositor's proportional share of a vault |
| **LST** | Liquid Staking Token — represents staked assets usable in DeFi while earning staking rewards |
| **Manager Bot** | Off-chain automated system executing vault strategies, managing positions, monitoring risk |
| **Market Scanner** | Monitors Kamino Multiply pool APYs and recommends switching when economics justify |
| **Multiply Risk Scorer** | 4-dimension evaluation: depeg risk (30%), liquidation proximity (30%), exit liquidity (20%), reserve pressure (20%) |
| **Lending Risk Scorer** | 5-dimension evaluation: TVL scale (30%), protocol maturity (20%), reserve utilization (25%), deposit concentration (15%), incident history (10%) |
| **PDA** | Program Derived Account — Solana account owned by a program, used for non-custodial asset custody |
| **Share Price** | Value of one LP token in the vault's base asset. Increases as the vault earns yield |
| **TVL** | Total Value Locked — total assets deposited in a vault or protocol |
| **Voltr** | Vault framework by Ranger Finance on which Dawn Vault is built |
| **Yield Provenance** | Practice of decomposing and disclosing every source of vault yield |
| **Yield Smoothing Reserve (YSR)** | Reserve funded by validator commission that stabilizes vault APY during unfavorable conditions |
