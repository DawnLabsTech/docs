# AI Advisor

Dawn Vault includes an AI-powered advisory system that monitors the vault's state and provides actionable recommendations to the operator.

## Overview

The AI Advisor is a **non-executing** advisory layer. It observes the vault's positions, market data, and risk metrics, then suggests actions — but never takes them autonomously. All execution remains under the rule-based system or manual operator approval.

This design prioritizes safety: the worst case for an AI error is a missed or irrelevant notification, never an unauthorized trade.

## How It Works

```
Bot Internal State (positions, FR, APY, risk scores, health rates, events)
    ↓
Context Builder → Structured summary for the LLM
    ↓
Claude API (Sonnet) → Analyzes context, produces recommendations
    ↓
Store (SQLite) + Notify (Telegram) + Dashboard (sidebar)
```

1. **Context Builder** gathers the bot's current state: portfolio positions, funding rate history, APY across protocols, Multiply health rates, risk scores, recent events, and daily PnL.
2. **Claude API** receives this structured context along with a system prompt describing the vault's strategy rules, and returns a JSON array of recommendations.
3. **Recommendations** are stored in SQLite for history/accuracy tracking, sent via Telegram, and displayed on the monitoring dashboard.

## When It Runs

| Trigger | Condition | Frequency |
|---|---|---|
| **Periodic** | Every 6 hours | Scheduled |
| **SOL price move** | >= 5% change since last advisor run | Checked every 5 min |
| **Risk score spike** | Composite score crosses 50 from below | Checked every 5 min |
| **FR regime change** | Funding rate sign flip or >= 10% annualized swing | Checked every 5 min |

All event-based triggers enforce a **30-minute cooldown** to prevent excessive API calls.

## Recommendation Format

Each recommendation includes:

- **Category**: `rebalance`, `dn_entry`, `dn_exit`, `risk_alert`, or `param_adjust`
- **Confidence**:
  - `high` — Data-backed, directly verifiable from the numbers (e.g., "health rate is 1.05, below emergency threshold")
  - `medium` — Reasonable inference that depends on near-term market assumptions
  - `low` — Speculative; rarely issued
- **Urgency**:
  - `immediate` — Something is broken or a threshold is being breached
  - `next_cycle` — Should be addressed within 6–12 hours
  - `informational` — For awareness only, no action needed
- **Override flag** — `true` when the AI recommends a different action than the rule-based system would take. Displayed as a yellow highlight on the dashboard.

## Dashboard

The AI Advisor panel appears as a **sticky sidebar** on the monitoring dashboard. Recommendations are shown in a compact one-line format (category + action summary + confidence badge + time ago). Click to expand for full reasoning and rule context.

## Safety Guarantees

- **No execution authority**: The advisor cannot trigger trades, rebalances, or parameter changes.
- **Graceful degradation**: If the API key is missing or the API is unreachable, the bot continues operating normally with rule-based logic.
- **Cost control**: Daily API call limit (default 20), 30-minute event cooldown, compact context to minimize token usage (~$0.02/call).
- **Accuracy tracking**: All recommendations are stored with an `outcome` field for post-hoc evaluation.

## Cost

Using Claude Sonnet, each evaluation costs approximately **$0.01–0.03** (1,500–2,000 input tokens + 500–1,000 output tokens). At 4–6 calls per day, monthly cost is approximately **$2–5**.
