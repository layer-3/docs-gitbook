---
description: >-
  How Yellow turns matched orders into balance updates, applies fees, and
  records fills in the off-chain ledger.
---

# The Yellow Clearing Network

The **Yellow Clearing Network** is the system that records fills, updates balances, and manages settlement obligations inside the platform.

#### What “clearing” means here

When your order fills, Yellow needs to:

* confirm the fill details (price, size, fee)
* update both sides’ balances in the off-chain ledger
* apply fees and (for perps) funding and PnL rules
* enforce margin/risk constraints (for perps)

This is clearing: converting a match into ledger updates and obligations.

#### Where it sits in the flow

1. You submit an order.
2. Yellow checks that you have sufficient balance/margin.
3. The order is matched (internally or via an execution route).
4. The Clearing Network finalizes the fill:
   * writes the fill to your history
   * updates balances immediately in the off-chain ledger
   * applies fees
5. Your updated balance becomes available for the next action (new orders, position changes, withdrawals).

#### Netting (reducing transfers)

Clearing systems typically reduce the amount that needs to be settled by netting opposing obligations. In practice this means:

* many trades can occur while your assets remain inside the state channel
* on-chain movement is limited to deposits/withdrawals, not each trade

#### Related: execution timing

Clearing and matching are part of the execution path, but the time a user experiences also includes client-side and network latency.

For how Yellow defines matching speed and how it differs from end-to-end latency, see [**Execution & Latency**.](execution-and-latency.md)

#### What this means for you

* Fills update balances immediately in the platform.
* Deposits and withdrawals are the only points where assets move on-chain.

_Diagram: Order → Match → Clearing (ledger updates + fees) → Account balance updated_\
_Screenshot: Fill confirmation in trade history + balance change after fill_
