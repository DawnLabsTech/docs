# Emergency Response

This document defines Dawn Vault's emergency response framework — the policies, procedures, and communication channels for responding swiftly and appropriately when asset-impacting risks arise, such as protocol exploits, collateral crashes, or network outages.

## A. Detection

Catch anomalies as early as possible. Multiple detection layers run in parallel to eliminate single points of failure.

### A1. Protocol Anomaly Detection (Planned)

Monitor contracts of utilized protocols (Kamino, Jupiter, etc.) for abnormal large transfers or permission changes. Integration with external security feeds (Hypernative, Forta Network, etc.) is also under consideration.

### A2. Borrow Rate Spike Detection (Implemented)

Borrow rates are sampled every 5 minutes and evaluated against three independent conditions. Each condition emits an alert and feeds the deleverage policy (see B3).

| Condition | Default Threshold | Severity | Resulting Action |
|-----------|-------------------|----------|------------------|
| Negative spread (effective APY below threshold) | `effective APY < 0` | Critical | Full emergency deleverage |
| Rate of change (1-hour window) | `+500 bps / hour` | Warning | Soft deleverage (-20% size) |
| Absolute borrow APY | `> 20% annualized` | Warning | Soft deleverage (-20% size) |

**How it works.** Each Multiply position's APY breakdown (base borrow / base supply / effective / native yield / leverage) is recorded into a SQLite `borrow_rate_history` table at the same 5-minute cadence as the Multiply health check, deduplicated by 5-minute window. On every sample, the monitor evaluates the three conditions in order of severity (negative spread first) and returns at most one alert. The 1-hour rate-of-change is computed from the oldest sample within the trailing hour vs. the newest sample. Duplicate alerts are suppressed by a 30-minute cooldown per `(label, severity)` key, so the same warning will not re-fire within the cooldown window even while the condition persists; critical alerts always fire when re-detected. Samples older than `sampleRetentionDays` (default 7) are pruned. Thresholds and retention are configurable under `borrowRateSpike` in `config/default.json`.

Detection currently fires on the latest single sample; a "sustained for N minutes" gate is not yet implemented and may be added to suppress one-off transient spikes.

### A3. Protocol Circuit Breaker (Implemented)

Automatically exit positions from a protocol when any of the following conditions are met:

| Trigger | Threshold | Action |
|---------|-----------|--------|
| TVL crash | -20% / 1 hour | Full withdrawal from affected protocol |
| Oracle deviation (warning) | ≥ 50bps | Alert only (no auto-exit) |
| Oracle deviation (critical) | ≥ 100bps | Full withdrawal from **all** active protocols |
| Consecutive balance-check failures | 3 times | Disable affected protocol |

After exit, a 24-hour cooldown period applies before automatic re-enablement is attempted.

**How it works.** A scheduled task runs every 60 seconds across all active lending protocols (currently Kamino and Jupiter Lend) and performs three checks per cycle:

