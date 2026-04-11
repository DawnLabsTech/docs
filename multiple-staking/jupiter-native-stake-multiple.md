# Jupiter Native Stake Multiple

A looping strategy that uses Jupiter Lend to borrow SOL against natively staked SOL with the Dawn Labs validator, then restakes the borrowed SOL to amplify staking yield.

## How to Use

### 1. Native Stake SOL with the DawnLabs Validator

Stake SOL to the Dawn Labs validator using native staking from your wallet:

- **Phantom**: [Stake SOL in Phantom with native staking](https://help.phantom.com/hc/en-us/articles/4406374138771-Stake-SOL-in-Phantom-with-native-staking)
- **Solflare**: [Native Solana (SOL) Staking on Solflare: A Step-by-Step Guide](https://help.solflare.com/en/articles/9264008-native-solana-sol-staking-on-solflare-a-step-by-step-guide)
- **Ledger**: [Stake SOL with Ledger](https://www.ledger.com/staking/ledger-node/solana)

### 2. Wait for Your Stake to Become Active

Native stake activation takes 1 epoch (~2–3 days). Wait until your stake status shows **Active** before proceeding.

### 3. Deposit nsDawn as Collateral on Jupiter Lend

Once your stake is active, it is tokenized as **nsDawn** (an nsToken representing your native stake position). Go to [Jupiter Lend](https://jup.ag/lend/borrow/73) and deposit the desired amount of nsDawn as collateral.

### 4. Borrow SOL Against Your Collateral

On Jupiter Lend, borrow SOL using your deposited nsDawn as collateral.

### 5. Restake Borrowed SOL and Repeat

Return to step 1 and native stake the borrowed SOL with the Dawn Labs validator again. By repeating this loop, you can achieve up to **~7x leverage** on your staking yield.

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
