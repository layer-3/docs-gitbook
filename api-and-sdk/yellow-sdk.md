---
description: >-
  How the SDKs wrap REST and WebSocket, what languages are supported, and the
  safest way to get started.
hidden: true
---

# Yellow SDK

Yellow SDKs wrap REST + WebSocket and handle:

* authentication and signing
* retries and rate-limit backoff
* typed request/response objects (language dependent)
* streaming subscriptions

**Link:** SDK docs (docs.yellow.org)

### Supported languages

List SDKs here (example):

* TypeScript
* Python

### Install

Provide install commands per language.

**Screenshot:** Install snippet per language

### Minimal examples

#### Read markets

Example: fetch markets and parameters.

#### Place a limit order

Example: create order, read response, confirm status.

#### Listen for fills

Example: WS subscribe, handle fill events.

### Operational guidance

* run the SDK behind a stable clock (timestamp signing)
* store secrets in env vars
* use separate keys for production and testnet
* rotate keys periodically
