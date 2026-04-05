# Roadmap

## Phase 1: USDC Vault (Current)

**Status: Active**

The flagship vault establishing Dawn Vault's core infrastructure and track record.

- **Base Layer**: Kamino Multiply (ONyc/USDC ~16%, USDG/PYUSD ~9.5%) + USDC Lending (Kamino, Jupiter, 3-8%)
- **Alpha Layer**: SOL delta-neutral with dawnSOL enhancement (15-20% APY)
- **Target APY**: 9-16%+

### Milestones
- [x] Strategy backtest (821 days, Sharpe Ratio 31.50)
- [x] Manager Bot development
- [x] Live deployment with own capital
- [ ] Public deposits
- [ ] Performance reporting system
- [ ] Documentation site

---

## Phase 2: SOL Vault

**Status: Planned**

| Parameter | Value |
|-----------|-------|
| **Base Layer** | Validator Staking (6-7%) |
| **Alpha Layer** | LST Loop (10-20%) |
| **Rebalance Freq** | Monthly (swap cost sensitive) |
| **Decision Metric** | LST yield - SOL borrow rate spread |

---

## Phase 3: BTC Vault

**Status: Planned**

| Parameter | Value |
|-----------|-------|
| **Base Layer** | cbBTC Lending (1-3%) |
| **Alpha Layer** | cbBTC collateral → USDC borrow → SOL DN (3.5-11%) |
| **Rebalance Freq** | Weekly-Monthly (LTV management) |
| **Decision Metric** | SOL FR + USDC borrow cost + BTC price |

---

## Future Considerations

- **JPY stablecoin integration** for Japanese market access
- **Multi-venue CEX support** to reduce single-exchange risk
- **On-chain attestation** of off-chain positions
- **Expanded consulting services** for enterprise validator operations

---

*Timelines are indicative and subject to change based on market conditions and development progress.*
