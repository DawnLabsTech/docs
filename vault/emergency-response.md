# Emergency Response

This document defines Dawn Vault's emergency response framework — the policies, procedures, and communication channels for responding swiftly and appropriately when asset-impacting risks arise, such as protocol exploits, collateral crashes, or network outages.

## A. Detection

Catch anomalies as early as possible. Multiple detection layers run in parallel to eliminate single points of failure.

### A1. Protocol Anomaly Detection (Planned)

Monitor contracts of utilized protocols (Kamino, Jupiter, etc.) for abnormal large transfers or permission changes. Integration with external security feeds (Hypernative, Forta Network, etc.) is also under consideration.

### A2. Borrow Rate Spike Detection (Planned)

When Kamino Multiply borrow rates spike and exceed collateral yield (negative carry) for a sustained period, trigger an alert and initiate automatic deleverage.

### A3. Protocol Circuit Breaker (Implemented)

Automatically exit positions from a protocol when any of the following conditions are met:

| Trigger | Threshold | Action |
|---------|-----------|--------|
| TVL crash | -20% / 1 hour | Full withdrawal from affected protocol |
| Oracle deviation | ≥ 100bps | Full withdrawal from affected protocol |
| Consecutive withdrawal failures | 3 times | Disable affected protocol |

After exit, a 24-hour cooldown period applies before automatic re-enablement is attempted.

### A4. Kill Switch (Implemented)

A manual last-resort mechanism for humans to immediately halt all operations. Detected on the next health check (every 5 seconds), triggering the full emergency exit sequence for all positions.

## B. Decision

Pre-define what to do and in what order after detection.

### B1. Dependency Map

A relationship diagram of protocols, assets, and infrastructure that vault positions depend on. Makes explicit what breaks when each component fails.

```
ONyc/USDC Multiply Position
│
├── Kamino Protocol
│   ├── Smart Contract → Hack / Bug
│   ├── Multiply SDK → Loop execution failure
│   └── Market Config (RWA Market) → Parameter change / Freeze
│
├── Collateral Asset: ONyc
│   ├── Issuer: Onre → Operations halt / Redemption freeze
│   ├── Native Yield (~10.25%) → Yield decline / Stop
│   ├── Depeg → Direct liquidation risk
│   └── DEX Liquidity (ONyc/USDC) → Exit impossible
│
├── Borrowed Asset: USDC
│   ├── Oracle (Pyth/Switchboard) → Price feed anomaly
│   ├── Borrow Rate → Spike causing negative carry
│   └── USDC Depeg → NAV-wide impact
│
├── Infrastructure
│   ├── Solana RPC (Helius) → TX submission failure
│   ├── Solana Network → Congestion / Halt
│   └── Jupiter (swap route) → Swap failure during deleverage
│
└── External Contagion Scenarios
    ├── Other protocol hack → Kamino TVL crash / Rate spike
    ├── Solana-wide DeFi panic → Liquidity evaporation
    └── Stablecoin uncertainty → Impact on both USDC and ONyc
```

#### Response Table by Failure Point

| Failure Point | Impact | Detection | Response |
|--------------|--------|-----------|----------|
| Kamino contract hack | Loss of deposited assets | A1, A3 (TVL crash) | Kill switch → Notify Kamino |
| ONyc depeg (>2%) | Health rate decline → Liquidation | Multiply Risk Scorer | Staged deleverage |
| ONyc DEX liquidity vanishes | Unable to swap during deleverage | C1 (periodic simulation) | Split withdrawal, raise slippage cap |
| Onre operations halt | ONyc unredeemable, yield stops | Manual monitoring, contact Onre | Full exit |
| Borrow rate spike | Negative carry erodes NAV | A2 (spike detection) | Auto deleverage |
| Pyth oracle anomaly | Incorrect liquidation or misjudgment | A3 (deviation detection) | Circuit breaker trips |
| Helius RPC outage | TX submission failure | Guardrails (consecutive failure detection) | Failover to backup RPC |
| Jupiter swap route down | Unable to deleverage | C4 (periodic dry run) | Alternative route or wait |

### B2. Withdrawal Priority Order (Planned)

Pre-define which positions to exit first based on liquidity depth. Exit illiquid positions first while liquidity remains, rather than attempting a single bulk withdrawal.

### B3. Negative Carry Auto-Deleverage (Planned)

Automatically reduce position size when borrow rates exceed collateral yield for a sustained period.

### B4. NAV Freeze Criteria (To be implemented at Strata migration)

Define conditions under which NAV calculation is frozen and Instant Redemption is disabled. Prevents information-advantaged LPs from front-running exits, which would unfairly socialize losses onto remaining participants.

## C. Execution

Ensure positions can actually be closed after a decision is made.

### C1. Liquidity Depletion Simulation (Planned)

Periodically test whether Multiply deleverage can complete when ONyc/USDC DEX liquidity is extremely thin.

### C2. Staged Withdrawal Logic (Planned)

Split withdrawals into tranches rather than attempting a single bulk exit, executing while liquidity remains available.

### C3. TX Failure Retry Enhancement (Partially Implemented)

Automatic priority fee escalation and Jito bundle utilization during Solana network congestion.

### C4. Emergency Exit Dry Run (Planned)

Periodically execute small-amount withdrawals and re-deposits in the production environment to verify that exit routes remain functional.

## D. Communication

### D1. Internal Escalation

```
Detection (Bot auto Telegram notification)
  │
  ├─ Immediate: Telegram alert to Yutaro
  │
  ├─ Within +5 min: Initial impact assessment
  │
  └─ Within +10 min: Response decision (full exit / partial reduction / monitor)
```

### D2. External Notification Targets

| Target | When to Notify | Purpose |
|--------|---------------|---------|
| **Kamino** | Collateral, oracle, or market anomaly | Information sharing / Freeze request |
| **Jupiter** | Swap / Lending anomaly | Route verification / Status sharing |
| **Helius** | RPC anomaly / Solana outage | Infrastructure status check |
| **Onre (ONyc)** | ONyc depeg / Redemption halt | Redemption availability check |
| **LP Investors** | When NAV is impacted | Status report / Policy communication |

{% hint style="info" %}
Specific contact details (Discord, email, security contacts) for each target are maintained in internal documentation.
{% endhint %}

### D3. LP Incident Report Template

```
[ALERT] [datetime] Anomaly Detected

- What happened:
- Vault impact: NAV change / Position status
- Response status: Exited / Reducing / Monitoring
- Next update: [time]
```

## Implementation Roadmap

### Phase 1 — Immediate (Low cost, high impact)

- Collect and store contact list for D2
- Share D3 template with stakeholders
- Run C1 liquidity simulation manually once

### Phase 2 — Next Sprint (Development required)

- A2: Borrow rate spike detection
- B2: Withdrawal priority order as config values

### Phase 3 — Medium Term (Before Strata / LP expansion)

- A1: Protocol anomaly detection / External security feed integration
- B4: NAV freeze / Instant Redemption disable functionality
- C2: Staged withdrawal logic