1. **TVL** — fetched from the Kamino market metrics endpoint (or Jupiter's earn-tokens endpoint as a proxy). Each cycle's TVL is appended to a rolling history bounded by the 1-hour window; if the oldest sample in-window is ≥20% above the latest, the protocol is tripped.
2. **Withdrawal health** — the protocol's `getBalance()` is called as a liveness probe. Each failure increments a per-protocol counter; success resets it. Three consecutive failures trip the protocol on the assumption that withdrawals are unlikely to succeed either.
3. **USDC oracle** — USDC price is fetched from the Jupiter Price API and compared to $1.00. ≥50bps deviation emits a warning alert; ≥100bps trips every active protocol simultaneously, since a USDC depeg breaks the borrow leg of every position.

When tripped, the breaker invokes an `onTrip` callback (wired by the orchestrator to the protocol's emergency withdrawal), records an `ALERT` event, and adds the protocol to a disabled set so subsequent cycles skip it. The 24-hour cooldown is checked on every cycle: once elapsed, the protocol is removed from the disabled set, the failure counter is cleared, and monitoring resumes. A manual `enableProtocol()` override is also available for operator intervention.

### A4. Kill Switch (Implemented)

A manual last-resort mechanism for humans to immediately halt all operations. Detected on the next health check (every 5 seconds), triggering the full emergency exit sequence for all positions.

**How it works.** The bot's 5-second health-check loop calls `checkKillSwitch()` first, before any other guardrail. The check is a simple file existence test against `/tmp/vault-kill` (overridable via the `VAULT_KILL_SWITCH_PATH` env var) — operators activate it by SSH-ing into the host and running `touch /tmp/vault-kill`. When the file is detected: a critical alert is fired, an `EMERGENCY_EXIT` action with `reason: 'kill_switch'` is dispatched (which unwinds the base delta-neutral position), the orchestrator stops scheduled tasks, and the process exits with code 1 so the supervisor (Docker / systemd) does not auto-restart. A file-based trigger was chosen over an API endpoint or env var so the path is reachable even when the bot's internal state machine or HTTP server is wedged.

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

### B3. Negative Carry Auto-Deleverage (Implemented)

Borrow rate spikes are routed through the same staged deleverage policy used for health and risk-score breaches, prioritized by severity:

- **Critical (negative spread)** — when effective APY drops below the threshold (default 0, i.e. borrow yield exceeds collateral yield), the position is fully unwound via `emergencyDeleverage()`.
- **Warning (rate of change / absolute threshold)** — position size is reduced by 20%; if a health-rate or risk-score reduction is also active, the largest of the three reductions is taken (not summed).

**How it works.** On every Multiply health-check cycle, the orchestrator (a) records the latest borrow/supply APY into the borrow-rate history, (b) asks the spike monitor for a current alert level, and (c) feeds that level alongside health rate and risk score into a single policy function (`determineMultiplyRiskAction`). The policy resolves to one of three outcomes:

- `emergency` if any input is at critical level (borrow-rate spike is critical *or* health < emergency threshold *or* risk score ≥ emergency score) — full `emergencyDeleverage()` is dispatched.
- `reduce` if any input is at warning level — three candidate reduction amounts are computed (20% for borrow-rate warning, 20% for health < alert, `currentBalance - riskCap` for risk-score breach) and the **maximum** of the three is taken as the actual reduction. This avoids over-reducing when multiple warnings overlap (a 20% borrow-rate cut plus a 20% health cut should not stack to 40%).
- `none` otherwise.

The `reason` attached to the action records which input dominated, so the alert message and event log can attribute the action correctly. Borrow-rate-driven actions are tagged `borrow_rate_spike_emergency` / `borrow_rate_spike_soft`.

Caveat: detection is based on the latest sample with a 30-minute alert cooldown. A sustained-period gate (e.g. negative carry held for N consecutive samples) is not yet implemented — see A2.

### B4. NAV Freeze Criteria (To be implemented at Strata migration)

Define conditions under which NAV calculation is frozen and Instant Redemption is disabled. Prevents information-advantaged LPs from front-running exits, which would unfairly socialize losses onto remaining participants.

## C. Execution

Ensure positions can actually be closed after a decision is made.

### C1. Liquidity Depletion Simulation (Planned)

Periodically test whether Multiply deleverage can complete when ONyc/USDC DEX liquidity is extremely thin.

### C2. Staged Withdrawal Logic (Planned)

Split withdrawals into tranches rather than attempting a single bulk exit, executing while liquidity remains available.

### C3. TX Failure Retry Enhancement (Partially Implemented)

Automatic priority fee escalation during Solana network congestion. Jito bundle submission is not yet integrated.

**How it works.** The shared transaction sender wraps every on-chain action with up to 3 send/confirm attempts. On each attempt for a self-built (legacy) transaction, the priority fee is estimated per writable account — using Helius's `getPriorityFeeEstimate` (recommended tier) when available, otherwise the 75th percentile of `getRecentPrioritizationFees` — and clamped to `[1,000, 1,000,000]` µLamports/CU. A `ComputeBudgetProgram.setComputeUnitPrice` instruction is prepended along with a CU limit of 300k, the latest blockhash is refreshed, and the tx is re-signed before each send. If the confirm window (default 60s) expires or the send throws, the fee is multiplied by `1.5` and the next attempt fires after a linearly-growing backoff (500ms × attempt). For externally-built versioned transactions (e.g. Jupiter swap routes whose fee can no longer be modified), the same retry/backoff applies but without fee bumping. Transactions whose blockhash has expired are not retried because re-sending the same tx is futile.

Pending: routing congestion-sensitive transactions (emergency deleverage, swap legs) through Jito bundles for tip-prioritized landing. This is the "partially" qualifier — fee escalation alone is insufficient when block space is auctioned via tips rather than fees.

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
