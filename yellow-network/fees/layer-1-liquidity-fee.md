---
description: Direct P2P rewards for liquidity providers
---

# Layer 1 - Liquidity Fee

The Liquidity Fee is agreed upon between the brokers when opening a channel. The broker acting as a market maker will set off the fees paid by the broker acting as a market taker.

The Liquidity Fee is volume-based, calculated at position opening in % of the base currency of the market taker and added to, or subtracted from, the market taker's liability.

<mark style="background-color:yellow;">**Liquidity Fee = Trade Nominal in  Liability Ccy x Market Taker Fee (%)**</mark>

Broker Fees are due on settlement, directly charged to the market taker's wallet, and transferred to the market maker.

Upon settlement, Layer-1 fees are paid in full (100%) from the market taker to the market maker.&#x20;

| Defined by  | Brokers when they establish a trading channel                          |
| ----------- | ---------------------------------------------------------------------- |
| Beneficiary | Market making broker                                                   |
| Payment Ccy | Liability in currency of trade                                         |
| Periodicity | Calculated on every trade and charged on the settlement at 0.00 am UTC |
