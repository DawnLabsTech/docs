# AI Support

Dawn Vault includes two AI-assisted operator tools:

- **AI Advisor**: an event-driven advisory layer that evaluates the vault and produces recommendations
- **Vault AI Chat**: an interactive chat interface that answers questions about the live vault state and can run read-only support tools

Both features are **non-executing**. They can observe, summarize, explain, and simulate, but they cannot place trades or override the rule-based system.

## Overview

| Feature | Primary Role | Access Point | Output |
|---|---|---|---|
| **AI Advisor** | Push-style monitoring and recommendations | Dashboard sidebar, Telegram notifications, stored history | Structured recommendations |
| **Vault AI Chat** | Pull-style operator Q&A and scenario analysis | Dashboard chat widget, `/api/chat` SSE endpoint | Streaming plain-text responses |

This design keeps AI in the support layer. The worst case for an AI mistake is an irrelevant suggestion or a low-quality explanation, not an unauthorized on-chain or exchange action.

## AI Advisor

The AI Advisor continuously monitors the vault's state and produces actionable recommendations for the operator.

### How It Works

```
Bot Internal State (positions, FR, APY, risk scores, health rates, events)
    ↓
Context Builder → Structured summary for the LLM
    ↓
Claude API → Analyzes context, produces recommendations
    ↓
Store (SQLite) + Notify (Telegram) + Dashboard (sidebar)
```

1. The context builder gathers the bot's current state: portfolio positions, funding rate history, APY across protocols, Multiply health rates, risk scores, recent events, and daily PnL.
2. Claude receives this structured context along with a system prompt describing the vault's strategy rules, then returns a JSON array of recommendations.
3. Recommendations are stored in SQLite for history and accuracy tracking, sent via Telegram, and displayed on the monitoring dashboard.

### When It Runs

| Trigger | Condition | Frequency |
|---|---|---|
| **Periodic** | Every 6 hours | Scheduled |
| **SOL price move** | >= 5% change since last advisor run | Checked every 5 min |
| **Risk score spike** | Composite score crosses 50 from below | Checked every 5 min |
| **FR regime change** | Funding rate sign flip or >= 10% annualized swing | Checked every 5 min |

All event-based triggers enforce a **30-minute cooldown** to prevent excessive API calls.

### Recommendation Format

Each recommendation includes:

- **Category**: `rebalance`, `dn_entry`, `dn_exit`, `risk_alert`, or `param_adjust`
- **Confidence**:
  - `high` — data-backed and directly verifiable from the numbers
  - `medium` — reasonable inference that depends on near-term assumptions
  - `low` — speculative; rarely issued
- **Urgency**:
  - `immediate` — something is broken or a threshold is being breached
  - `next_cycle` — should be addressed within 6–12 hours
  - `informational` — awareness only
- **Override flag** — `true` when the AI recommends a different action than the rule-based system would take

### Surfaces

- Dashboard sticky sidebar
- Telegram notifications
- SQLite recommendation history with outcome tracking

### Safety Guarantees

- **No execution authority**: the advisor cannot trigger trades, rebalances, or parameter changes
- **Graceful degradation**: if the API key is missing or the API is unreachable, the bot continues with rule-based logic only
- **Cost control**: daily API call limit, 30-minute event cooldown, compact context
- **Accuracy tracking**: recommendations are stored with an `outcome` field for post-hoc review

## Vault AI Chat

Vault AI Chat is the operator-facing conversational interface for Dawn Vault. It answers questions using the current vault context and recent session history.

### How It Works

```
User question
    ↓
Current vault context + recent AI Advisor recommendations + session history
    ↓
Claude chat model
    ↓
Streaming answer in dashboard chat widget or `/api/chat`
```

On first message, the chat injects the latest vault context directly into the conversation. On later turns, it keeps short session memory and continues answering against the latest known vault state.

### What It Can Do

- Explain current positions, allocation, performance, and risk posture
- Answer in Japanese or English depending on the operator's language
- Stream responses token-by-token in the UI
- Store per-session chat history in SQLite
- Use built-in support tools when the question requires more than plain Q&A

### Built-in Support Tools

| Tool | Purpose |
|---|---|
| `run_backtest` | Run "what if" simulations for APYs, funding rate thresholds, allocation ratios, date ranges, and initial capital |
| `get_advisor_history` | Retrieve recent AI Advisor recommendations and 7-day accuracy statistics |

These are **chat tools**, not separate AI products. They extend the chat's ability to answer operator questions with simulations and historical advisory context.

### Access Points

- Floating chat widget on the monitoring dashboard
- `/api/chat` SSE endpoint for streaming responses

### Limits and Safety

- **No execution authority**: chat cannot place trades or mutate live strategy state
- **Requires API key**: if `ANTHROPIC_API_KEY` is not configured, chat remains disabled
- **Per-session rate limit**: 30 user messages per hour by default
- **Read-only support scope**: analysis, explanation, and backtests only
