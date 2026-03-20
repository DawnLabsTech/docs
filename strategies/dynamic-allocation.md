# Dynamic Allocation

Dynamic allocation is the core decision-making logic that determines how vault assets are distributed between the Base Layer and Alpha Layer.

## Design Philosophy

> Every Dawn Vault follows one rule: **yield should never drop to zero.**

The Base Layer (lending or staking) runs continuously. The Alpha Layer only activates when market conditions make it profitable. This two-layer design means depositors always earn something, with upside capture during favorable periods.

## How It Works

```mermaid
graph TB
    M[Manager Bot] --> MON[Monitor Market Signals]
    MON --> FR{Check<br>Conditions}
    FR -->|Favorable| INC[Increase Alpha Allocation]
    FR -->|Neutral| HOLD[Maintain Current Split]
    FR -->|Unfavorable| DEC[Decrease Alpha Allocation]
    INC --> EXEC[Execute Rebalance]
    HOLD --> WAIT[Continue Monitoring]
    DEC --> EXEC
    EXEC --> VP[Update Vault Positions]
```

### Decision Inputs

| Signal | Description |
|--------|-------------|
| **Primary** | SOL Funding Rate |
| **Secondary** | Lending rates across protocols |

## USDC Vault Allocation Model

The USDC Vault uses a **state machine** with two states:

```
BASE_ONLY ←→ BASE_PLUS_DN
```

### State Transitions

| From | To | Condition |
|------|----|-----------|
| BASE_ONLY | BASE_PLUS_DN | SOL FR > 15% annualized for 2 consecutive days |
| BASE_PLUS_DN | BASE_ONLY | SOL FR < -2% annualized for 1 day |
| BASE_PLUS_DN | EMERGENCY_EXIT | SOL FR < -10% annualized (no time condition) |

### Allocation Ranges

| State | Lending | Delta-Neutral | Liquidity Buffer |
|-------|---------|---------------|-----------------|
| BASE_ONLY | 100% | 0% | Included in lending |
| BASE_PLUS_DN | 30–50% | 50–70% | 30% minimum maintained |

## Parameter Optimization

All threshold parameters are:

- **Backtested** against 5.5 years of historical data
- **Stress-tested** against major market events
- **Externalized** as vault configuration (adjustable without code changes)
- **Continuously updated** based on live performance data

See [USDC Vault — Performance](../vaults/usdc-vault.md#performance) for backtest methodology and results.
