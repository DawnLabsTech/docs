# Emergency Response

This document defines Dawn Vault's emergency response framework — the policies, procedures, and communication channels for responding swiftly and appropriately when asset-impacting risks arise, such as protocol exploits, collateral crashes, or network outages.

## A. Detection

Catch anomalies as early as possible. Multiple detection layers run in parallel to eliminate single points of failure.

### A1. Protocol Anomaly Detection (Partially Implemented)

Monitor contracts of utilized protocols (Kamino, Jupiter, etc.) for abnormal large transfers or permission changes. Integration with external security feeds (Hypernative, Forta Network, etc.) is also under consideration as a longer-term complement.

**Phase 1 — Upgrade Authority Monitoring (Implemented).** The Kamino Lending program's upgrade authority is the highest-signal change to watch: any change here means a new code path could be deployed against existing user funds, so detecting it within seconds is critical.

| Trigger | Threshold | Action |
|---------|-----------|--------|
| Kamino Lending program upgrade authority change | Any change (or transition to/from frozen) | Critical Telegram alert + `ANOMALY` event recorded |

**How it works.** The bot derives the BPF Loader Upgradeable **ProgramData PDA** for the Kamino Lending program (this is the on-chain account that stores the `upgrade_authority` field) and monitors it through two parallel channels:

