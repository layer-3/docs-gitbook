---
description: >-
  What public and private streams are available, how to connect and reconnect,
  and how to keep state in sync.
hidden: true
---

# WebSocket API Reference

WebSocket is the recommended way to keep account state in sync.

This page lists the stream types.\
For message schemas and channel names, use **docs.yellow.org**.

**Link:** Full WebSocket reference (docs.yellow.org)

### Public streams

* order book updates
* trades
* candles/ticker (if provided)

### Private streams (auth required)

* user orders (status updates)
* user fills
* positions (perps)
* balances
* funding events (perps)

### Connection rules

* Keep-alive: respond to ping/pong if required.
* Reconnect: on disconnect, reconnect and resubscribe.
* Ordering: do not assume events are strictly ordered across channels.
* Consistency: periodically reconcile using REST (example: fetch open orders).

### Typical client loop

1. connect
2. authenticate
3. subscribe
4. process messages
5. on disconnect: reconnect + resubscribe

**Screenshot:** WS auth + subscribe example
