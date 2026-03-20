# Kamino LST Multiple

A looping strategy that uses Kamino (K-Lend) and Sanctum to leverage Dawn Labs' LST (dawnSOL) for amplified staking yield. Kamino's Multiply feature enables one-click leveraged position construction.

## How It Works

1. Stake SOL into dawnSOL (via Sanctum)
2. Deposit dawnSOL as collateral on Kamino
3. Borrow SOL, convert back to dawnSOL, and deposit again (Looping)
4. Multiply feature enables up to ~7x leverage

Kamino's Multiply feature uses Flash Loans to build the entire leveraged position in a single transaction:

1. Flash Loan borrows SOL without collateral (fee: 0.001%)
2. Convert borrowed SOL to dawnSOL (via Sanctum)
3. Deposit dawnSOL as collateral on K-Lend
4. Borrow SOL against the new collateral
5. Repay Flash Loan with borrowed SOL — position complete

The entire process executes with one click on the UI.

## Features

| | |
|---|---|
| **Collateral** | dawnSOL (LST) |
| **Borrow Asset** | SOL |
| **Max Leverage** | ~10x (with eMode) |
| **Operation** | One-click (Kamino Multiply) |

- One-click leveraged position construction via Kamino Multiply
- Maintain liquidity as an LST while earning amplified yield
- Built-in risk management features (auto-deleverage, etc.)

### eMode (Elevation Mode)

eMode raises the LTV cap for correlated asset pairs (SOL and LST), enabling higher leverage.

| Mode | LTV Cap | Max Leverage |
|------|---------|--------------|
| Standard | 75% | ~4x |
| eMode | 90% | ~10x |

Since SOL and LST are fundamentally the same asset with correlated pricing, eMode application is justified.

## Risks

### Borrow Rate Volatility

**Risk**: SOL borrow rates fluctuate with market conditions. If the borrow rate exceeds staking yield, returns decrease. Prolonged periods could push the collateral ratio below the liquidation threshold, resulting in principal loss.

**Mitigation**: Kamino's SOL borrow market has deep liquidity and rates are generally stable. Since this is a SOL-to-SOL borrow structure, liquidation can only occur if borrow rates significantly exceed staking yield for an extended period — a scenario that has never occurred historically.

### Smart Contract Risk

**Risk**: Vulnerabilities may exist in the DeFi protocol's smart contracts.

**Mitigation**: Kamino has completed multiple independent security audits and is a major protocol in the Solana ecosystem with a long operational track record. A bug bounty program is continuously maintained.

**Audit History (K-Lend)** ([full list](https://github.com/Kamino-Finance/audits)):
- [Certora](https://github.com/Kamino-Finance/audits/blob/master/kamino_lend_certora.pdf) — Formal verification proving mathematical safety
- OtterSec — Solana-specialized security audit
- [Sec3](https://github.com/Kamino-Finance/audits/blob/master/kamino_klend_sec3.pdf) — Automated Solana program security analysis
- Ackee Blockchain — Fuzz testing security verification
- [OSec](https://github.com/Kamino-Finance/audits/blob/master/kamino_lend_osec_formal_verification.pdf) — Formal verification
- RX Auditors — Security audit

**Bug Bounty**: Up to [$1.5M reward program on Immunefi](https://immunefi.com/bug-bounty/kamino)

### LST Depeg Risk

**Risk**: If dawnSOL's market price deviates from its theoretical price (depeg), collateral value drops and liquidation risk increases.

**Mitigation**: dawnSOL is issued through Sanctum and can be redeemed for SOL at theoretical price at any time. Sanctum is one of Solana's largest LST infrastructure providers with a stable operational track record. Kamino's auto-deleverage feature further mitigates risk during collateral ratio deterioration.

## Risk Management

Kamino provides multiple built-in risk management features:

- **Auto-Deleverage**: Automatically unwinds collateral and debt when LTV deteriorates, maintaining safe levels without manual intervention
- **Partial Liquidation**: When LTV exceeds the threshold, only ~2% of collateral is liquidated — unlike full liquidation in typical protocols, minimizing impact on principal
- **KRAF (Kamino Risk Assessment Framework)**: Comprehensive risk assessment framework performing volatility measurement, stress testing, and real-time monitoring
