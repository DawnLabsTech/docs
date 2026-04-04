# Fees & How to Use

## Fee Schedule

| Fee | Rate | Description |
|-----|------|-------------|
| **Performance Fee** | 20% | Charged on profits above the High Water Mark |
| **Management Fee** | 1% per year | Annual fee on total assets under management |
| **Deposit Fee** | 0% | No fee on deposits |
| **Redemption Fee** | 0.1% | Small fee on withdrawals |

### Performance Fee (HWM)

The performance fee is only charged on **new profits** using a High Water Mark system:

```
Share Price History:

$1.00 → $1.10 → $1.05 → $1.15
       ↑ Fee      No fee  ↑ Fee
       (on $0.10)         (only on $0.05,
                           i.e. $1.15-$1.10)
```

- Fee is only charged when share price exceeds the previous HWM
- If the vault loses money and recovers, no fee until the previous high is surpassed
- Calculated using LP token dilution model — depositor share prices reflect fees automatically

### Locked Profit Mechanism

Profits are locked and released linearly over a configurable duration (Yearn V2 model). This prevents frontrunning — you can't deposit right before a profit event and withdraw immediately after.

All fee parameters are set on-chain and visible to anyone.

---

## How to Deposit

### Prerequisites

- A Solana wallet (Phantom, Solflare, or Backpack recommended)
- USDC on Solana (SPL token)
- A small amount of SOL for transaction fees (~0.01 SOL)

### Steps

1. Visit the Dawn Vault web app and **Connect Wallet**
2. Select the **USDC Vault** and review details (current APY, TVL, remaining capacity)
3. Enter the amount of USDC to deposit
4. Review the transaction summary (deposit amount, LP tokens to receive, estimated APY)
5. Click **Deposit** and approve the transaction in your wallet
6. After confirmation, LP tokens appear in your wallet and yield begins accruing immediately

### What Happens After Deposit

- Your USDC is transferred to the vault's PDA (non-custodial)
- You receive LP tokens representing your share of the vault
- The Manager Bot allocates your capital according to the current strategy
- Yield auto-compounds — no claiming or harvesting needed

---

## How to Withdraw

### Steps

1. Connect your wallet and navigate to the USDC Vault page
2. Click **Withdraw** and enter the amount of LP tokens to redeem (or "Max" for full withdrawal)
3. Review: LP tokens to burn, USDC to receive (after 0.1% redemption fee)
4. Click **Confirm Withdrawal** and approve the transaction
5. USDC appears in your wallet

**Your withdrawal amount = LP tokens × current share price**

### Processing Time

- **Standard**: Processed immediately on-chain
- **Large withdrawals**: May take additional time if positions need to be unwound

### Important Notes

- No lock-up period — withdraw anytime
- Yield is reflected in the share price (no separate "claim" step)
- Performance fees are already deducted from the share price — what you see is what you get
- Deposits may be rejected if the vault has reached its TVL cap
