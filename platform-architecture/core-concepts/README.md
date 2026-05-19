---
description: >-
  How Yellow handles custody, execution, clearing, and settlement, and what
  happens off-chain vs on-chain when you trade or withdraw.
hidden: true
---

# Core Concepts

This section explains how Yellow handles custody, trading, and settlement at a system level.

Yellow separates **trading updates** from **on-chain settlement**:

* Trading updates happen in your **off-chain account ledger** (fast balance updates after fills).
* On-chain settlement happens when you **deposit** or **withdraw** (assets move between your wallet and the state channel).

Core components:

* **State Channels**: where funds are locked on-chain and balances are updated off-chain.
* **Yellow Clearing Network**: the system that validates trades, updates balances, and manages obligations across participants.
* **Aggregated Liquidity**: how Yellow combines liquidity sources for execution.

A typical flow:

1. You deposit from your on-chain wallet.
2. Funds are locked in a smart contract and credited to your off-chain account ledger.
3. You place orders; fills update your ledger instantly.
4. You withdraw; assets are bridged from the state channel back to your on-chain wallet.

_Diagram: System overview (Wallet → State Channel → Off-chain Ledger → Withdrawal back to Wallet)_\
_Screenshot: Deposit/Withdraw screens showing the “on-chain vs account” boundary_
