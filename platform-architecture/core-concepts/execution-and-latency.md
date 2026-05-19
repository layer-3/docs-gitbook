---
description: >-
  What ~1 ms matching speed means, what it does not mean, and how network
  conditions affect the latency users actually see.
---

# Execution & Latency

This page explains how Yellow processes orders and how to interpret execution speed statements.

### Execution path (high level)

Yellow separates **trade execution** from **on-chain settlement**.

* Trade execution happens off-chain.
* On-chain transactions are used for deposits and withdrawals.

A typical order flow:

1. Client sends an order request (browser/app/bot)
2. Yellow receives and validates the request
3. Risk checks are applied (balance / margin / order rules)
4. Order is matched
5. Fill is cleared (ledger update, fees applied)
6. Result is returned to the client
7. Updates are streamed to the client (WebSocket, if connected)

Each step adds time.

**Diagram:** Client → API → Validation/Risk → Matching → Clearing → Response / WebSocket update

### Matching latency vs end-to-end latency

When Yellow refers to **\~1 ms matching speed**, this refers to **matching engine processing latency** under internal test conditions.

It does **not** mean every user order will complete end-to-end in 1 ms.

End-to-end latency depends on:

* user location
* network routing and ISP conditions
* client setup (browser/app/bot/server)
* internet congestion
* cloud/network path conditions

This is why different users can observe different latency for the same market.

### No artificial delay (current behavior)

Yellow currently does not add artificial delay to customer orders as part of the matching process, unless explicitly documented for a specific market or feature.

This means:

* orders are processed as received, subject to validation and risk checks
* Yellow does not intentionally delay retail order flow for speed-tiering or priority routing (unless later documented)

If this changes for a feature or market, it should be documented in the relevant page.

### Public mempool and front-running context

Yellow order execution does not rely on a public blockchain mempool for trade matching.

This means mempool-based reordering/sandwich behavior associated with public on-chain execution does not apply to Yellow’s trade execution path.

This does not remove all execution risk. Users should still consider:

* market volatility
* liquidity depth
* queue position
* order type selection (market vs limit)

### What this metric is (and is not)

The **\~1 ms** number is useful as:

* an engine-level performance indicator

It is not:

* a user-experience SLA
* an API round-trip SLA
* a guarantee of fill time under all market conditions

For integration guidance, see **API & SDK → Introduction**.

**Screenshot:** (optional) Latency explanation callout in docs/help center
