---
description: >-
  How Yellow combines liquidity sources into one trading experience, how orders
  can split into multiple fills, and what that means for price and fees.
---

# Aggregated Liquidity

Aggregated liquidity means Yellow can execute orders using more than one liquidity source while presenting a unified trading experience.

### Why Yellow aggregates liquidity

Liquidity is often fragmented across venues, chains, and counterparties. Aggregation is used to:

* increase available depth
* reduce partial fills on larger orders
* improve execution consistency

### One order book model (user view)

Yellow presents markets through a unified order book view, while sourcing liquidity from multiple inputs supported by the platform.

A liquidity source can include:

* Yellow-native order flow
* connected brokers or counterparties
* integrated external liquidity sources supported by Yellow

### How orders are executed

Yellow selects an execution path based on:

* available size
* price levels
* order constraints (for example: post-only, reduce-only)
* risk and settlement rules

A single order may fill in multiple parts across different price levels.

This means:

* your order may have multiple fill entries
* fees are calculated per fill
* the final average fill price can differ from the top-of-book price shown at submission

### What does not change

After fills:

* balances update in your Yellow off-chain account ledger
* on-chain settlement still happens only when you withdraw

**Diagram:** One order → multiple liquidity sources → multiple fills → ledger update\
**Screenshot:** Trade history showing one order split into multiple fills
