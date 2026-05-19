---
description: >-
  What you can do with the API and SDK, which environments exist, and where to
  find the full developer reference.
hidden: true
---

# Introduction

Yellow provides an API for:

* reading market data and account state
* placing and managing orders
* streaming updates over WebSocket (fills, orders, positions)
* exporting history (fills, funding, fees, transfers)

This section is a product-level overview.\
For the complete developer reference (all endpoints, schemas, error codes), use **docs.yellow.org**.

**Link:** Full API docs (docs.yellow.org)

### Environments

Yellow exposes separate environments:

* Production
* Testnet / Sandbox (if available)

Each environment has its own base URL and API keys.

**Screenshot:** API page showing Base URL(s) and environment selector (if UI has it)

### Key concepts

* Authentication is done using API keys (and request signing if required).
* Orders submitted by API behave the same as orders placed in the UI.
* WebSocket streams are the recommended way to track real-time user state (orders/fills/positions).

### Execution speed

Matching engine speed and client-observed latency are different metrics.

References to **\~1 ms matching speed** describe engine processing under internal test conditions. They do not guarantee:

* API round-trip time
* order acknowledgment time
* end-to-end fill time from your infrastructure

Your observed latency depends on:

* server location
* network path
* connection quality
* client implementation (retries, serialization, WebSocket handling)

Use WebSocket for real-time state updates and measure latency from your own infrastructure.

### Before you start

* Create API keys with the minimum permissions required.
* If withdrawals via API are supported, keep them disabled unless you explicitly need them.
