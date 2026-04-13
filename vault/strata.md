# Strata — Phase 2: Tranched Yield Vault

> *"Stop losing sleep over hacks. Take the Senior tranche."*

Strata is Dawn Labs' next product: a tranched yield vault on Solana designed for institutional depositors. By splitting a single underlying strategy into Senior and Junior tranches, Strata structurally separates risk and return — putting institutions into the protected tier and risk-tolerant LPs into the high-yield tier.

---

## Why Now

Since the Drift hack (2025), institutional capital has largely stayed out of DeFi vaults. The common objection: *"We can't get internal approval when total-loss risk exists."* Strata answers this directly. Junior depositors absorb losses first; Senior depositors are protected until Junior NAV reaches zero. This is the on-chain equivalent of a CLO (Collateralized Loan Obligation) — a structure that has worked in traditional finance for decades.

---

## Two Layers

### Layer 1 — Risk: Senior / Junior Tranche

| | Senior Vault | Junior Vault |
|---|---|---|
| **Return** | 8% fixed (capped) | Variable — all residual upside |
| **Loss priority** | Last to absorb | First-loss buffer |
| **Withdrawal** | Instant | 7-day lock |
| **Target** | Institutions, stable-yield seekers | High-yield seekers, risk-tolerant LPs |

**Accounting Waterfall (exit priority):**

```
Senior payout = user_senior_share × min(total_nav, senior_total_deposits)
Junior payout = user_junior_share × max(total_nav - senior_total_deposits, 0)
```

- **Profit scenario**: Senior receives 8% fixed first → remaining distributed to Junior
- **Loss scenario**: Junior NAV is drawn down first → Senior is protected until Junior is fully wiped

### Layer 2 — Strategy: Base / Alpha / DN

| | Base Layer | Alpha Layer | DN Layer |
|---|---|---|---|
| **Strategy** | Kamino Lending | Kamino Multiply | SOL Delta-Neutral |
| **Yield** | ~5–7% (stable) | ~14–16% (primary) | Variable (funding rate dependent) |
| **Activation** | Always (overflow) | Always (primary) | SOL funding rate > 10% for 3+ days |
| **Risk** | Low | Medium (leveraged) | Medium (hedged) |

Capital allocation priority: concentrate in Alpha (Multiply) → overflow excess into Base (Lending) → activate DN conditionally.

---

## Architecture

```
[Dawn Senior Vault]     [Dawn Junior Vault]
        │                       │
        └───────────┬───────────┘
                    ▼
        Dawn Tranche Program (Anchor)
        · NAV tracking
        · Waterfall accounting
        · Withdrawal priority enforcement
                    │  (single capital pool)
                    ▼
        Ranger Vault (execution layer)
        ├─ Kamino Multiply adaptor  (Alpha Layer / primary)
        └─ Kamino Lending adaptor   (Base Layer / overflow)
```

**Tech stack:**
- **Anchor** (Dawn Tranche Program): Issues Senior-LP / Junior-LP tokens, applies waterfall math at withdrawal
- **[Ranger Finance](https://docs.ranger.finance/)** (Vault-as-a-Service infrastructure)
  - `withdrawal_waiting_period` natively implements the 7-day Junior lock
  - CPI: `deposit_vault` / `request_withdraw_vault` / `withdraw_vault` / `instant_withdraw`
- **Kamino Finance**: Multiply + Lending adaptors

**Key design insight:**
Ranger manages a single asset-per-share ratio across all LPs in a vault. The Tranche Program sits between Ranger and end users — it holds Ranger LP tokens as an intermediary, then applies its own waterfall math at withdrawal to distribute unequally to Senior vs. Junior depositors. Senior gets the protected slice; Junior gets what's left.

---

## Junior Bootstrap

Initial Junior capital is provided by **Dawn Labs itself**.

- Puts Dawn Labs' own money at first-loss, signaling skin-in-the-game to Senior LPs
- As track record accumulates, external Junior LPs seeking high yield can be onboarded
- At hackathon demo stage, Dawn Labs capital fully covers the Junior tranche

---

## Competitive Landscape

### TradFi Precedents
- **CDO / CLO**: Mortgages and leveraged loans tranched into AAA (Senior) through Equity (Junior). Strata is the on-chain CLO equivalent.
- **Real estate preferred/subordinate structures** (common in Japan): Priority investor (principal protected) / subordinate investor (absorbs losses first, captures upside). Identical structure.

### Crypto Precedents (Failed)
- **Barnbridge** (Ethereum, 2021): SMART Yield split Compound yields into Senior/Junior. $178M TVL → collapsed when low interest rates made the fixed Senior rate unsustainable.
- **Saffron Finance** (Ethereum, same era): Same structure, same failure mode.

### Direct Solana Competitor
- **Kormos** (Cypherpunk Sep 2025, DeFi 2nd place, C4 accelerator)
  - Structure: Liquid depositor (Junior-like) / Locked depositor (Senior-like)
  - Narrative: "Fractional reserve banking on DeFi"
  - Key difference: **Locked = first-loss** — the tranche direction is inverted. Not targeting institutions or explicit total-loss protection.

### Strata's Differentiation
1. **Narrative**: Directly responds to the Drift-hack-era "total-loss risk kills institutional approval" pain point
2. **Target customer**: Institutions in the Senior tranche — designed to pass internal investment committees
3. **Yield headroom**: Kamino Multiply (~16%) provides enough raw yield for Senior 8% fixed to be sustainable — the failure mode of Barnbridge
4. **Operating track record**: Dawn Labs runs Phase 1 Vault on live capital; strategy and risk management credibility already exists

---

## Open Questions / Todo

- [ ] Anchor program detailed design — how much NAV calculation lives on-chain vs. off-chain
- [ ] Ranger Vault Owner registration and adaptor configuration steps (to confirm with Ranger team)
- [ ] CPI routing feasibility — can a single Anchor program hold Ranger LP tokens for two separate depositor pools?
- [ ] Resolve potential conflicts: locked profit degradation timing vs. Senior 8% payout, HWM fee accrual vs. Tranche-side accounting
- [ ] Junior external onboarding: timing, eligibility, and minimum size design
- [ ] Pitch deck creation