1. **Helius webhook (primary, low latency).** A webhook configured against the ProgramData PDA address fires whenever that account is touched (which only happens on program upgrades — normal program invocations do not load the PDA into a transaction's account keys). The webhook hits a dedicated `POST /webhook/helius` endpoint authenticated by a separate `HELIUS_WEBHOOK_AUTH` bearer (independent of the dashboard auth token), and triggers an immediate check.
2. **Scheduled polling (fallback, every 1 hour).** Even if the webhook is misconfigured or Helius is degraded, a polling task in the orchestrator's scheduler runs the same check. This bounds detection latency to at most ~1 hour in the worst case.

In both cases the detection logic is identical and intentionally **does not parse the webhook payload** — instead, the bot re-fetches the ProgramData account directly via `getAccountInfo`, parses the `upgrade_authority` field from the bincode-serialized account data, and compares it against the baseline persisted in a SQLite `anomaly_baseline` table. The baseline is seeded once at bot startup. On divergence:

- A critical-level Telegram alert is fired with both the previous and current authority pubkeys (or `<frozen>` if the program transitioned to having no upgrade authority).
- An `ANOMALY` event is recorded in the events ledger with `metadata.action = 'anomaly_upgrade_authority_change'` so it appears in the dashboard separately from generic alerts.
- The baseline is then updated to the new value so subsequent checks don't re-fire on the same change. Operator follow-up is expected to investigate whether the change was legitimate (e.g. announced Kamino upgrade) or hostile.

This payload-agnostic design means the system continues to work even if Helius changes its webhook payload format or if the watched address starts receiving unrelated traffic.

**Roadmap for A1**

| Phase | Detection | Status |
|-------|-----------|--------|
| Phase 1 | Kamino Lending program upgrade authority change | Implemented |
| Phase 2 | Large transfers from Kamino USDC reserve / ONyc collateral vaults (>20% of reserve in 1h = warning, >40% = critical) | Planned |
| Phase 2 | Reserve config changes (LTV, oracle source, liquidation bonus) | Planned |
| Phase 3 | Jupiter Lend program coverage | Planned |
| Phase 3 | Anomaly event timeline in the frontend dashboard | Planned |

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
| Consecutive balance-check failures | 3 times | Disable affected protocol |

After exit, a 24-hour cooldown period applies before automatic re-enablement is attempted.

**How it works.** A scheduled task runs every 60 seconds across all active lending protocols (currently Kamino and Jupiter Lend) and performs two checks per cycle:

1. **TVL** — fetched from the Kamino market metrics endpoint (or Jupiter's earn-tokens endpoint as a proxy). Each cycle's TVL is appended to a rolling history bounded by the 1-hour window; if the oldest sample in-window is ≥20% above the latest, the protocol is tripped.
2. **Withdrawal health** — the protocol's `getBalance()` is called as a liveness probe. Each failure increments a per-protocol counter; success resets it. Three consecutive failures trip the protocol on the assumption that withdrawals are unlikely to succeed either.

Oracle anomaly detection — previously a third check inside the circuit breaker — has been split out into a dedicated monitor (see A5) that uses multi-source cross-checks rather than a single price feed.

When tripped, the breaker invokes an `onTrip` callback (wired by the orchestrator to the protocol's emergency withdrawal), records an `ALERT` event, and adds the protocol to a disabled set so subsequent cycles skip it. The 24-hour cooldown is checked on every cycle: once elapsed, the protocol is removed from the disabled set, the failure counter is cleared, and monitoring resumes. A public `trip(name, reason)` entry point lets external monitors (e.g. A5) request a circuit-breaker trip through the same code path. A manual `enableProtocol()` override is also available for operator intervention.

### A4. Kill Switch (Implemented)

A manual last-resort mechanism for humans to immediately halt all operations. Detected on the next health check (every 5 seconds), triggering the full emergency exit sequence for all positions.

**How it works.** The bot's 5-second health-check loop calls `checkKillSwitch()` first, before any other guardrail. The check is a simple file existence test against `/tmp/vault-kill` (overridable via the `VAULT_KILL_SWITCH_PATH` env var) — operators activate it by SSH-ing into the host and running `touch /tmp/vault-kill`. When the file is detected: a critical alert is fired, an `EMERGENCY_EXIT` action with `reason: 'kill_switch'` is dispatched (which unwinds the base delta-neutral position), the orchestrator stops scheduled tasks, and the process exits with code 1 so the supervisor (Docker / systemd) does not auto-restart. A file-based trigger was chosen over an API endpoint or env var so the path is reachable even when the bot's internal state machine or HTTP server is wedged.

### A5. Oracle Anomaly Detection (Implemented)

Detects when the prices Kamino uses for liquidation diverge from independent reference sources. The dangerous case is the oracle silently over-pricing the collateral relative to actual market depth — health-rate readings look fine, but the real liquidation buffer is much smaller than reported.

| Check | Source comparison | Default threshold | Severity / Action |
|-------|-------------------|-------------------|-------------------|
| Stable depeg | Kamino reserve oracle vs $1.00 | ≥ 50 bps / ≥ 100 bps | Warn / Critical → trip all protocols + emergency-deleverage every Multiply position |
| Cross-source divergence | Kamino implied collateral/debt ratio vs Jupiter swap quote | ≥ 50 bps / ≥ 100 bps | Warn / Critical → emergency-deleverage the affected market |
| Collateral over-pricing (directional) | Kamino oracle > Jupiter quote | ≥ 75 bps | Critical → emergency-deleverage the affected market (tighter than the symmetric threshold because this direction is the dangerous one) |
| Kamino stale cache | Reserve `stored` vs `oracle` price | ≥ 100 bps | Warning only (observability — points to refresh delay) |
| Pyth staleness | Hermes `publishTime` | > 60 s old | Warning only |
| Pyth confidence | Pyth `conf / price` | > 1.0 % | Warning only |

**Sustained gate.** Critical actions only fire after the condition has held for **3 consecutive samples** (≈15 minutes at the default 5-minute cadence). A single sample produces a critical event (visible in logs and event ledger) but its `sustained` flag is false until the gate is satisfied; the orchestrator gates trip / deleverage dispatch on `sustained === true`. Transient fetch failures (Jupiter / Pyth API hiccups) preserve the counter rather than resetting — only a cycle that successfully evaluates the condition and finds it OK clears the counter. Warning-tier events still emit on the first sample, so operators see the situation immediately even though automated action waits.

**How it works.** A scheduled task runs every 5 minutes (aligned with `kamino-multiply-health`) and, for each Multiply market, reads the on-chain Kamino reserve state for both collateral and debt: `getOracleMarketPrice()` (the fresh oracle price) and `getReserveMarketPrice()` (the cached price Kamino uses for liquidation math until the next refresh). Each market then runs four checks:

1. **Stable depeg** — for any side whose mint is a known $1.00-pegged stablecoin (USDC, USDT, PYUSD, USDG), the Kamino oracle is compared to the peg.
2. **Kamino stored vs oracle** — the stored/oracle delta. A wide delta means Kamino's internal cache hasn't caught up to a moving oracle, so health-rate calc is running on a stale price.
3. **Cross-source divergence** — for non-stable collateral, Jupiter is asked for a real swap quote (~$100 of collateral → debt token). The Kamino-implied ratio (`collOracle / debtOracle`) is compared to the Jupiter quote. The directional check (Kamino over-prices vs DEX) uses a tighter threshold because that's the failure mode that silently inflates health rate.
4. **Pyth health (optional)** — for any mint with a configured Pyth Hermes price feed (`ORACLE_PYTH_PRICE_IDS` env), staleness and confidence-interval width are checked as independent oracle health signals.

Action dispatch lives in the orchestrator. `stable-depeg` events are NAV-wide (the borrow leg of every position is suspect) so they trip the circuit breaker for all protocols and emergency-deleverage every Multiply adapter. `cross-source-dev` events are market-specific. Pyth and stale-cache warnings are alert-only — they're early-warning signals, not action triggers, since acting on them automatically would create false-positive churn.

**Configuration.** All thresholds and the cadence live under `oracleMonitor` in `config/default.json`; the bot picks up changes via the file-watcher hot reload. Pyth Hermes monitoring is opt-in via the `ORACLE_PYTH_PRICE_IDS` environment variable, set to a comma-separated `mint:priceId` list (e.g. `EPjFWdd5...:0xeaa020c6...,DAwNds...:0xabc...`). When the env is unset or empty, Pyth checks are skipped while Kamino-internal and Jupiter cross-source checks continue unaffected.

| Key | Default | Meaning |
|---|---|---|
| `checkIntervalMs` | 300_000 (5 min) | Cycle cadence — aligned with `kamino-multiply-health` |
| `sustainedSamples` | 3 | Consecutive samples a critical condition must hold before automated action |
| `usdcDeviationWarnBps` / `...CriticalBps` | 50 / 100 | Stable depeg thresholds |
| `kaminoStaleStoredBps` | 100 | Stored-vs-oracle delta that flags a stale Kamino cache |
| `onycCrossSourceWarnBps` / `...CriticalBps` | 50 / 100 | Symmetric cross-source divergence thresholds |
| `onycOverpriceCriticalBps` | 75 | Tighter directional threshold for Kamino over-pricing collateral |
| `pythStalenessSec` | 60 | Maximum tolerated `publishTime` age |
| `pythConfidencePct` | 1.0 | Maximum tolerated `conf / price` ratio |
| `alertCooldownMs` | 1_800_000 (30 min) | Suppresses repeat Telegram alerts for the same `(kind, market, mint, severity)` |

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
│   ├── Depeg → Direct liquidation risk (Multiply Risk Scorer + A5 cross-source)
│   └── DEX Liquidity (ONyc/USDC) → Exit impossible (A5 cross-source picks up the gap between Kamino oracle and DEX)
│
├── Borrowed Asset: USDC
│   ├── Oracle (Pyth/Switchboard) → Price feed anomaly (A5)
│   ├── Borrow Rate → Spike causing negative carry (A2)
│   └── USDC Depeg → NAV-wide impact (A5 stable-depeg)
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
| ONyc depeg (>2%) | Health rate decline → Liquidation | Multiply Risk Scorer + A5 (cross-source) | Staged deleverage; A5 sustained critical → emergency deleverage |
| ONyc DEX liquidity vanishes | Unable to swap during deleverage | C1 (periodic simulation) + A5 cross-source (Kamino oracle vs DEX gap widens) | Split withdrawal, raise slippage cap |
| USDC depeg | NAV-wide impact (every borrow leg) | A5 stable-depeg | Sustained critical → trip all protocols + emergency deleverage every Multiply position |
| Onre operations halt | ONyc unredeemable, yield stops | Manual monitoring, contact Onre | Full exit |
| Borrow rate spike | Negative carry erodes NAV | A2 (spike detection) | Auto deleverage |
| Pyth oracle anomaly | Incorrect liquidation or misjudgment | A5 (multi-source cross-check, staleness, confidence) | Sustained critical → emergency deleverage / circuit breaker trips |
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
