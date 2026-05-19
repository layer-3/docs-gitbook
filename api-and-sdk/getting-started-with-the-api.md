---
description: >-
  Set up an API key, fetch markets, place an order, and subscribe to WebSocket
  updates with the minimum required steps.
hidden: true
---

# Getting Started with the API

This guide covers the minimum steps to:

1. create an API key
2. fetch markets
3. place a limit order
4. listen for fills via WebSocket

For full endpoint docs, see **docs.yellow.org**.

### Step 1 — Create an API key

In the app:

* Go to **Settings → API Keys**
* Create a key
* Select permissions:
  * Read (market/account)
  * Trade (place/cancel orders)
  * Withdraw (only if required)

Store the secret securely. You won’t be able to view it again.

_Screenshot: API key creation screen with permission scopes_

### Step 2 — Check base URLs

Select the correct environment:

* Production base URL
* Testnet base URL (if available)

_Screenshot: Base URL shown in docs / settings_

### Step 3 — Fetch markets

Goal: list symbols, tick size, step size, min size.

Use:

* REST: `GET /markets` (example name; actual path in docs.yellow.org)

You need market parameters to format orders correctly.

_Screenshot: Markets response example (symbol + tick/step/min)_

### Step 4 — Place a limit order

Minimum fields:

* market (symbol)
* side (buy/sell)
* type (limit)
* size
* price

Common rejections:

* price not aligned to tick size
* size not aligned to step size
* below minimum size/notional
* insufficient available balance/margin

_Screenshot: Order request example + example “rejected” response_

### Step 5 — Subscribe to WebSocket user stream

Use WebSocket to receive:

* order updates (open/filled/canceled)
* fills
* position updates (perps)
* balance updates

Keep a reconnect loop and resubscribe on reconnect.

_Screenshot: WS subscription example_

### Safety recommendations

* Use idempotency keys for order placement if supported.
* Always verify order status after submission.
* Do not assume a request succeeded without checking the response.
* Implement rate-limit backoff and retry rules.
