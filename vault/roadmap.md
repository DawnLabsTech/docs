# Roadmap

## Phase 1: USDC Vault (Current)

**Status: Active**

The flagship vault establishing Dawn Vault's core infrastructure and track record.

- **Base Layer**: Kamino Multiply (ONyc/USDC ~16%, USDG/PYUSD ~9.5%) + USDC Lending (Kamino, Jupiter, 3-8%)
- **Alpha Layer**: SOL delta-neutral with dawnSOL enhancement (15-20% APY)
- **Target APY**: 9-16%+

### Milestones
- [x] Strategy backtest (821 days, Sharpe Ratio 27.01)
- [x] Manager Bot development
- [x] Live deployment with own capital
- [ ] Public deposits
- [ ] Performance reporting system
- [ ] Documentation site

---

## Phase 2: Strata — Tranched Vault

**Status: Planned**

Split the USDC Vault into Senior and Junior tranches, enabling institutional investors to participate with defined risk/return profiles.

| | Senior Vault | Junior Vault |
|---|---|---|
| **Return** | 8% fixed | Variable (residual upside) |
| **Loss priority** | Protected (last to absorb) | First-loss buffer |
| **Withdrawal** | Instant | 7-day lock |
| **Target** | Institutions, stable-yield seekers | High-yield seekers |

Underlying strategy remains the same as Phase 1. Risk separation is enforced via an on-chain Accounting Waterfall (Anchor program) built on Ranger Finance infrastructure.

> *"Stop losing sleep over hacks. Take the Senior tranche."*

→ [Detailed Design: strata.md](strata.md)

---

## Phase 3: Japan Institutional Access

**Status: Planned**

Expand access to Japanese institutional investors and qualified investors (適格投資家) through regulatory alignment and JPY-native on-ramps.

- **Regulatory & audit compliance**: Structured reporting and audit trails meeting Japanese financial regulations
- **JPY stablecoin integration**: Enable vault participation via JPY stablecoins (e.g., JPYC, progmat coin) to remove FX friction for domestic investors
- **KYC/AML layer**: Permissioned deposit flow for compliance with Japanese fund regulations
- **Localized reporting**: Performance reports and risk disclosures in Japanese, aligned with domestic institutional requirements

---

*Timelines are indicative and subject to change based on market conditions and development progress.*
