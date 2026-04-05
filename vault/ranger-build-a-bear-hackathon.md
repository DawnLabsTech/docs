# Ranger Build-A-Bear Hackathon

This page is the submission hub for Dawn Vault's Ranger Build-A-Bear Hackathon entry. It gathers the key materials judges need to review the strategy, implementation, and live build progress in one place.

## Dawn Vault in One Paragraph

Dawn Vault is a validator-native yield vault built on Ranger's Voltr vault framework and designed as a Ranger Earn strategy. Instead of relying on a single DeFi primitive, it combines a stable **Base Layer** (Kamino Multiply + lending) with a conditional **Alpha Layer** (SOL delta-neutral with dawnSOL staking) so the vault can pursue higher yield without depending on directional SOL exposure.

## Submission Materials

### 🎥 Demo / Pitch Video (max 3 min)

**Link:** `TBD`

Walkthrough of the strategy, where the edge comes from, and how the vault operates on Ranger Earn.

### 📄 Strategy Documentation

**Primary links:**

- [Overview](overview.md)
- [USDC Vault](usdc-vault.md)
- [Backtest](backtest.md)
- [Risk & Security](risk-and-security.md)
- [Transparency](transparency.md)

These pages explain the strategy thesis, portfolio construction, entry / exit logic, position sizing, drawdown controls, and rebalancing framework.

### 💻 Code Repository

**Link:** [DawnLabsTech/vault](https://github.com/DawnLabsTech/vault)

Open-source reference implementation for the vault, backtest engine, and supporting strategy logic. If any submission-specific repository remains private, organizers and judges should be granted review access as required.

> If a private repository is used, please add `@jakeyvee` on GitHub so the code can be reviewed. Submissions without verifiable code will not be considered.

### 🔗 On-chain Verification

**Wallet / Vault Address:** `TBD`

This will be the wallet address or vault address used during the hackathon build window (**Mar 9 - Apr 6**). Judges can verify trade activity and performance directly on-chain via Solscan.

## What This Submission Is Showing

- A validator-native yield strategy rather than a pure DeFi aggregator
- A two-layer architecture with always-on base yield and conditional alpha activation
- Rule-based risk management instead of discretionary portfolio management
- A realistic path from hackathon build to production vault operations

## Judging Criteria

### Strategy Quality & Edge

- Validator infrastructure is part of the yield stack, creating a source of edge that pure aggregators do not have
- The vault combines stable base carry with conditional alpha instead of forcing one strategy across all market regimes
- The delta-neutral layer targets funding rate income while keeping net directional SOL exposure near zero
- Please evaluate whether this structure is meaningfully differentiated and defensible over time

### Risk Management

- Multiply allocation is gated by a dedicated risk scorer covering depeg risk, liquidation proximity, exit liquidity, and reserve pressure
- Lending exposure is diversified and monitored with protocol-level circuit breakers
- The delta-neutral layer uses explicit entry, exit, and emergency exit thresholds based on funding conditions
- Please evaluate the practicality of the drawdown controls, position sizing rules, and rebalancing discipline

### Technical Implementation

- The vault is built on Ranger's vault framework with a clear separation between vault program, adapters, and manager logic
- Assets remain in PDA-controlled vault accounts with adapter whitelisting and role separation
- The implementation includes backtest tooling, documented strategy logic, and production-oriented operational controls
- Please evaluate code quality, adapter integration design, vault architecture, and security assumptions

### Production Viability

- The strategy is designed around existing Solana liquidity venues and operationally realistic execution paths
- The base layer can continue to earn while the alpha layer is inactive, which supports more stable deployment behavior
- Capacity, monitoring, and operational safeguards are considered from the beginning rather than treated as future work
- Please evaluate whether the vault has a credible path from hackathon prototype to live managed product

### Novelty & Innovation

- The submission connects validator economics, LST yield, DeFi carry, and automated vault logic into one product
- It frames the vault as infrastructure-first yield, not just APY routing
- The Yield Smoothing Reserve concept extends validator revenue into yield stabilization for depositors
- Please evaluate whether these design choices contribute something new to the Ranger and Solana ecosystem
