# Risk & Security

Dawn Vault is an experimental DeFi product. All deposits are subject to risk.

## Smart Contract Security

### Non-Custodial Design

All vault assets are held in **Program Derived Accounts (PDAs)** controlled by the Vault Program — not by any individual or multisig wallet. Depositors can always withdraw by burning their LP tokens.

### Permission Separation

| Role | Permissions | Holder |
|------|------------|--------|
| **Admin** | Add/remove adapters, change fees, replace manager, calibrate HWM | Multisig (Squads) |
| **Manager** | Execute rebalances, harvest fees, manage positions via adapters | Manager Bot |

The Manager Bot cannot add new adapters, change fee parameters, withdraw to arbitrary addresses, or bypass adapter whitelisting.

### Adapter Whitelisting

External protocol interactions are gated through Adapter Programs. Only whitelisted adapters can access vault funds, and adding a new adapter requires Admin (multisig) approval.

### Voltr Framework

Built on **Voltr** (Ranger Finance) — battle-tested with multiple vaults in production. Includes LP token accounting, fee management, HWM tracking, and locked profit mechanism.

### Anti-MEV Protections

- **Locked Profit**: Yearn V2-style linear release prevents sandwich/frontrun attacks
- **Redemption Fee**: 0.1% makes sandwich attacks unprofitable
- **Priority Fee Management**: Critical transactions use elevated priority fees

### Audit Status

- Voltr framework has been audited
- Kamino audited by Certora, OtterSec, Sec3, Ackee
- Dawn Vault source code is [open source on GitHub](https://github.com/DawnLabsTech/vault) — anyone can review the code
- Dedicated third-party audit planned

## Risk Disclosures

### 1. Smart Contract Risk — Medium

Vault interacts with multiple on-chain programs (Voltr, Kamino adapters, Jupiter adapters). Any could contain bugs or vulnerabilities.

**Mitigations:** Audited frameworks, adapter whitelisting, non-custodial PDA architecture.

### 2. Multiply / Leverage Risk — Medium

Kamino Multiply uses leveraged stablecoin loops (2.5x-5.75x). Collateral depeg, borrow rate spikes, or high utilization could impact positions.

**Mitigations:** Multiply Risk Scorer evaluates pools on 4 dimensions (depeg risk, liquidation proximity, exit liquidity, reserve pressure). Multi-stage deleverage: target health 1.15, soft deleverage at < 1.10, emergency at < 1.05. Active positions stopped at risk score ≥ 75, fully exited at ≥ 90.

### 3. Funding Rate / Market Risk — Medium

Delta-neutral strategy depends on positive SOL funding rates. Rapid FR reversal may cause losses before positions close.

**Mitigations:** Entry requires FR > 10% for 3 days. Emergency exit at FR < -10% (no delay). Base Layer continues generating yield during FR downturns.

### 4. Liquidation Risk — Low

DN uses 1x leverage (margin = position size) — liquidation risk effectively zero. Multiply positions protected by staged deleverage.

### 5. Exchange / Counterparty Risk — Medium

Delta-neutral uses Binance for perpetual futures. Exchange insolvency, API outages, or regulatory actions could affect margin funds.

**Mitigations:** Position size limits, minimum on-chain balance maintained, multi-venue consideration for future phases.

### 6. Oracle Risk — Low-Medium

Price feeds from Pyth may be delayed, manipulated, or stale.

**Mitigations:** Multi-source aggregation, staleness checks (>2 slots → pause), deviation monitoring (>5% → halt).

### 7. Operational Risk — Low-Medium

Manager Bot is a critical off-chain component.

**Mitigations:** 5-minute health checks with auto-retry, externalized configuration, kill switch and guardrails.

### 8. Liquidity Risk — Low

Large withdrawals may require unwinding positions.

**Mitigations:** Buffer maintained (5%), redemption fee (0.1%), locked profit mechanism.

### 9. Protocol / Composability Risk — Low-Medium

Failure in any composed protocol (lending, Multiply, DEX) could cascade.

**Mitigations:** Protocol diversification (max 60% per protocol), circuit breaker (auto-exit on TVL crash, oracle drift, withdrawal failure), lending risk scorer tracks incident history.

> **Note:** Drift has been excluded due to the 2025 hack.

### 10. Regulatory Risk — Unknown

DeFi regulations are evolving. Vault operates under non-custodial architecture to reduce regulatory exposure.

### 11. Solana Network Risk

Network outages, congestion, or consensus issues could prevent operations.

**Mitigations:** Conservative leverage survives multi-hour outages, priority fee management, post-outage health checks.

## Summary Risk Matrix

| Risk | USDC Vault |
|------|-----------|
| Smart Contract | Medium |
| Multiply / Leverage | Medium |
| Market / FR | Medium |
| Liquidation | **Low** |
| Exchange | Medium |
| Oracle | Low-Med |
| Operational | Low-Med |
| Liquidity | Low |
| Protocol | Low-Med |
| Regulatory | Unknown |

## Capacity Management

Alpha strategies have limited capacity. Without caps, yield degrades as TVL grows.

| Mechanism | Description |
|-----------|-------------|
| **Hard Cap** | Maximum TVL enforced at the smart contract level |
| **Soft Cap** | Warning threshold; new deposits monitored for yield impact |
| **Dynamic Adjustment** | Caps adjusted based on market liquidity and strategy capacity |

> We prioritize quality of yield over quantity of AUM. We would rather run a $5M vault at 15% APY than a $50M vault at 6% APY.

{% hint style="warning" %}
**Please do not deposit more than you can afford to lose.** Past performance does not guarantee future results. See our [Disclaimer](../legal/disclaimer.md) for important legal information.
{% endhint %}
