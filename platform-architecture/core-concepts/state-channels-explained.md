---
description: >-
  How deposits lock funds in smart contracts, how balances update off-chain
  during trading, and when assets move back on-chain on withdrawal.
---

# State Channels Explained

A **state channel** is a mechanism that locks assets on-chain while allowing balance updates to happen off-chain.

#### What happens on-chain

On-chain actions are limited to:

* **Deposit**: assets move from your wallet into a smart contract associated with the state channel.
* **Withdraw**: assets move from the state channel back to your wallet.

These actions create on-chain transactions.

#### What happens off-chain

Inside Yellow, trading does not create an on-chain transaction per fill.

Instead:

* Your trade fills update your **off-chain account ledger**.
* Ownership changes are recorded instantly in that ledger.
* Your off-chain balance is what the platform uses for order placement and risk checks.

#### Why this exists

State channels allow:

* fast balance updates after fills
* fewer on-chain transactions during active trading
* on-chain settlement only when you enter or exit the channel

#### Settlement boundary (important)

* A filled **spot trade** updates your off-chain balances immediately.
* The actual on-chain movement of tokens happens when you withdraw.

#### Withdrawal and bridging

When you withdraw:

* Yellow bridges assets from the state channel back to your on-chain wallet.
* The withdrawal creates the final on-chain settlement for that portion of your balance.

_Diagram: Deposit (wallet → state channel contract) vs Trade (ledger update) vs Withdraw (state channel → wallet)_\
_Screenshot: Balances view showing “Available in account” and the withdrawal flow steps_
