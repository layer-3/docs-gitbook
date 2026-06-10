# README

## Overview

The Yellow platform provides REST and WebSocket APIs for authentication, trading, account management, and market data. There are two base URLs:

* **Authentication API** — wallet-based authentication, JWT tokens, and session management
* **Trading API** — trading, positions, transfers, market data, and the WebSocket stream

New here? Start with [**Getting Started**](/broken/pages/598ade6681c73e0fa1dc10958cbbc77cc2459a7b) to authenticate and obtain your API keys.

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

<table data-view="cards"><thead><tr><th>Section</th><th>Description</th><th data-card-target data-type="content-ref">Target</th></tr></thead><tbody><tr><td><strong>Getting Started</strong></td><td>Authentication flow, JWT and API key (HMAC) auth</td><td><a href="/broken/pages/598ade6681c73e0fa1dc10958cbbc77cc2459a7b">Broken link</a></td></tr><tr><td><strong>API Key Management</strong></td><td>Create, list, and revoke API keys; scopes and IP allow-lists</td><td><a href="/broken/pages/ec147ad67071d832318aac1f934248e948963a53">Broken link</a></td></tr><tr><td><strong>Signing Requests with API Keys</strong></td><td>HMAC-SHA256 signature algorithm with Node.js and Python examples</td><td><a href="/broken/pages/bd0a002af558c9bbcc6292fe7c2b05b6876c7585">Broken link</a></td></tr><tr><td><strong>Authentication Service API</strong></td><td>Challenge, verify, refresh, login, logout, /auth/me</td><td><a href="/broken/pages/578f9763d2d40b78e48c8f2facfb43abbfed2f2a">Broken link</a></td></tr><tr><td><strong>Trading API Service</strong></td><td>Health, auth testing, and exchange market data</td><td><a href="/broken/pages/8b554b50ef9f96b851d891892a6275ef4b6591c8">Broken link</a></td></tr><tr><td><strong>Market Data API</strong></td><td>Prices, momentum, std, VWMA (no auth required)</td><td><a href="/broken/pages/7bdd7f76fdbc648389cfe547c41d95e65d3b6d3a">Broken link</a></td></tr><tr><td><strong>Spot Trading</strong></td><td>Spot accounts, orders, trades, deposits, withdrawals</td><td><a href="/broken/pages/d00fd8a1adf15936bec76105a8c1bbe57c4bb8f9">Broken link</a></td></tr><tr><td><strong>Perpetuals Trading</strong></td><td>Positions, leverage, margin, perp orders, funding</td><td><a href="/broken/pages/061f2de556a991ceddc49694e8fec148e2f4ed83">Broken link</a></td></tr><tr><td><strong>Account Transfers</strong></td><td>Internal transfers and send history</td><td><a href="/broken/pages/3fb59763b7f32913cbdd96d7cb0bc773c7b2b8a9">Broken link</a></td></tr><tr><td><strong>WebSocket API</strong></td><td>Real-time streams and liquidation warnings</td><td><a href="/broken/pages/8ad619a32b03586f611fad20989dbf7fa99f0f83">Broken link</a></td></tr><tr><td><strong>Error Handling</strong></td><td>Error response formats, codes, HTTP statuses</td><td><a href="/broken/pages/ed2858f2496393c43a527a581fcadfbf2aa99620">Broken link</a></td></tr><tr><td><strong>Reference</strong></td><td>Rate limits, data types, implementation notes</td><td><a href="/broken/pages/b93db56a4d5e4b2656bac7bec7c7292840b07205">Broken link</a></td></tr></tbody></table>
