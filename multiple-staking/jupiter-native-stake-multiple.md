# Jupiter Native Stake Multiple

A looping strategy that uses Jupiter Lend to borrow SOL against natively staked SOL with the Dawn Labs validator, then restakes the borrowed SOL to amplify staking yield.

## How It Works

1. Stake SOL with the Dawn Labs validator
2. Tokenize staked SOL into nsToken (via Solana's official Single Pool Program)
3. Deposit nsToken as collateral on Jupiter Lend Vault
4. Borrow SOL against the collateral
5. Restake borrowed SOL with the Dawn Labs validator
6. Repeat to achieve up to ~7x leverage

Staking rewards continue to accrue even after the staked SOL is used as collateral, since the stake state is maintained throughout the borrowing structure.

## Features

| | |
|---|---|
| **Collateral** | Natively staked SOL (nsToken) |
| **Borrow Asset** | SOL |
| **Max Leverage** | ~7x |
| **LST Price Risk** | None (native stake, not LST) |
| **Operation** | Manual (Jupiter is developing automated management) |

- No LST price deviation risk since native stake is used directly
- Collateral and borrowed asset are both SOL — no USD price-driven liquidation risk
- Borrow rate follows a utilization-based variable rate model, dampening sudden rate spikes

## Risks

### Borrow Rate Volatility

**Risk**: SOL borrow rates fluctuate with market conditions. If the borrow rate exceeds staking yield, returns decrease. Prolonged periods could push the collateral ratio below the liquidation threshold, resulting in principal loss.

**Mitigation**: Jupiter's SOL borrow market has deep liquidity and rates are generally stable. Since this is a SOL-to-SOL borrow structure, liquidation can only occur if borrow rates significantly exceed staking yield for an extended period — a scenario that has never occurred historically.

### Smart Contract Risk

**Risk**: Vulnerabilities may exist in the DeFi protocol's smart contracts.

**Mitigation**: Jupiter has completed multiple independent security audits and is one of the largest protocols in the Solana ecosystem (integrated with 500+ projects) with a long operational track record. A bug bounty program is continuously maintained.

**Audit History** ([full list](https://dev.jup.ag/resources/audits)):
- [OtterSec](https://dev.jup.ag/resources/audits) — 4 audits (latest: November 2025)
- [Offside Labs](https://dev.jup.ag/resources/audits) — v6 audit (October 2025, April 2024), Oracle & Flashloan audit (October 2025)
- [Zenith](https://dev.jup.ag/resources/audits) — June–July 2025, all findings resolved
- [Sec3](https://dev.jup.ag/resources/audits) — v3 audit
- [Code4rena](https://code4rena.com/audits/2026-02-jupiter-lend) — Competitive audit contest ($107K scope, February–March 2026)

### Liquidity Risk

**Risk**: Unstaking native SOL requires a cooldown period (~2–3 days). Immediate liquidity needs may not be met.

**Mitigation**: Solana's cooldown period is short compared to other chains (Ethereum: several days to weeks).

### Manual Management Risk

**Risk**: Looping currently requires manual operations, adding position management overhead.

**Mitigation**: Dawn Labs provides support for position setup and management. Jupiter has planned development of automated looping management features, which will significantly reduce operational burden.
