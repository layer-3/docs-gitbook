---
description: >-
  A map of REST API groups, auth, rate limits, and error handling, with links to
  the full endpoint reference.
hidden: true
---

# REST API Reference

This page is a map of the REST API.\
For full paths, schemas, and examples, use **docs.yellow.org**.

**Link:** Full REST reference (docs.yellow.org)

### Common groups

#### Public (no auth)

* Markets
* Order book snapshots
* Trades
* Candles

#### Private (auth required)

* Account balances
* Open orders / order history
* Place / cancel orders
* Fills (trade history)
* Positions (perps)
* Funding history (perps)
* Fees and tiers
* Transfers (deposits/withdrawals)

### Authentication summary

Yellow uses API keys and may require request signing.

* Key is sent as a header
* Requests may require:
  * timestamp
  * signature over method + path + body

Exact rules are defined in docs.yellow.org.

### Rate limits

Requests are rate limited.

* If you receive a rate-limit error, back off and retry.
* Do not parallel spam order placement.

Exact thresholds are defined in docs.yellow.org.

### Error handling

Errors return:

* HTTP status code
* machine-readable error code
* message

Use error codes for handling, not message text.

**Screenshot:** Example error payload
