# Ranger Build-A-Bear Hackathon

This page is the submission hub for Dawn Vault's Ranger Build-A-Bear Hackathon entry. It gathers the key materials judges need to review the strategy, implementation, and live build progress in one place.

## Dawn Vault in One Paragraph

Dawn Vault is a validator-native yield strategy being developed for Ranger Earn. The current implementation focuses on strategy logic, risk management, and backtest tooling, with future on-chain deployment planned to integrate Ranger's vault framework and protocol adapters. Instead of relying on a single DeFi primitive, it combines a stable **Base Layer** (Kamino Multiply + lending) with a conditional **Alpha Layer** (SOL delta-neutral with dawnSOL staking) so the vault can pursue higher yield without depending on directional SOL exposure.

## Submission Materials

### 🎥 Demo / Pitch Video (max 3 min)

**Link:** `TBD`

### 📄 Strategy Documentation

- [Overview](overview.md)
- [USDC Vault](usdc-vault.md)
- [Backtest](backtest.md)
- [AI Support](ai-support.md)
- [Risk & Security](risk-and-security.md)
- [Transparency](transparency.md)

### 📊 Live Dashboard

**Link:** [Dawn Vault Dashboard](https://frontend-sand-six-69.vercel.app/)

### 💻 Code Repository

**Link:** [DawnLabsTech/vault](https://github.com/DawnLabsTech/vault)

### 🔗 On-chain Verification

**Wallet / Vault Address:** `3S1kvQuLYLisn8KxHNiCoqTgtZNJQAxJi4oyGLxeeYVT`

- [View on Solscan](https://solscan.io/account/3S1kvQuLYLisn8KxHNiCoqTgtZNJQAxJi4oyGLxeeYVT)

## Judging Criteria

### Strategy Quality & Edge

- Validator infrastructure is part of the yield stack, creating a source of edge that pure aggregators do not have
- The vault combines stable base carry with conditional alpha instead of forcing one strategy across all market regimes
- The delta-neutral layer targets funding rate income while keeping net directional SOL exposure near zero

### Risk Management

- Multiply allocation is gated by a dedicated risk scorer covering depeg risk, liquidation proximity, exit liquidity, and reserve pressure
- Lending exposure is diversified and monitored with protocol-level circuit breakers
- The delta-neutral layer uses explicit entry, exit, and emergency exit thresholds based on funding conditions

### Technical Implementation

- The current implementation centers on strategy logic, backtest tooling, monitoring, and production-oriented operational controls
- The operator stack includes AI Advisor and Vault AI Chat for read-only monitoring, explanation, and support workflows
- This submission validates the allocation logic, risk controls, and execution design before vault-framework integration
- Ranger vault framework integration and protocol adapters are planned as the next step toward production deployment

### Production Viability

- The strategy is designed around existing Solana liquidity venues and operationally realistic execution paths
- The base layer can continue to earn while the alpha layer is inactive, which supports more stable deployment behavior
- Capacity, monitoring, and operational safeguards are considered from the beginning rather than treated as future work

### Novelty & Innovation

- The submission connects validator economics, LST yield, DeFi carry, and automated vault logic into one product
- It adds an AI support layer for operator-facing recommendations, vault-state interpretation, and scenario analysis without execution authority
- It frames the vault as infrastructure-first yield, not just APY routing
- The Yield Smoothing Reserve concept extends validator revenue into yield stabilization for depositors
