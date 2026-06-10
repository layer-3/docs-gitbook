# README

## Overview

The Yellow platform provides REST and WebSocket APIs for authentication, trading, account management, and market data. There are two base URLs:

* **Authentication API** — wallet-based authentication, JWT tokens, and session management
* **Trading API** — trading, positions, transfers, market data, and the WebSocket stream

New here? Start with [**Getting Started with API**](getting-started-with-api.md) to authenticate and obtain your API keys.

## Base URLs

| API                          | Base URL                       |
| ---------------------------- | ------------------------------ |
| Authentication               | `https://auth.api.yellow.pro`  |
| Trading (REST + market data) | `https://trade.api.yellow.pro` |
| WebSocket                    | `wss://trade.api.yellow.pro`   |

{% hint style="info" %}
All examples in these docs use the production hosts above. Market data endpoints are served from the Trading API base URL and require no authentication.
{% endhint %}

## Contents

<table data-view="cards"><thead><tr><th>Section</th><th>Description</th><th data-card-target data-type="content-ref">Target</th></tr></thead><tbody><tr><td><strong>Getting Started</strong></td><td>Authentication flow, JWT and API key (HMAC) auth</td><td><a href="getting-started-with-api.md">getting-started-with-api.md</a></td></tr><tr><td><strong>API Key Management</strong></td><td>Create, list, and revoke API keys; scopes and IP allow-lists</td><td><a href="api-key-managment.md">api-key-managment.md</a></td></tr><tr><td><strong>Signing Requests with API Keys</strong></td><td>HMAC-SHA256 signature algorithm with Node.js and Python examples</td><td><a href="signing-requests-with-api-keys.md">signing-requests-with-api-keys.md</a></td></tr><tr><td><strong>Authentication Service API</strong></td><td>Challenge, verify, refresh, login, logout, /auth/me</td><td><a href="authentication-service-api.md">authentication-service-api.md</a></td></tr><tr><td><strong>Trading API Service</strong></td><td>Health, auth testing, and exchange market data</td><td><a href="trading-api-service.md">trading-api-service.md</a></td></tr><tr><td><strong>Market Data API</strong></td><td>Prices, momentum, std, VWMA (no auth required)</td><td><a href="market-data-api.md">market-data-api.md</a></td></tr><tr><td><strong>Spot Trading</strong></td><td>Spot accounts, orders, trades, deposits, withdrawals</td><td><a href="spot-trading-api.md">spot-trading-api.md</a></td></tr><tr><td><strong>Perpetuals Trading</strong></td><td>Positions, leverage, margin, perp orders, funding</td><td><a href="perpetuals-trading-api.md">perpetuals-trading-api.md</a></td></tr><tr><td><strong>Account Transfers</strong></td><td>Internal transfers and send history</td><td><a href="account-transfers-api.md">account-transfers-api.md</a></td></tr><tr><td><strong>WebSocket API</strong></td><td>Real-time streams and liquidation warnings</td><td><a href="websocket-api.md">websocket-api.md</a></td></tr><tr><td><strong>Error Handling</strong></td><td>Error response formats, codes, HTTP statuses</td><td><a href="error-handling-api.md">error-handling-api.md</a></td></tr><tr><td><strong>Reference</strong></td><td>Rate limits, data types, implementation notes</td><td><a href="reference.md">reference.md</a></td></tr></tbody></table>
