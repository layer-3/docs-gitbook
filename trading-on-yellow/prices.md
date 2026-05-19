---
description: >-
  Learn which price values appear on Yellow market pages, what they mean, and
  how they relate to order execution.
---

# Prices

Yellow can show different price types for the same market. Each one is used for a different purpose.

### Last Price

The **Last Price** is the price of the most recent trade.

Use it for:

* seeing the latest executed price in the market

Do not assume the **Last Price** is the same as the price your order will fill at.

Your actual fill depends on:

* available liquidity
* order type
* order size
* current order book conditions

### Reference Price

Yellow may also display a **reference price** derived from external market data.

This price can be used as a benchmark for market information and price comparison.

### Which Price Is Used for What

At a high level:

* order matching uses the order book, not the displayed reference price directly
* the **Last Price** shows the most recent completed trade
* displayed reference values may be used to provide additional market context in the interface

### Why This Matters

The price shown in the market header is useful for reference, but it does not guarantee the exact execution price of your order.

Execution depends on current market conditions at the time your order reaches the book.

### What to Check in the UI

Before placing an order, confirm:

* the market you selected
* the price shown in the order form
* your order type
* your entered size and total order value
