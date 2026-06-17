---
description: >-
  Public REST and WebSocket API reference for the Yellow trading platform:
  authentication, market data, spot, perpetuals, transfers, and clearnet.
---

# API Reference

## Overview

The Yellow platform provides comprehensive REST APIs for authentication, trading operations, account management, and market data access. The system consists of two main services:

- **Authentication Service**: Handles wallet-based authentication, JWT token management, and session handling
- **Trading API Service**: Provides trading operations, position management, and market data access

## Services & Base URLs

### Authentication Service

- **Development**: `http://localhost:8081`
- **UAT**: `https://auth.uat.assetum.neodax.app`
- **Production**: `https://auth.neodax.com`

### Trading API Service

- **Development**: `http://localhost:8086`
- **UAT**: `https://api.uat.assetum.neodax.app`
- **Production**: `https://api.neodax.com`

### Quote API Service

- **Development**: `http://localhost:8084`
- **Production**: `https://quotes.neodax.com`

### Trading WS Service
- **Development**: `ws://localhost:8086`
- **UAT**: `wss://api.uat.assetum.neodax.app`
- **Production**: `wss://api.neodax.com`

## Authentication Flow

Yellow uses Ethereum wallet-based authentication with a challenge-response mechanism:

1. **Request Challenge**: POST to `/auth/challenge` with wallet address
2. **Sign Challenge**: Sign the challenge text with your Ethereum wallet
3. **Verify Signature**: POST to `/auth/verify` with wallet address, challenge, and signature
4. **Receive Tokens**: Get JWT access token and refresh token for API access
5. **Use Access Token**: Include JWT in Authorization header for authenticated endpoints

```http
Authorization: Bearer <your-jwt-token>
```

### Security Notes

- Access tokens expire in 15 minutes (configurable)
- Refresh tokens expire in 7 days (configurable)
- Sessions can be invalidated via logout
- Rate limiting is enforced on authentication endpoints

### Authentication Methods

All authenticated endpoints support two authentication methods:

**1. JWT Token Authentication**

- Include JWT token in Authorization header: `Authorization: Bearer <token>`
- Obtained via `/auth/verify` endpoint
- Supported by all authenticated endpoints

**2. API Key Authentication**

- Include API key headers:
  - `X-API-Key`: Your API key
  - `X-Signature`: HMAC signature of the request
  - `X-Timestamp`: Request timestamp in milliseconds
- Supported by all authenticated endpoints (spot, perpetuals, transfers, clearnet)
- Automatically falls back to JWT if API key headers are not present

---

# Authentication Service API

## Health Endpoints

### GET /health

Check if the authentication service is running and healthy.

**Authentication**: Not required

**Response**:

```json
{
  "status": "UP"
}
```

**Status Codes**:

- `200` - Service is healthy
- `500` - Service is unhealthy

---

## Authentication Endpoints

### POST /auth/challenge

Generate a unique authentication challenge for wallet signature verification.

**Authentication**: Not required

**Request Body**:

```json
{
  "wallet_address": "0x1234567890abcdef1234567890abcdef12345678"
}
```

**Request Parameters**:

- `wallet_address` (string, required): Ethereum wallet address (0x-prefixed, 42 characters)

**Response**:

```json
{
  "challenge": "NeoDax Challenge: 1234567890abcdef, Timestamp: 2023-12-07T10:30:00Z, Nonce: 1234567890abcdef1234567890abcdef",
  "expires_at": "2023-12-07T10:35:00Z"
}
```

**Response Fields**:

- `challenge`: The challenge text that must be signed with your wallet
- `expires_at`: When the challenge expires (ISO 8601 format)

**Status Codes**:

- `200` - Challenge generated successfully
- `400` - Invalid wallet address or request format
- `429` - Rate limit exceeded (configurable per minute)
- `500` - Internal server error

### POST /auth/verify

Verify the signature of a challenge and issue JWT tokens.

**Authentication**: Not required

**Request Body**:

```json
{
  "wallet_address": "0x1234567890abcdef1234567890abcdef12345678",
  "challenge": "NeoDax Challenge: 1234567890abcdef, Timestamp: 2023-12-07T10:30:00Z, Nonce: 1234567890abcdef1234567890abcdef",
  "signature": "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef12"
}
```

**Request Parameters**:

- `wallet_address` (string, required): Same wallet address used for challenge
- `challenge` (string, required): The challenge text that was generated
- `signature` (string, required): Ethereum signature of the challenge (0x-prefixed, 132 characters)

**Response**:

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "rt_1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
  "expires_in": 900,
  "token_type": "Bearer"
}
```

**Response Fields**:

- `access_token`: JWT token for API authentication
- `refresh_token`: Token for refreshing the access token
- `expires_in`: Access token lifetime in seconds
- `token_type`: Always "Bearer"

**Status Codes**:

- `200` - Signature verified successfully, tokens issued
- `400` - Invalid request format
- `401` - Invalid signature or expired challenge
- `429` - Rate limit exceeded
- `500` - Internal server error

### POST /auth/refresh

Generate a new access token using a valid refresh token.

**Authentication**: Not required

**Request Body**:

```json
{
  "refresh_token": "rt_1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef"
}
```

**Request Parameters**:

- `refresh_token` (string, required): Valid refresh token (rt_ prefixed, 67 characters)

**Response**:

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "rt_9876543210fedcba9876543210fedcba9876543210fedcba9876543210fedcba",
  "expires_in": 900,
  "token_type": "Bearer"
}
```

**Status Codes**:

- `200` - Token refreshed successfully
- `400` - Invalid request format
- `401` - Invalid or expired refresh token
- `429` - Rate limit exceeded (20 requests per minute)
- `500` - Internal server error

### POST /auth/clearnet-login

Alternative authentication method for clearnet users, providing direct login without the challenge-response flow.

**Authentication**: Not required

**Request Body**:

```json
{
  "wallet_address": "0x1234567890abcdef1234567890abcdef12345678",
  "credentials": "user_credentials_data"
}
```

**Request Parameters**:

- `wallet_address` (string, required): Ethereum wallet address (0x-prefixed, 42 characters)
- `credentials` (string, required): Clearnet authentication credentials

**Response**:

```json
{
  "access_token": "<ACCESS_TOKEN>",
  "refresh_token": "<REFRESH_TOKEN>",
  "expires_in": 900,
  "token_type": "Bearer"
}
```

**Response Fields**:

- `access_token`: JWT token for API authentication
- `refresh_token`: Token for refreshing the access token
- `expires_in`: Access token lifetime in seconds
- `token_type`: Always "Bearer"

**Status Codes**:

- `200` - Authentication successful, tokens issued
- `400` - Invalid request format
- `401` - Invalid credentials
- `429` - Rate limit exceeded (configurable)
- `500` - Internal server error

### POST /auth/logout

Invalidate the current session by blacklisting it and revoking all associated refresh tokens.

**Authentication**: Required

**Response**:

```json
{
  "message": "Successfully logged out"
}
```

**Status Codes**:

- `200` - Successfully logged out
- `401` - Missing or invalid access token
- `429` - Rate limit exceeded
- `500` - Internal server error

### GET /auth/me

Retrieve information about the authenticated user from their JWT token.

**Authentication**: Required

**Response**:

```json
{
  "wallet_address": "0x1234567890abcdef1234567890abcdef12345678",
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "token_id": "550e8400-e29b-41d4-a716-446655440001",
  "issued_at": 1701938200,
  "expires_at": 1701939100
}
```

**Response Fields**:

- `wallet_address`: User's Ethereum wallet address
- `session_id`: Current session identifier
- `token_id`: Current token identifier (JTI)
- `issued_at`: Token issued timestamp (Unix epoch)
- `expires_at`: Token expiration timestamp (Unix epoch)

**Status Codes**:

- `200` - User information retrieved successfully
- `401` - Missing or invalid access token
- `429` - Rate limit exceeded
- `500` - Internal server error

---

# Trading API Service

## Health & System Endpoints

### GET /health

Check if the trading API service is running and healthy.

**Authentication**: Not required

**Response**:

```json
{
  "status": "UP"
}
```

**Status Codes**:

- `200` - Service is healthy
- `500` - Service is unhealthy

---

## Authentication Testing

### GET /test-auth

Test authentication functionality and retrieve user context from JWT token.

**Authentication**: Required

**Response**:

```json
{
  "message": "Authentication test successful",
  "wallet_address": "0x1234567890abcdef",
  "session_id": "session123",
  "token_id": "token123"
}
```

**Status Codes**:

- `200` - Authentication successful
- `401` - Authentication failed or token invalid

---

## Market Data

### GET /exchangeInfo

Retrieve exchange information including available markets and their details.

**Authentication**: Not required

**Response**:

```json
{
  "timezone": "UTC",
  "server_time": 1640995200000,
  "symbols": [
    {
      "symbol": "BTCUSD",
      "status": "TRADING",
      "base_asset": "BTC",
      "base_asset_precision": 8,
      "quote_asset": "USD",
      "quote_asset_precision": 8,
      "maker_fee_rate": 0.001,
      "taker_fee_rate": 0.002
    }
  ]
}
```

**Response Fields**:

- `timezone`: Exchange timezone
- `server_time`: Current server timestamp in milliseconds
- `symbols`: Array of available trading symbols
  - `symbol`: Trading symbol name
  - `status`: Market status (e.g., "TRADING")
  - `base_asset`: Base asset symbol
  - `base_asset_precision`: Decimal precision for base asset
  - `quote_asset`: Quote asset symbol
  - `quote_asset_precision`: Decimal precision for quote asset
  - `maker_fee_rate`: Maker fee rate
  - `taker_fee_rate`: Taker fee rate

**Status Codes**:

- `200` - Exchange information retrieved successfully
- `500` - Internal server error

### GET /orderbook

Retrieve order book data for a specific trading symbol.

**Authentication**: Not required

**Query Parameters**:

- `symbol` (string, required): Trading symbol to get order book for

**Request**:

```http
GET /orderbook?symbol=BTCUSD
```

**Response**:

```json
{
  "bids": [
    ["35000", "1.5"],
    ["34999", "2.0"]
  ],
  "asks": [
    ["35001", "1.2"],
    ["35002", "0.8"]
  ]
}
```

**Response Fields**:

- `bids`: Array of bid levels, each containing [price, amount]
- `asks`: Array of ask levels, each containing [price, amount]

**Status Codes**:

- `200` - Order book retrieved successfully
- `400` - Missing symbol parameter
- `404` - Symbol not found
- `500` - Internal server error

### GET /klines

Retrieve OHLC/candlestick data for a specific trading symbol. Compatible with Binance API format.

**Authentication**: Not required

**Query Parameters**:

- `symbol` (string, required): Trading symbol to get klines for
- `interval` (string, optional): Kline interval - `1m`, `3m`, `5m`, `15m`, `30m`, `1h`, `2h`, `4h`, `6h`, `8h`, `12h`, `1d`, `3d`, `1w`, `1M`
- `startTime` (integer, optional): Start time in milliseconds since Unix epoch
- `endTime` (integer, optional): End time in milliseconds since Unix epoch
- `limit` (integer, optional): Number of klines to return (default: 500, max: 1000)
- `timeZone` (string, optional): Timezone offset (e.g., "UTC", "+08:00")

**Request**:

```http
GET /klines?symbol=BTCUSD&interval=1h&limit=24
```

**Response**:

```json
[
  [
    1759224600000,
    "113216.5",
    "113300.1",
    "112888",
    "112945.1",
    "2602.44",
    41934
  ]
]
```

**Response Format** (each kline array contains):

- `[0]`: Open time (milliseconds)
- `[1]`: Open price
- `[2]`: High price
- `[3]`: Low price
- `[4]`: Close price
- `[5]`: Volume
- `[6]`: Number of trades

**Status Codes**:

- `200` - Klines retrieved successfully
- `400` - Invalid parameters or missing symbol
- `404` - Symbol not found
- `500` - Internal server error

### GET /ticker/24hr

Retrieve 24-hour ticker statistics for a specific trading symbol.

**Authentication**: Not required

**Query Parameters**:

- `symbol` (string, required): Trading symbol to get ticker for

**Request**:

```http
GET /ticker/24hr?symbol=BTCUSD
```

**Response**:

```json
{
  "marketId": "BTCUSD",
  "time": 1759228650754,
  "min": "111844.3",
  "max": "114188",
  "first": "112024.5",
  "last": "113964.6",
  "volume": "66457.75557923",
  "quoteVolume": "7528165635.780153719",
  "vwap": "113277.4582916087302791",
  "priceChange": "+1.73%"
}
```

**Response Fields**:

- `marketId`: Market identifier (e.g., BTCUSD)
- `time`: Timestamp of the data in Unix milliseconds
- `min`: Lowest price in the period
- `max`: Highest price in the period
- `first`: First price in the period (open price)
- `last`: Last trade price
- `volume`: Total traded volume in the base currency
- `quoteVolume`: Total traded volume in the quote currency
- `vwap`: Volume-weighted average price
- `priceChange`: Price change percentage over the period

**Status Codes**:

- `200` - Ticker retrieved successfully
- `400` - Missing symbol parameter
- `404` - Symbol not found
- `500` - Internal server error

### GET /trading-parameters/insurance-fund/history

Retrieve daily total-equity history for the perpetuals insurance fund account. Data is sourced from Analytics `portfolio_equity_snapshots` (field `total_equity`, USD-aggregated equity). One point per calendar day in `Asia/Shanghai`, anchored at local **08:00** (mapped to UTC `bucket_hour`). If the anchor hour has no snapshot, the latest prior snapshot is used (forward-fill). The `current` field is the globally latest snapshot for the insurance fund account and is not limited to the requested date range.

**Authentication**: Not required

**Query Parameters** (provide one of the following):

- `range` (string, optional): Preset window — `7d` (last 7 Shanghai calendar days, inclusive of today) or `30d` (last 30 days). When set, `start` and `end` are ignored.
- `start` (string, optional): Start date `YYYY-MM-DD` in `Asia/Shanghai`. Required with `end` when `range` is omitted. Inclusive.
- `end` (string, optional): End date `YYYY-MM-DD` in `Asia/Shanghai`. Required with `start` when `range` is omitted. Inclusive. Custom ranges are limited to 400 days on the Analytics service.

**Request**:

```http
GET /trading-parameters/insurance-fund/history?range=7d
```

```http
GET /trading-parameters/insurance-fund/history?start=2026-04-01&end=2026-04-23
```

**Response**:

```json
{
  "anchor": {
    "timezone": "Asia/Shanghai",
    "local_time": "08:00:00"
  },
  "equity_field": "total_equity",
  "points": [
    {
      "day": "2026-04-23",
      "as_of": "2026-04-23T08:00:00+08:00",
      "total_equity_usd": "12345.67"
    },
    {
      "day": "2026-04-22",
      "as_of": "2026-04-22T08:00:00+08:00",
      "total_equity_usd": null
    }
  ],
  "current": {
    "as_of": "2026-04-23T09:00:00Z",
    "total_equity_usd": "13000"
  }
}
```

**Response Fields**:

- `anchor`: Daily sampling anchor metadata
  - `timezone`: Calendar timezone (`Asia/Shanghai`)
  - `local_time`: Local anchor time (`08:00:00`)
- `equity_field`: Snapshot column used for values (always `total_equity`)
- `points`: Array of daily points (one per Shanghai calendar day in the requested range)
  - `day`: Calendar date `YYYY-MM-DD` (Shanghai)
  - `as_of`: RFC3339 instant of the local 08:00 anchor for that day
  - `total_equity_usd`: Decimal string; `null` when no snapshot is available for that day (including when forward-fill finds no prior row in the fetch window)
- `current` (optional): Latest snapshot row for the insurance fund account
  - `as_of`: RFC3339 `bucket_hour` of the latest snapshot
  - `total_equity_usd`: Decimal string

**Notes**:

- Requires Trading API `analytics_grpc_address` (Analytics gRPC). If Analytics is not configured, the endpoint returns `503`.
- Insurance fund account identifiers are fixed platform constants (`app_session_id` `00000000-0000-0000-0000-000000000002`); this endpoint does not accept wallet or session parameters.

**Status Codes**:

- `200` - History retrieved successfully
- `400` - Invalid or missing query parameters (`range` must be `7d` or `30d` when set; otherwise both `start` and `end` are required)
- `502` - Analytics service error
- `503` - Analytics gRPC not configured on this Trading API instance

---

# Quote Service API

The Quote Service provides real-time market data endpoints for prices, technical indicators, and market analysis data.

## Health Endpoints

### GET /health

Check if the quote service is running and healthy.

**Authentication**: Not required

**Response**:

```json
{
  "status": "UP"
}
```

**Status Codes**:

- `200` - Service is healthy
- `500` - Service is unhealthy

---

## Market Data Endpoints

### GET /data/price

Retrieve the latest price observation for all markets.

**Authentication**: Not required

**Response**:

```json
{
  "header": {
    "event_id": "550e8400-e29b-41d4-a716-446655440000",
    "timestamp": "2023-12-07T10:30:00Z"
  },
  "market": "BTCUSD",
  "price": "35000.50000000"
}
```

**Response Fields**:

- `header`: Event metadata
  - `event_id`: Unique event identifier
  - `timestamp`: Event timestamp
- `market`: Market symbol (e.g., "BTCUSD")
- `price`: Latest observed price

**Status Codes**:

- `200` - Latest price retrieved successfully
- `204` - No price data available
- `500` - Internal server error

### GET /data/momentum

Retrieve the latest momentum indicator data for all markets.

**Authentication**: Not required

**Request**:

```http
GET /data/momentum
```

**Response**:

```json
{
  "Market": {
    "Base": "btc",
    "Quote": "usd"
  },
  "Value": "0.0023",
  "Time": "2025-09-30T15:40:54.455631184Z"
}
```

**Response Fields**:

- `Market`: Market information
  - `Base`: Base currency (e.g., "btc")
  - `Quote`: Quote currency (e.g., "usd")
- `Value`: Momentum indicator value (decimal)
- `Time`: Timestamp when the indicator was computed (ISO 8601 format)

**Status Codes**:

- `200` - Latest momentum retrieved successfully
- `204` - No momentum data available
- `500` - Internal server error

### GET /data/std

Retrieve the latest moving standard deviation data for all markets.

**Authentication**: Not required

**Request**:

```http
GET /data/std
```

**Response**:

```json
{
  "Market": {
    "Base": "btc",
    "Quote": "usd"
  },
  "Value": "145.67",
  "Time": "2025-09-30T15:40:54.455631184Z"
}
```

**Response Fields**:

- `Market`: Market information
  - `Base`: Base currency (e.g., "btc")
  - `Quote`: Quote currency (e.g., "usd")
- `Value`: Moving standard deviation value (decimal)
- `Time`: Timestamp when the indicator was computed (ISO 8601 format)

**Status Codes**:

- `200` - Latest standard deviation retrieved successfully
- `204` - No standard deviation data available
- `500` - Internal server error

### GET /data/vwma

Retrieve the latest volume-weighted moving average (VWMA) data for all markets.

**Authentication**: Not required

**Request**:

```http
GET /data/vwma
```

**Response**:

```json
{
  "Market": {
    "Base": "btc",
    "Quote": "usd"
  },
  "Value": "35250.45",
  "Time": "2025-09-30T15:40:54.455631184Z"
}
```

**Response Fields**:

- `Market`: Market information
  - `Base`: Base currency (e.g., "btc")
  - `Quote`: Quote currency (e.g., "usd")
- `Value`: Volume-weighted moving average value (decimal)
- `Time`: Timestamp when the indicator was computed (ISO 8601 format)

**Status Codes**:

- `200` - Latest VWMA retrieved successfully
- `204` - No VWMA data available
- `500` - Internal server error

---

## Spot Trading

The spot trading API provides endpoints for managing spot trading accounts, placing and canceling orders, and retrieving trade history.

### GET /spot/exchangeInfo

Retrieve exchange information including available spot markets and their details.

**Authentication**: Not required

**Response**:

```json
{
  "timezone": "UTC",
  "server_time": 1640995200000,
  "symbols": [
    {
      "symbol": "BTCYTEST.USD",
      "status": "active",
      "base_asset": "BTC",
      "base_asset_precision": 8,
      "quote_asset": "YTEST.USD",
      "quote_asset_precision": 8
    }
  ]
}
```

**Response Fields**:

- `timezone`: Exchange timezone
- `server_time`: Current server timestamp in milliseconds
- `symbols`: Array of available spot trading symbols
  - `symbol`: Trading symbol name (e.g., "BTCYTEST.USD")
  - `status`: Market status (e.g., "active")
  - `base_asset`: Base asset symbol (e.g., "BTC")
  - `base_asset_precision`: Decimal precision for base asset
  - `quote_asset`: Quote asset symbol (e.g., "YTEST.USD")
  - `quote_asset_precision`: Decimal precision for quote asset

**Status Codes**:

- `200` - Exchange information retrieved successfully
- `500` - Internal server error

---

### GET /spot/networks

Retrieve supported blockchain networks and the tokens available for deposit and withdrawal on each network.

**Authentication**: Not required

**Response**:

```json
{
  "networks": [
    {
      "chain_id": 1,
      "name": "ethereum",
      "custody_address": "0x5FbDB2315678afecb367f032d93F642f64180aa3",
      "tokens": [
        {
          "symbol": "ETH",
          "contract_address": "0x0000000000000000000000000000000000000000",
          "decimals": 18,
          "can_deposit": true,
          "can_withdraw": true
        },
        {
          "symbol": "USDC",
          "contract_address": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
          "decimals": 6,
          "can_deposit": true,
          "can_withdraw": true
        }
      ]
    },
    {
      "chain_id": 42161,
      "name": "arbitrum",
      "custody_address": "0xABC123def456789012345678901234567890abcd",
      "tokens": [
        {
          "symbol": "ETH",
          "contract_address": "0x0000000000000000000000000000000000000000",
          "decimals": 18,
          "can_deposit": true,
          "can_withdraw": true
        }
      ]
    }
  ]
}
```

**Response Fields**:

- `networks`: Array of supported blockchain networks
  - `chain_id`: EVM chain ID (e.g., 1 for Ethereum mainnet, 42161 for Arbitrum)
  - `name`: Human-readable network name
  - `custody_address`: Cage custody contract address on this network (deposit destination)
  - `tokens`: Array of tokens available on this network
    - `symbol`: Asset symbol (e.g., "ETH", "USDC")
    - `contract_address`: Token ERC-20 contract address (`0x000...000` for native ETH)
    - `decimals`: On-chain token decimals
    - `can_deposit`: Whether deposits are currently enabled for this token on this network
    - `can_withdraw`: Whether withdrawals are currently enabled for this token on this network

**Status Codes**:

- `200` - Networks retrieved successfully

---

### GET /spot/accounts

Retrieve all spot accounts for the authenticated user.

**Authentication**: Required

**Response**:

```json
[
  {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "app_session_id": "spot_session_123",
    "owner": "0x1234567890abcdef1234567890abcdef12345678",
    "balances": [
      {
        "asset_symbol": "BTC",
        "total_balance": "1.50000000",
        "available_balance": "1.20000000",
        "locked_balance": "0.30000000",
        "total_balance_usd": "169343.931544",
        "available_balance_usd": "135475.145235",
        "locked_balance_usd": "33868.786309",
        "last_updated": "2023-12-07T10:30:00.000000Z"
      },
      {
        "asset_symbol": "USDT",
        "total_balance": "10000.00000000",
        "available_balance": "8500.00000000",
        "locked_balance": "1500.00000000",
        "total_balance_usd": "10000.00000000",
        "available_balance_usd": "8500.00000000",
        "locked_balance_usd": "1500.00000000",
        "last_updated": "2023-12-07T10:30:00.000000Z"
      }
    ],
    "state": "active",
    "opened_at": "2023-12-01T08:00:00.000000Z"
  }
]
```

**Response Fields**:

- `id`: Unique spot account identifier (UUID)
- `app_session_id`: Application session identifier
- `owner`: Owner's Ethereum wallet address
- `balances`: Array of balance objects
  - `asset_symbol`: Asset symbol (e.g., "BTC", "USDT")
  - `total_balance`: Total balance (available + locked)
  - `available_balance`: Balance available for trading
  - `locked_balance`: Balance locked in open orders
  - `total_balance_usd`: Total balance valued in stablecoin (USD/USDT); `"0"` when no price
  - `available_balance_usd`: Available balance valued in stablecoin; `"0"` when no price
  - `locked_balance_usd`: Locked balance valued in stablecoin; `"0"` when no price
  - `last_updated`: Last balance update timestamp
- `state`: Account state (`active`, `closed`)
- `opened_at`: Account creation timestamp

**Status Codes**:

- `200` - Accounts retrieved successfully
- `401` - Authentication failed
- `500` - Internal server error

### GET /spot/account

Retrieve information for a specific spot account.

**Authentication**: Required

**Query Parameters**:

- `app_session_id` (string, required): Spot account app session ID
- `asset` (string, optional): Case-insensitive fuzzy filter on `asset_symbol` (contains match)

**Request**:

```http
GET /spot/account?app_session_id=spot_session_123
```

**Response**:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "app_session_id": "spot_session_123",
  "owner": "0x1234567890abcdef1234567890abcdef12345678",
  "balances": [
    {
      "asset_symbol": "BTC",
      "total_balance": "1.50000000",
      "available_balance": "1.20000000",
      "locked_balance": "0.30000000",
      "total_balance_usd": "169343.931544",
      "available_balance_usd": "135475.145235",
      "locked_balance_usd": "33868.786309",
      "last_updated": "2023-12-07T10:30:00.000000Z"
    }
  ],
  "state": "active",
  "opened_at": "2023-12-01T08:00:00.000000Z"
}
```

**Status Codes**:

- `200` - Account retrieved successfully
- `400` - Missing app_session_id parameter
- `401` - Authentication failed
- `404` - Account not found
- `500` - Internal server error

### POST /spot/order

Create a new spot trading order.

**Authentication**: Required

When using the **sync gRPC lock** path (`LockSpotAsset` then Kafka `CommandOrderExecute`), if Kafka publish fails **after** PM lock succeeds, Trading API calls `UnlockSpotAsset` as compensation (best effort). The HTTP **`400`** body then uses **`error`: `publish_failed`**, **`message`**, and **`rollback_status`**: `ok` or `failed` (same shape as **`POST /perpetual/order`** publish failures).

**Request Body**:

```json
{
  "app_session_id": "spot_session_123",
  "market": "BTCUSDT",
  "side": "buy",
  "type": "limit",
  "amount": "0.5",
  "price": "35000",
  "time_in_force": "gtc"
}
```

**Request Parameters**:

- `app_session_id` (string, required): Spot account app session ID
- `market` (string, required): Trading market (e.g., "BTCUSDT", "ETHUSDC")
- `side` (string, required): Order side - `buy` or `sell`
- `type` (string, required): Order type - `limit` or `market`
- `amount` (string, required): Order amount in base asset (decimal format)
- `price` (string, optional): Limit price (required for `limit` orders, ignored for `market` orders)
- `time_in_force` (string, optional): Time in force - `gtc` (Good-Till-Cancelled), `ioc` (Immediate-Or-Cancel), `fok` (Fill-Or-Kill). Defaults to `GTC` for `limit` orders, `IOC` for `market` orders.

**Response**:

```json
{
  "order_uuid": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Response Fields**:

- `order_uuid`: UUID of the created order

**Status Codes**:

- `200` - Order created successfully
- `400` - Invalid request parameters, validation failed, Portfolio Manager rejected the spot lock **before** publish, or **Kafka publish failed after a successful lock** (see publish-failure body below)
- `401` - Authentication failed
- `500` - Internal server error

**Error response** (typical for `400` — lock or validation):

```json
{
  "error": "insufficient_balance",
  "message": "failed to lock funds: validation failed: insufficient available balance for lock: ..."
}
```

- `error`: Machine-readable code (`snake_case`). When the failure comes from the **spot** Portfolio Manager **lock** step (`LockSpotAsset`), this equals the gRPC `error_code` (see **Common Error Codes** → **Trading API Service** → `LockSpotAssetResponse`). If PM returns `success=false` with an empty `error_code`, the gateway uses `lock_funds_rejected`. Other gateway or request validation failures may still use `order_creation_failed`. **If PM lock succeeded but publishing `CommandOrderExecute` to Kafka failed**, the gateway returns **`error`: `publish_failed`** with **`rollback_status`** (see next subsection), not `order_creation_failed`.
- `message`: Human-readable detail (often the PM `message` or a wrapped error string).

**Error response** (`400` — **publish failed after successful PM lock**):

```json
{
  "error": "publish_failed",
  "message": "<underlying Kafka / publisher error>",
  "rollback_status": "ok"
}
```

- `rollback_status`: `ok` if `UnlockSpotAsset` compensation completed (or unlock was not required for this path); `failed` if all configured unlock retries failed.
- **`rollback_status` is not** included when the failure occurs **before** publish (e.g. lock rejection, request validation); those responses use the lock/validation shape above.

### DELETE /spot/order

Cancel an existing spot order.

**Authentication**: Required

**Request Body**:

```json
{
  "app_session_id": "spot_session_123",
  "market": "BTCUSDT",
  "order_uuid": "550e8400-e29b-41d4-a716-446655440000",
  "type": "limit"
}
```

**Request Parameters**:

- `app_session_id` (string, required): Spot account app session ID
- `market` (string, required): Trading market
- `order_uuid` (string, required): UUID of the order to cancel
- `type` (string, required): Order type

**Response**:

```json
{
  "message": "Spot order cancellation request sent successfully"
}
```

**Status Codes**:

- `200` - Cancellation request sent successfully
- `400` - Invalid request or missing parameters
- `401` - Authentication failed
- `500` - Internal server error

### GET /spot/open_orders

Retrieve open spot orders with pagination.

**Authentication**: Required

**Query Parameters**:

- `app_session_id` (string, required): Spot account app session ID
- `asset` (string, optional): Case-insensitive fuzzy filter on `asset_symbol` (contains match)
- `market` (string, optional): Filter by trading market
- `page` (integer, optional): Page number (default: 1)
- `page_size` (integer, optional): Number of orders per page (default: 50, max: 100)

**Request**:

```http
GET /spot/open_orders?app_session_id=spot_session_123&market=BTCUSDT&page=1&page_size=20
```

**Response**:

```json
{
  "orders": [
    {
      "id": "1234",
      "order_id": "550e8400-e29b-41d4-a716-446655440000",
      "channel_id": "spot_session_123",
      "market": "BTCUSDT",
      "price": "35000.00000000",
      "amount": "0.50000000",
      "origin_amount": "0.50000000",
      "notional": "17500.00000000",
      "side": "buy",
      "type": "limit",
      "state": "wait",
      "event": "",
      "reason": "",
      "created_at": "2023-12-07T10:30:00.000000Z",
      "updated_at": "2023-12-07T10:30:00.000000Z",
      "completed_at": ""
    }
  ],
  "total": 1,
  "page": 1,
  "page_size": 20
}
```

**Response Fields**:

- `orders`: Array of order objects
  - `id`: Internal order record ID
  - `order_id`: Order UUID
  - `channel_id`: Spot account app session ID
  - `market`: Trading market
  - `price`: Order price
  - `amount`: Current order amount (may be partially filled)
  - `origin_amount`: Original order amount
  - `notional`: Notional value (amount x price)
  - `side`: Order side (`buy` or `sell`)
  - `type`: Order type (`limit` or `market`)
  - `state`: Order state (`wait`, `done`, `canceled`)
  - `event`: Last order event (empty string)
  - `reason`: Reason for last state change (empty string)
  - `created_at`: Order creation timestamp
  - `updated_at`: Last update timestamp
  - `completed_at`: Completion timestamp (empty for open orders)
- `total`: Total number of matching orders
- `page`: Current page number
- `page_size`: Number of orders per page

**Status Codes**:

- `200` - Orders retrieved successfully
- `400` - Invalid query parameters (including malformed `cursor`)
- `401` - Authentication failed
- `500` - Internal server error

### GET /spot/orders

Retrieve spot order history with pagination.

**Authentication**: Required

**Query Parameters**:

- `app_session_id` (string, required): Spot account app session ID
- `market` (string, optional): Filter by trading market
- `page` (integer, optional): Page number (default: 1)
- `page_size` (integer, optional): Number of orders per page (default: 50, max: 100)

**Request**:

```http
GET /spot/orders?app_session_id=spot_session_123&page=1&page_size=50
```

**Response**: Same format as `/spot/open_orders` but includes all orders (open, filled, and cancelled)

**Status Codes**:

- `200` - Orders retrieved successfully
- `400` - Invalid query parameters (including malformed `cursor`)
- `401` - Authentication failed
- `500` - Internal server error

### GET /spot/trades

Retrieve spot trade history with pagination.

**Authentication**: Required

**Query Parameters**:

- `app_session_id` (string, required): Spot account app session ID
- `market` (string, optional): Filter by trading market
- `page` (integer, optional): Page number (default: 1)
- `page_size` (integer, optional): Number of trades per page (default: 50, max: 100)

**Request**:

```http
GET /spot/trades?app_session_id=spot_session_123&market=BTCUSDT&page=1&page_size=50
```

**Response**:

```json
{
  "trades": [
    {
      "id": "5721092",
      "order_id": "12345",
      "market": "BTCUSDT",
      "amount": "0.50000000",
      "price": "35000.00000000",
      "is_buyer": true,
      "is_maker": false,
      "executed_at": "2023-12-07T10:30:00.000000Z",
      "created_at": "2023-12-07T10:30:01.000000Z"
    }
  ],
  "total": 1,
  "page": 1,
  "page_size": 50
}
```

**Response Fields**:

- `trades`: Array of trade objects
  - `id`: Internal trade record ID (string)
  - `order_id`: User's own order ID in this trade (string, numeric)
  - `market`: Trading market
  - `amount`: Trade amount in base asset
  - `price`: Trade execution price
  - `is_buyer`: Whether the user was the buyer in this trade (boolean)
  - `is_maker`: Whether the user was the maker (liquidity provider) in this trade (boolean)
  - `executed_at`: Trade execution timestamp
  - `created_at`: Trade record creation timestamp
- `total`: Total number of matching trades
- `page`: Current page number
- `page_size`: Number of trades per page

**Note**: The API response only exposes the user's own order ID and does not expose counterparty information for privacy reasons. It provides user-relative flags (`is_buyer`, `is_maker`) to indicate the user's role in the trade.

**Status Codes**:

- `200` - Trades retrieved successfully
- `400` - Invalid query parameters (including malformed `cursor`)
- `401` - Authentication failed
- `500` - Internal server error

### GET /spot/deposits

Retrieve spot deposit history.

**Authentication**: Required

**Query Parameters**:

- `app_session_id` (string, required): Spot account app session ID

**Request**:

```http
GET /spot/deposits?app_session_id=spot_session_123
```

**Response**:

```json
{
  "deposits": [
    {
      "spot_account_id": "550e8400-e29b-41d4-a716-446655440001",
      "asset_symbol": "USDT",
      "amount": "10000.00000000",
      "transaction_hash": "0xabcdef1234567890",
      "created_at": "2023-12-07T10:30:01.000000Z",
      "deposited_at": "2023-12-07T10:30:00.000000Z",
      "chain_id": 1
    }
  ],
  "total": 6,
  "page": 1,
  "page_size": 50
}
```

**Response Fields**:

- `deposits`: Array of deposit objects
  - `spot_account_id`: Spot account ID (UUID)
  - `asset_symbol`: Asset symbol (e.g., "USDT", "BTC")
  - `amount`: Deposit amount
  - `transaction_hash`: ClearNet transaction hash
  - `created_at`: Record creation timestamp
  - `deposited_at`: Timestamp when deposit occurred
  - `chain_id`: Blockchain chain ID where the deposit was received (e.g., 1 for Ethereum, 42161 for Arbitrum)
- `total`: Total number of deposits matching criteria
- `page`: Current page number
- `page_size`: Number of deposits per page

**Status Codes**:

- `200` - Deposits retrieved successfully
- `400` - Missing app_session_id parameter
- `401` - Authentication failed
- `500` - Internal server error

### GET /spot/withdrawals

Retrieve spot withdrawal history.

**Authentication**: Required

**Query Parameters**:

- `app_session_id` (string, required): Spot account app session ID

**Request**:

```http
GET /spot/withdrawals?app_session_id=spot_session_123
```

**Response**:

```json
{
  "withdrawals": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "spot_account_id": "550e8400-e29b-41d4-a716-446655440001",
      "asset_symbol": "USDT",
      "amount": "5000.00000000",
      "status": "completed",
      "transaction_hash": "0xabcdef1234567890",
      "failure_reason": "",
      "requested_at": "2023-12-07T10:30:00.000000Z",
      "completed_at": "2023-12-07T10:31:00.000000Z",
      "created_at": "2023-12-07T10:30:01.000000Z",
      "chain_id": 1
    }
  ],
  "total": 0,
  "page": 1,
  "page_size": 50
}
```

**Response Fields**:

- `withdrawals`: Array of withdrawal objects
  - `id`: Withdrawal ID (UUID)
  - `spot_account_id`: Spot account ID (UUID)
  - `asset_symbol`: Asset symbol (e.g., "USDT", "BTC")
  - `amount`: Withdrawal amount
  - `status`: Withdrawal status (`pending`, `completed`, `failed`)
  - `transaction_hash`: ClearNet transaction hash (when completed)
  - `failure_reason`: Reason for failure (when status is `failed`)
  - `requested_at`: Timestamp when withdrawal was requested
  - `completed_at`: Timestamp when withdrawal completed (or failed)
  - `created_at`: Record creation timestamp
  - `chain_id`: Blockchain chain ID where the withdrawal was executed (e.g., 1 for Ethereum, 42161 for Arbitrum)
- `total`: Total number of withdrawals matching criteria
- `page`: Current page number
- `page_size`: Number of withdrawals per page

**Status Codes**:

- `200` - Withdrawals retrieved successfully
- `400` - Missing app_session_id parameter
- `401` - Authentication failed
- `500` - Internal server error

### POST /spot/withdrawal

Request a new spot withdrawal.

**Authentication**: Required

> **Known Issue**: Server returns `500` instead of `400` when withdrawal amount exceeds available balance.

**Request Body**:

```json
{
  "app_session_id": "spot_session_123",
  "asset_symbol": "USDT",
  "amount": "5000.00000000",
  "chain_id": 1
}
```

**Request Parameters**:

- `app_session_id` (string, required): Spot account app session ID
- `asset_symbol` (string, required): Asset to withdraw (e.g., "USDT", "BTC")
- `amount` (string, required): Amount to withdraw (decimal format, must be positive)
- `chain_id` (integer, optional): Target blockchain chain ID for the withdrawal (e.g., 1 for Ethereum, 42161 for Arbitrum). Use `GET /spot/networks` to discover available chains. When omitted, falls back to the server-configured default chain. Returns `400` if omitted and no default is configured.

**Response**:

```json
{
  "withdrawal_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "pending",
  "message": "Withdrawal request accepted and funds reserved"
}
```

**Response Fields**:

- `withdrawal_id`: UUID of the withdrawal request
- `status`: Initial status (`pending`)
- `message`: Confirmation message

**Status Codes**:

- `200` - Withdrawal request accepted
- `400` - Invalid request or insufficient funds
- `401` - Authentication failed
- `500` - Internal server error

---

## Perpetuals Trading

The perpetuals trading API provides endpoints for managing cross-margin perpetual futures accounts, placing and canceling orders, monitoring positions, and retrieving trade history. All perpetuals accounts use cross-margin model where collateral is shared across all positions.

### Perpetuals Calculation Formulas

For frontend display (margin ratio, break-even price, ROI %, etc.), see `perpetuals-frontend-formulas-from-code.md`. Key formulas:

- **Cross margin ratio**: `margin_ratio = total_account_equity / total_maintenance_margin` (from `GET /perpetual/account` or `GET /perpetual/accounts`)
- **Break-even price**: Long `entry_price - realized_pnl/amount`, Short `entry_price + realized_pnl/amount` (derived, not returned)
- **Liquidation price**: Returned by API and WebSocket `position_update` for cross-margin positions

---

### Perpetual list pagination (Phase A dual-mode)

Most perpetual **list** endpoints support **two pagination modes** during the Phase A rollout. Legacy clients can keep using `page` + `page_size`; new clients may use opaque **cursor** pagination (the cursor path avoids `COUNT(*)` except when explicitly requested).

**Dispatch** (server chooses the read path):

| Client sends | Path |
|---|---|
| `use_cursor=true`, or non-empty `cursor` with `page` omitted / `0` | **Cursor** — follow `next_cursor` / `has_more`; response `page` is `0` |
| `page` ≥ 1 | **Legacy offset** — `total` + `page` unchanged; `next_cursor` / `has_more` included as migration hints |
| Neither `page` nor `cursor` | **Cursor first page** (default) |

**Shared query parameters** (where noted on each endpoint):

- `cursor` (string, optional): Opaque token from previous `next_cursor`. Pass unchanged.
- `use_cursor` (boolean, optional): Force cursor mode even when `page` would select offset mode.
- `compute_total` (boolean, optional): Cursor path only — optionally compute `total` (expensive). Ignored on transaction history (`total` is always `null`).

**Shared response fields**:

- `next_cursor` (string): Next-page token; empty when `has_more` is `false`.
- `has_more` (boolean): Authoritative “another page exists” signal.

**Errors**: Malformed `cursor` → `400` with `"error": "invalid_cursor"`.

**Cursor encoding** (opaque to clients; for debugging only):

| Endpoint | Cursor shape |
|---|---|
| `GET /perpetual/orders`, `GET /perpetual/open_orders` | `<order_row_id>` (decimal int64 string) |
| `GET /perpetual/trades` | `<executed_at_unix_ms>:<trade_id>` |
| `GET /perpetual/position-history` | `<sort_ts_unix_ms>:<position_id>` (`sort_by` selects opened_at vs closed_at) |
| `GET /perpetual/funding-rates` | `<interval_end_unix_ms>:<funding_rate_id>` |
| `GET /perpetual/transaction/history` | `<ledger_ts_unix_ms>:<transaction_id>:<asset>` |


---

### GET /perpetual/exchangeInfo

Retrieve exchange information for perpetuals markets including available markets and their details.

**Authentication**: Not required

**Response**:

```json
{
  "timezone": "UTC",
  "server_time": 1640995200000,
  "symbols": [
    {
      "symbol": "BTCYTEST.USD-PERP",
      "status": "TRADING",
      "base_asset_name": "Bitcoin",
      "base_asset": "BTC",
      "quote_asset_name": "US Dollar",
      "quote_asset": "USD",
      "amount_precision": 8,
      "price_precision": 8,
      "amount_display_precision": 8,
      "price_display_precision": 8,
      "maker_fee_rate": "0.001",
      "taker_fee_rate": "0.002",
      "max_allowed_leverage": "100",
      "maintenance_margin_rate": "0.007",
      "filters": [
        {"filter_type": "PRICE_FILTER", "config": {"tick_size": "0.01", "price_min_ratio": "0.9", "price_max_ratio": "1.1"}},
        {"filter_type": "LOT_SIZE", "config": {"step_size": "0.001", "min_qty": "0.001", "max_qty": "999999999"}},
        {"filter_type": "MIN_NOTIONAL", "config": {"min_notional": "1"}},
        {"filter_type": "DEPTH_MERGE", "config": {"depth_level": ["0.001", "0.01", "0.1", "1"]}}
      ]
    }
  ]
}
```

**Response Fields**:

- `timezone`: Exchange timezone
- `server_time`: Current server timestamp in milliseconds
- `symbols`: Array of available perpetuals markets
  - `symbol`: Trading symbol name (e.g., "BTCYTEST.USD-PERP")
  - `status`: Market status (e.g., "TRADING")
  - `base_asset_name`: Display name for base asset
  - `base_asset`: Base asset symbol (e.g., "BTC")
  - `quote_asset_name`: Display name for quote asset
  - `quote_asset`: Quote asset symbol (e.g., "USD")
  - `amount_precision`: Decimal precision for amount (base asset)
  - `price_precision`: Decimal precision for price (quote asset)
  - `amount_display_precision`: Display precision for amount in UI
  - `price_display_precision`: Display precision for price in UI
  - `maker_fee_rate`: Maker fee rate as string (e.g., "0.001")
  - `taker_fee_rate`: Taker fee rate as string (e.g., "0.002")
  - `max_allowed_leverage`: Max allowed leverage as string (e.g., "100")
  - `maintenance_margin_rate`: Maintenance margin rate (MMR) for this market as decimal string (e.g., `"0.007"` = 0.7%). From PM market config; tiered maintenance on positions may still compute `maintenance_margin` from brackets.
  - `filters`: Trading rules (PRICE_FILTER, LOT_SIZE, MIN_NOTIONAL, DEPTH_MERGE)

**Status Codes**:

- `200` - Exchange information retrieved successfully
- `500` - Internal server error

---

### GET /perpetual/market-risk-tiers

Retrieve **per-market risk tier brackets** (quote **notional** cap per bracket, max leverage, initial / maintenance margin rates) as served by the perpetuals Portfolio Manager. Data reflects the in-memory ladder loaded from `perps_market_risk_tiers` when present, or a **synthetic single tier** derived from `perps_markets` when no table rows exist for that symbol (`from_config_table: false`).

**Authentication**: Not required

**Query Parameters**:

- `symbol` (string, optional): Exact perpetual market symbol (e.g. `BTCUSDT-PERP`), matching `symbol` from `GET /perpetual/exchangeInfo`. Whitespace is trimmed. If omitted, the response includes tiers for **all active** markets (multiple symbols in one `tiers` array).

**Caching (Trading API)**:

- Responses are backed by Redis key `trading_api:perp_risk_tiers:v1`, which stores a **full snapshot** (same shape as the no-`symbol` response). When `symbol` is provided, the gateway **filters** cached rows where `tier.symbol` equals the query value (exact match, case-sensitive).
- TTL is set by Nacos `perp_risk_tier_cache_ttl` on the trading-api service. If unset, zero, or negative, the default TTL is **60 seconds**.
- Redis read errors are ignored and the gateway refetches from PM; invalid JSON in Redis is discarded and refetched. `**404` responses are not cached.**

**Example** (single market):

```
GET /perpetual/market-risk-tiers?symbol=BTCUSDT-PERP
```

**Example** (all active markets):

```
GET /perpetual/market-risk-tiers
```

**Response**:

```json
{
  "tiers": [
    {
      "symbol": "BTCUSDT-PERP",
      "tier_index": 1,
      "max_position_qty": "50000",
      "max_leverage": "125",
      "initial_margin_rate": "0.008",
      "maintenance_margin_rate": "0.004",
      "config_version": 1,
      "from_config_table": true
    }
  ]
}
```

**Response Fields**:

- `tiers`: Array of risk tier rows (sorted by `tier_index` per symbol as returned by PM).
  - `symbol`: Market symbol for this row.
  - `tier_index`: Bracket index (1 = smallest notional band; larger index = larger allowed position **notional** under that bracket).
  - `max_position_qty`: Upper bound on position **notional in quote** for this tier (decimal string; e.g. USDT for USDT-linear), Binance USD-M `notionalCap`-style. The JSON field name is historical — this is **not** |base| contract amount.
  - `max_leverage`: Max leverage for this bracket (decimal string).
  - `initial_margin_rate` / `maintenance_margin_rate`: Decimal strings (IMR / MMR).
  - `config_version`: Config version from PM / DB when table-backed.
  - `from_config_table`: `true` if rows come from `perps_market_risk_tiers`; `false` if synthesized from market config only.

**Status Codes**:

- `200` - Risk tiers returned successfully (possibly empty `tiers` only when no active markets contribute rows)
- `404` - `symbol` was provided but no tier rows matched that symbol (unknown or inactive market relative to the loaded snapshot)
- `500` - Internal server error (e.g. PM unavailable)

---

### GET /perpetual/account/fee-schedule

Retrieve maker/taker fee rate configuration for all markets associated with the specified app session.

**Authentication**: Required

**Query Parameters**:

- `app_session_id` (string, required): App session identifier passed by frontend

**Response**:

```json
[
  {
    "app_session_id": "perp_session_123",
    "account_id": "550e8400-e29b-41d4-a716-446655440000",
    "market": "BTCUSDT",
    "maker_rate": "0.001",
    "taker_rate": "0.002"
  }
]
```

**Response Fields**:

- `app_session_id`: Application session identifier (from request parameter)
- `account_id`: Account ID
- `market`: Market Symbol
- `maker_rate`: Decimal precision for maker fee rate
- `taker_rate`: Decimal precision for taker fee rate

**Status Codes**:

- `200` - Account Fee Rate retrieved successfully
- `500` - Internal server error

---

### GET /perpetual/accounts

Retrieve all perpetuals accounts for the authenticated user.

**Authentication**: Required

**Response**:

```json
[
  {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "app_session_id": "perp_session_123",
    "owner": "0x1234567890abcdef1234567890abcdef12345678",
    "state": "active",
    "opened_at": "2023-12-01T08:00:00.000000Z",
    "total_account_balance": "10000.00000000",
    "total_unrealized_pnl": "150.50000000",
    "total_account_equity": "10150.50000000",
    "total_allocated_margin": "2000.00000000",
    "total_locked_balance": "2000.00000000",
    "total_maintenance_margin": "1000.00000000",
    "available_balance": "8150.50000000",
    "transferable_balances": {
      "USDT": "6400.00000000"
    },
    "position_modes": { "BTCYTEST.USD-PERP": "HEDGE" },
    "initial_leverages": { "BTCYTEST.USD-PERP": "10" },
    "balances": [
      {
        "asset_symbol": "USDT",
        "asset_name": "Tether USD",
        "total_balance": "10000.00000000",
        "allocated_balance": "2000.00000000",
        "locked_balance": "2000.00000000",
        "available_balance": "8000.00000000",
        "max_transfer_out": "6400.00000000",
        "total_balance_usd": "10000.00000000",
        "allocated_balance_usd": "2000.00000000",
        "locked_balance_usd": "2000.00000000",
        "available_balance_usd": "8000.00000000",
        "last_updated": "2023-12-07T10:30:00.000000Z"
      }
    ],
    "positions": [
      {
        "market": "BTCYTEST.USD-PERP",
        "direction": "long",
        "amount": "1.50000000",
        "entry_price": "35000.00000000",
        "mark_price": "35100.00000000",
        "notional_value": "52650.00000000",
        "leverage": "10.00000000",
        "unrealized_pnl": "150.00000000",
        "realized_pnl": "75.00000000",
        "total_pnl": "225.00000000",
        "allocated_margin": "5265.00000000",
        "maintenance_margin": "2632.50000000",
        "liquidation_price": "31500.00000000",
        "margin_asset": "USDT"
      }
    ]
  }
]
```

**Response Fields**:

- `id`: Unique perpetuals account identifier (UUID)
- `app_session_id`: Application session identifier
- `owner`: Owner's Ethereum wallet address
- `state`: Account state (`active`, `closed`)
- `opened_at`: Account creation timestamp
- `total_account_balance`: Sum of all collateral balances
- `total_unrealized_pnl`: Total unrealized PnL across all positions
- `total_account_equity`: Total account equity (balance + unrealized PnL)
- `total_allocated_margin`: Total margin allocated to positions (historical name; same numeric value as `total_locked_balance`)
- `total_locked_balance`: Sum of locked collateral across all assets (orders + position margin); same value as `total_allocated_margin`
- `total_maintenance_margin`: Total maintenance margin required
- `available_balance`: (Account-level.) Balance available for new positions or withdrawals. **Includes unrealized PnL**: formula is total balance − allocated/locked + unrealized PnL. **Not** the max you can transfer perp→spot; use `transferable_balances` or `balances[].max_transfer_out` for that cap.
- `transferable_balances`: (optional) Map of `asset_symbol` → decimal string: max amount the user may transfer **perpetuals → spot** for that collateral asset. Same semantics as WebSocket `perpetuals_account.account_update` and as `balances[].max_transfer_out` (closing-fee reserve; 80% cap when cross-margin positions exist). Omitted when there are no balance rows.
- `balances`: Array of collateral balances
  - `asset_symbol`: Collateral asset symbol (e.g., "USDT")
  - `asset_name`: Display name (when known)
  - `total_balance`: Total balance for this asset
  - `allocated_balance`: Margin allocated to positions (historical name; same numeric value as `locked_balance`)
  - `locked_balance`: Locked collateral for this asset (orders + position margin); same value as `allocated_balance`
  - `available_balance`: (Per-asset.) Available for this asset. **Includes unrealized PnL** (same formula: total − locked + unrealized PnL; in single-collateral the PnL is attributed to that asset).
  - `max_transfer_out`: Max user perp→spot transfer for this asset (may be lower than `available_balance` when open positions require a closing-fee reserve or cross-margin transfer cap applies).
  - `total_balance_usd`: Total balance valued in stablecoin (USD/USDT); `"0"` when no price
  - `allocated_balance_usd`: Allocated/locked margin valued in stablecoin; `"0"` when no price
  - `locked_balance_usd`: Same as `allocated_balance_usd` (locked collateral in USD terms)
  - `available_balance_usd`: Available balance valued in stablecoin; `"0"` when no price
  - `last_updated`: Last balance update timestamp
- `position_modes`: (optional) Map of market → `ONE_WAY` or `HEDGE`. Drives how `**POST /perpetual/order`** uses `direction` / `reduce_only` (see that endpoint). Only present when the account has per-market settings.
- `initial_leverages`: (optional) Map of market -> leverage string (e.g. `"10"`). Only present when the account has per-market settings.
- `positions`: Array of open positions. **HEDGE** may include **two** rows for the same `market` (one `long`, one `short`). **ONE_WAY** has at most **one** net row per `market`, with `direction` still `**long` or `short`** (never `both`; `both` is only used when **placing** orders in one-way mode).
  - `market`: Trading market (e.g., "BTCYTEST.USD-PERP")
  - `direction`: Position direction (`long` or `short`)
  - `amount`: Position size (always positive)
  - `entry_price`: Average entry price
  - `mark_price`: Current mark price
  - `notional_value`: Current notional value (amount x mark_price)
  - `leverage`: Leverage multiplier
  - `unrealized_pnl`: Unrealized profit/loss
  - `realized_pnl`: Realized profit/loss from this position
  - `total_pnl`: Total profit/loss (unrealized + realized)
  - `allocated_margin`: Margin allocated to this position
  - `maintenance_margin`: Maintenance margin requirement
  - `liquidation_price`: Liquidation price (calculated by backend for cross-margin positions)
  - `margin_asset`: Collateral asset for margin calculation

**Status Codes**:

- `200` - Accounts retrieved successfully
- `401` - Authentication failed
- `500` - Internal server error

---

### POST /perpetual/account

Create a new perpetuals account with a specified app_session_id.

**Authentication**: Required

> **Important: Asynchronous Operation.** Account creation is **asynchronous**. This endpoint returns `200` immediately, but the account does not exist yet. Clients must poll `GET /perpetual/account` until it returns a non-null result (typically takes ~3 seconds). There is currently no webhook or WebSocket notification available for account creation completion.

**Request Body**:

```json
{
  "app_session_id": "spot_loadtest_1500",
  "owner": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"
}
```

**Request Parameters**:

- `app_session_id` (string, required): Application session ID for the new perpetual account (typically same as spot account ID)
- `owner` (string, optional): Owner wallet address. Defaults to authenticated user if not provided.

**Response**:

```json
{
  "app_session_id": "spot_loadtest_1500",
  "owner": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  "message": "Perpetuals account creation initiated"
}
```

**Response Fields**:

- `app_session_id`: The app session ID of the created account
- `owner`: Owner wallet address
- `message`: Status message

**Status Codes**:

- `200` - Account creation initiated successfully
- `400` - Invalid request parameters (missing app_session_id, invalid owner address)
- `401` - Authentication failed
- `403` - Forbidden (owner mismatch - trying to create account for different wallet)
- `500` - Internal server error

**Notes**:

- The `app_session_id` should typically match the user's spot account ID to allow seamless transfers
- Account creation is asynchronous - the account will be available shortly after this request completes
- Attempting to create an account with an existing `app_session_id` will return an error

---

### GET /perpetual/account

Retrieve detailed information for a specific perpetuals account including all balances, positions, and account-level metrics.

**Authentication**: Required

**Query Parameters**:

- `app_session_id` (string, required): Perpetuals account app session ID

**Request**:

```http
GET /perpetual/account?app_session_id=perp_session_123
```

**Response**:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "app_session_id": "perp_session_123",
  "owner": "0x1234567890abcdef1234567890abcdef12345678",
  "state": "active",
  "opened_at": "2023-12-01T08:00:00.000000Z",
  "total_account_balance": "10000.00000000",
  "total_unrealized_pnl": "150.50000000",
  "total_account_equity": "10150.50000000",
  "total_allocated_margin": "2000.00000000",
  "total_locked_balance": "2000.00000000",
  "total_maintenance_margin": "1000.00000000",
  "available_balance": "8150.50000000",
  "transferable_balances": {
    "USDT": "6400.00000000"
  },
  "position_modes": { "BTCYTEST.USD-PERP": "HEDGE" },
  "initial_leverages": { "BTCYTEST.USD-PERP": "10" },
  "balances": [
    {
      "asset_symbol": "USDT",
      "asset_name": "Tether USD",
      "total_balance": "10000.00000000",
      "allocated_balance": "2000.00000000",
      "locked_balance": "2000.00000000",
      "available_balance": "8000.00000000",
      "max_transfer_out": "6400.00000000",
      "total_balance_usd": "10000.00000000",
      "allocated_balance_usd": "2000.00000000",
      "locked_balance_usd": "2000.00000000",
      "available_balance_usd": "8000.00000000",
      "last_updated": "2023-12-07T10:30:00.000000Z"
    }
  ],
  "positions": []
}
```

**Response Fields**: Same as `/perpetual/accounts` response (single account object), including optional `position_modes`, `initial_leverages`, and `transferable_balances`.
**Note on `available_balance` vs transfers**: The name appears in two places—(1) at the **account root**: aggregate balance available for new positions; (2) inside each `**balances[]`** element: available for that asset. **Both include unrealized PnL** (formula: total − allocated/locked + unrealized PnL). For **perp→spot transfer** limits, use `**transferable_balances`** (account root) or `**balances[].max_transfer_out**` per asset—these match WebSocket `perpetuals_account.account_update` and account for closing-fee reserve and cross-margin transfer caps.

**Status Codes**:

- `200` - Account retrieved successfully
- `400` - Missing app_session_id parameter
- `401` - Authentication failed
- `404` - Account not found
- `500` - Internal server error

---

### GET /perpetual/balance

Retrieve collateral balances for a specific perpetuals account.

**Authentication**: Required

**Query Parameters**:

- `app_session_id` (string, required): Perpetuals account app session ID

**Request**:

```http
GET /perpetual/balance?app_session_id=perp_session_123
```

**Response**:

```json
[
  {
    "asset_symbol": "USDT",
    "asset_name": "Tether USD",
    "total_balance": "10000.00000000",
    "allocated_balance": "2000.00000000",
    "locked_balance": "2000.00000000",
    "available_balance": "8000.00000000",
    "max_transfer_out": "6400.00000000",
    "total_balance_usd": "10000.00000000",
    "allocated_balance_usd": "2000.00000000",
    "locked_balance_usd": "2000.00000000",
    "available_balance_usd": "8000.00000000",
    "last_updated": "2023-12-07T10:30:00.000000Z"
  },
  {
    "asset_symbol": "USDC",
    "asset_name": "USD Coin",
    "total_balance": "5000.00000000",
    "allocated_balance": "1000.00000000",
    "locked_balance": "1000.00000000",
    "available_balance": "4000.00000000",
    "max_transfer_out": "4000.00000000",
    "total_balance_usd": "5000.00000000",
    "allocated_balance_usd": "1000.00000000",
    "locked_balance_usd": "1000.00000000",
    "available_balance_usd": "4000.00000000",
    "last_updated": "2023-12-07T10:30:00.000000Z"
  }
]
```

**Response Fields**:

- `asset_symbol`: Collateral asset symbol
- `asset_name`: Display name (when known)
- `total_balance`: Total balance for this asset
- `allocated_balance`: Margin allocated to positions (historical name; same numeric value as `locked_balance`)
- `locked_balance`: Locked collateral for this asset (orders + position margin); same value as `allocated_balance`
- `available_balance`: Balance available for new positions (includes unrealized PnL share for this asset)
- `max_transfer_out`: Max user perp→spot transfer for this asset (same as `transferable_balances[asset]` on `GET /perpetual/account`)
- `total_balance_usd`: Total balance valued in stablecoin (USD/USDT); `"0"` when no price
- `allocated_balance_usd`: Allocated/locked margin valued in stablecoin; `"0"` when no price
- `locked_balance_usd`: Same as `allocated_balance_usd` (locked collateral in USD terms)
- `available_balance_usd`: Available balance valued in stablecoin; `"0"` when no price
- `last_updated`: Last balance update timestamp

**Status Codes**:

- `200` - Balances retrieved successfully
- `400` - Missing app_session_id parameter
- `401` - Authentication failed
- `404` - Account not found
- `500` - Internal server error

---

### GET /perpetual/transfer-assets

Retrieve the **system-level** perpetual transfer asset configuration from `perps_assets`.

This endpoint returns which assets are configured for spot/perps transfer at the platform level.
Only assets that are both **active** and marked as **stablecoin** are returned.
It does **not** represent user-level transferability (e.g., user balance availability).

**Authentication**: Not required (public endpoint)

**Request**:

```http
GET /perpetual/transfer-assets
```

**Response**:

```json
[
  {
    "symbol": "USDT",
    "name": "Tether USD",
    "decimals": 6,
    "is_active": true
  }
]
```

**Response Fields**:

- `symbol`: Asset symbol from `perps_assets`
- `name`: Display name from `perps_assets`
- `decimals`: Decimal precision for min transfer amount logic (`10^(-decimals)`)
- `is_active`: Whether this asset is currently enabled for transfer

Only assets that are both active and stablecoin in `perps_assets` are included (stablecoin flag is not repeated in the JSON).

**Status Codes**:

- `200` - Transfer assets retrieved successfully
- `500` - Internal server error

---

### GET /perpetual/transaction/history

Retrieve unified perpetual transaction history for a wallet + app session, backed by `perps_transactions` + `perps_ledger_entries`.

This endpoint aggregates balance-changing events (funding payments, internal transfers, fees, realized PnL, liquidations, ADL, etc.) into a single paginated history from the **user's perspective**. Each row is one `(transaction_id, asset)` pair:

- Positive `amount` → balance increase
- Negative `amount` → balance decrease

**Authentication**: Required

**Query Parameters**:

- `app_session_id` (string, required): Perpetuals account app session ID
- `type` (string, optional): High-level transaction type filter. Supported values:
  - `""` (empty / omitted): User-facing types only (excludes internal order bookkeeping such as `ORDER_*`, `DEPOSIT`, `WITHDRAWAL`, etc.)
  - `"funding_fee"`: Funding payments (maps to `FUNDING_FEE_IN`, `FUNDING_FEE_OUT`)
  - `"transfer"`: Internal asset transfers (maps to `TRANSFER_IN`, `TRANSFER_OUT`)
  - `"fee"`: Trading fee-related events (maps to `FEE`, `FEE_WAIVER_DISTRESSED_CLOSE`, `POSITION_OPEN_FEE`, `POSITION_CLOSE_FEE`)
  - `"realized_pnl"`: Realized PnL events (maps to `REALIZED_PNL`)
  - `"liquidation"`: Liquidation events (maps to `LIQUIDATION`, `LIQUIDATION_COMPLETE`)
  - `"adl"`: ADL events (maps to `ADL_COUNTERPARTY_CLOSE`, `ADL_TAKEN_OVER_CLOSE`)
  - Compatibility note: ADL events are intentionally separated from the `liquidation` filter; clients that want all forced-deleveraging events should query both `type=liquidation` and `type=adl`.
- `market` (string, optional): Market filter (e.g. `BTC-USDT-PERP`)
- `asset` (string, optional): Asset symbol to filter by (e.g. margin asset symbol)
- `start_time` (integer, optional): Inclusive range start, **Unix timestamp in seconds (UTC)**. Passed through to Portfolio Manager Perp; see **Time range** below.
- `end_time` (integer, optional): Inclusive range end, **Unix timestamp in seconds (UTC)**. Passed through to Portfolio Manager Perp; see **Time range** below.
- `page` (integer, optional): Legacy offset page number (default: `1`, minimum: `1`). Omit or use `0` with cursor mode.
- `page_size` (integer, optional): Number of items per page (default: `50`, max: `100`)
- `cursor` (string, optional): Opaque cursor from previous `next_cursor` (Phase A). See [Perpetual list pagination (Phase A dual-mode)](#perpetual-list-pagination-phase-a-dual-mode).
- `use_cursor` (boolean, optional): Force cursor pagination.

**Time range** (server-side, Portfolio Manager Perp):

Trading API forwards `start_time` and `end_time` unchanged. When bounds are omitted (or sent as `0`), the backend applies a default window of the **last 30 days** ending at now (UTC):

| Client sends | Effective window |
|---|---|
| Neither `start_time` nor `end_time` | `[now − 30d, now]` |
| `start_time` only | `[start_time, now]` |
| `end_time` only | `[end_time − 30d, end_time]` |
| Both | `[start_time, end_time]` (if `start_time > end_time`, start is reset to `end_time − 30d`) |

The time window is applied on ledger `created_at`, which is written in the same instant as `perps_transactions.transaction_time` for normal audit events.

**Request** (explicit time range):

```http
GET /perpetual/transaction/history?app_session_id=perp_session_123&type=funding_fee&market=BTC-USDT-PERP&asset=USDT&start_time=1704067200&end_time=1704153600&page_size=20
```

**Request** (default last 30 days — omit `start_time` and `end_time`; cursor first page):

```http
GET /perpetual/transaction/history?app_session_id=perp_session_123&use_cursor=true&page_size=20
```

**Request** (legacy offset):

```http
GET /perpetual/transaction/history?app_session_id=perp_session_123&page=1&page_size=20
```

**Request** (cursor page 2):

```http
GET /perpetual/transaction/history?app_session_id=perp_session_123&use_cursor=true&cursor=1701943800000:5721092:USDT&page_size=20
```

**Response**:

```json
{
  "items": [
    {
      "transaction_id": 12345,
      "transaction_time": "2023-12-07T10:00:00.000000Z",
      "transaction_type": "FUNDING_FEE_IN",
      "market": "BTC-USDT-PERP",
      "asset": "USDT",
      "amount": "12.34567890"
    },
    {
      "transaction_id": 12344,
      "transaction_time": "2023-12-07T09:45:00.000000Z",
      "transaction_type": "TRANSFER_OUT",
      "market": "",
      "asset": "USDT",
      "amount": "-100.00000000"
    }
  ],
  "total": null,
  "has_next": false,
  "has_more": false,
  "next_cursor": "",
  "page": 0,
  "page_size": 20
}
```

**Pagination**:

- **Cursor mode** (recommended): `use_cursor=true` or omit `page`/`cursor` for first page. Follow `next_cursor` while `has_more` is `true`. Response `page` is `0`. `total` is always `null`.
- **Legacy offset**: `page` ≥ 1 with optional `page_size`. Use `has_next` (same as `has_more`) to detect more rows; increment `page` for the next window.
- Malformed `cursor` → `400` / `"error": "invalid_cursor"`.

**Response Fields**:

- `items`: Array of transaction history records
  - `transaction_id`: Internal transaction identifier
  - `transaction_time`: Transaction timestamp in RFC3339 format
  - `transaction_type`: Low-level transaction type (e.g., `FUNDING_FEE_IN`, `TRANSFER_OUT`, `FEE`, `REALIZED_PNL`, `LIQUIDATION`, `ADL_COUNTERPARTY_CLOSE`)
  - `market`: Related market symbol when applicable; empty string for non-market-specific events (e.g. some transfers)
  - `asset`: Asset symbol impacted by the transaction
  - `amount`: Signed amount string from the user's perspective (positive = balance increase, negative = balance decrease)
- `total`: Always `null` (kept for backward compatibility; not calculated)
- `has_next`: Whether more rows exist after this page (legacy; mirrors `has_more`)
- `has_more`: Phase A authoritative continuation flag
- `next_cursor`: Opaque next-page token (empty when no more pages)
- `page`: Request page (legacy) or `0` in cursor mode
- `page_size`: Number of items requested per page

**Status Codes**:

- `200` - Transaction history retrieved successfully
- `400` - Invalid query parameters (including malformed `cursor`) (e.g., missing `app_session_id`, invalid `type`, malformed `cursor`)
- `401` - Authentication failed
- `500` - Internal server error

---

### GET /perpetual/positions

Retrieve all open positions for a specific perpetuals account.

**Position mode**: Each row’s `direction` is `**long` or `short`** (stored leg). **HEDGE** may return two rows for the same `market` (long + short). **ONE_WAY** has at most one net row per `market`. Placing orders in one-way mode uses `direction: "both"` — that is **not** the same field semantics as this list.

**Authentication**: Required

**Query Parameters**:

- `app_session_id` (string, required): Perpetuals account app session ID

**Request**:

```http
GET /perpetual/positions?app_session_id=perp_session_123
```

**Response**:

```json
[
  {
    "market": "BTCYTEST.USD-PERP",
    "direction": "long",
    "amount": "1.50000000",
    "entry_price": "35000.00000000",
    "mark_price": "35100.00000000",
    "notional_value": "52650.00000000",
    "leverage": "10.00000000",
    "unrealized_pnl": "150.00000000",
    "realized_pnl": "75.00000000",
    "total_pnl": "225.00000000",
    "allocated_margin": "5265.00000000",
    "maintenance_margin": "2632.50000000",
    "liquidation_price": "31500.00000000",
    "margin_asset": "USDT",
    "margin_mode": "cross"
  }
]
```

**Response Fields**:

- `market`: Trading market (e.g., "BTCYTEST.USD-PERP")
- `direction`: Position direction (`long` or `short`). Use this value as `direction` for `**POST /perpetual/position/margin`** (do not use `both`).
- `amount`: Position size (always positive)
- `entry_price`: Average entry price
- `mark_price`: Current mark price used for PnL calculation
- `notional_value`: Current notional value (amount x mark_price)
- `leverage`: Leverage multiplier
- `unrealized_pnl`: Current unrealized profit/loss
- `realized_pnl`: Realized profit/loss from partial closes
- `total_pnl`: Total profit/loss (unrealized + realized)
- `allocated_margin`: Margin allocated to this position
- `maintenance_margin`: Maintenance margin requirement
- `liquidation_price`: Liquidation price (calculated by backend for cross-margin positions)
- `margin_asset`: Collateral asset (e.g., "USDT")
- `margin_mode`: `"cross"` or `"isolated"`

**Status Codes**:

- `200` - Positions retrieved successfully
- `400` - Missing app_session_id parameter
- `401` - Authentication failed
- `404` - Account not found
- `500` - Internal server error

---

### POST /perpetual/positions/close

Batch-submit **market IOC** close orders for every open position leg on the account that has **available** size greater than zero. Each leg uses the same path as `**POST /perpetual/order`** (Portfolio Manager `LockPerpAsset`, then `CommandOrderExecute` to Finex via Kafka).

If Kafka publish fails **after** PM lock succeeds, Trading API calls PM `UnlockPerpAsset` as compensation (best effort) so the temporary lock does not remain stuck. On that path, each failed leg in `failed` includes **`rollback_status`**: `ok` if unlock retries succeeded, `failed` if they did not (see `failed` fields below). Lock-only failures do **not** include `rollback_status`.

- If **`market`** is omitted or empty: all matching legs across markets are closed.
- If **`market`** is set: only positions for that market (case-insensitive match; symbol is normalized to uppercase for validation against exchange info).
- Legs are processed with bounded parallelism (worker pool) for lower latency.
- Safety cap: at most **50** closable legs are submitted per request; when more legs exist, the response includes `remaining_count` and `partial=true`.

**Position mode**: Uses per-market `**position_modes`** from the account (same as `**GET /perpetual/accounts**` / account payload). **HEDGE** legs use `direction` long/short with `reduce_only: true`. **ONE_WAY** uses `direction: "both"` and `reduce_only: true` (required for PM close semantics), with `buy`/`sell` derived from the stored net `**long`/`short`** row.

**Authentication**: Required (trade scope). Ownership of `app_session_id` is verified via `**GetPerpetualsAccount`**.

**Request Body**:

```json
{
  "app_session_id": "perp_session_123",
  "market": "BTCYTEST.USD-PERP",
  "idempotency_key": "close-20260423-001"
}
```

**Request Fields**:

- `app_session_id` (string, required): Perpetuals account app session ID
- `market` (string, optional): When present, only this market’s positions are closed; must exist in perpetuals exchange info when the market cache is configured
- `idempotency_key` (string, optional): Retry-safe key. When provided, each leg’s `order_uuid` is generated deterministically from `(idempotency_key, position identity)`, so repeated requests with the same key produce the same order UUIDs.

**Response** (200):

```json
{
  "message": "dispatched 2 close order(s)",
  "positions_submitted": 2,
  "failed_count": 0,
  "remaining_count": 0,
  "partial": false,
  "idempotency_key": "close-20260423-001",
  "orders": [
    {
      "market": "BTCYTEST.USD-PERP",
      "direction": "long",
      "margin_mode": "cross",
      "order_uuid": "550e8400-e29b-41d4-a716-446655440000"
    }
  ],
  "failed": []
}
```

**Response Fields**:

- `message` (string): Summary for the client
- `positions_submitted` (integer): Number of close orders successfully dispatched to Kafka
- `failed_count` (integer): Number of legs that failed validation, lock, or publish (see `failed`)
- `remaining_count` (integer): Number of additional closable legs not processed due to per-request cap
- `partial` (boolean): `true` when `remaining_count > 0` (caller should invoke again to continue)
- `idempotency_key` (string): Echo of request idempotency key (empty string when omitted)
- `orders` (array): Successful items (`market`, `direction`, `margin_mode` from the position row; `order_uuid` of the submitted order)
- `failed` (array): Per-leg failures with `market`, optional `direction` / `margin_mode`, `error`, optional `error_code`.
  - **Lock / validation / build errors**: `error` is a human-readable message (or structured lock rejection message); `error_code` is set for PM lock rejections (same idea as `POST /perpetual/order`). **`rollback_status` is omitted** (no post-lock publish, or lock failed before publish).
  - **Publish failure after a successful lock**: `error` is the literal string **`publish_failed`**; **`rollback_status`** is always set to `ok` or `failed` (`ok` if `UnlockPerpAsset` compensation succeeded, `failed` if all unlock retries failed—temporary lock may still be held).

Example `failed` entry when Kafka publish fails after lock:

```json
{
  "market": "BTCYTEST.USD-PERP",
  "direction": "long",
  "margin_mode": "cross",
  "error": "publish_failed",
  "rollback_status": "ok"
}
```

When there are no closable legs (all `available_amount` is zero), the handler returns `**200`** with `positions_submitted: 0` and an explanatory `message`.

**Status Codes**:

- `200` - Request completed (check `failed` for partial failures)
- `400` - Invalid JSON, missing `app_session_id`, or unknown `market` (when provided and exchange info is available)
- `401` - Authentication failed
- `403` - Permission denied for `(wallet_address, app_session_id)`
- `404` - Perpetual account not found
- `502` - PM request failed (non-permission/non-not-found backend error)
- `503` - Portfolio Manager router unavailable or PM temporarily unavailable

---

### POST /perpetual/position-mode

Set position mode for a market (`ONE_WAY` or `HEDGE`). Controls how `**POST /perpetual/order`** must set `direction`, `reduce_only`, and `position_mode` (see that endpoint).

- **ONE_WAY**: One net position per market. Orders must use `**direction: "both"`**; use `**buy`/`sell**` and `**reduce_only**` for open vs close (details under `POST /perpetual/order`).
- **HEDGE**: Long and short legs may both exist. Orders use `**direction: "long"` or `"short"`** (never `both`), with `**side**` / `**reduce_only**` for close semantics.

**Margin adjustment**: `POST /perpetual/position/margin` always uses `**direction: "long"` or `"short"`**, matching a row from `**GET /perpetual/positions**` (net direction in one-way mode; **not** `both`).

**Authentication**: Required

**Request Body**:

```json
{
  "app_session_id": "perp_session_123",
  "market": "BTCYTEST.USD-PERP",
  "position_mode": "ONE_WAY"
}
```

**Request Fields**:

- `app_session_id` (string, required): Perpetuals account app session ID
- `market` (string, required): Market symbol (e.g., "BTCYTEST.USD-PERP")
- `position_mode` (string, required): `"ONE_WAY"` or `"HEDGE"`

**Response**:

```json
{
  "success": true,
  "message": "Position mode set successfully"
}
```

**Status Codes**:

- `200` - Position mode set successfully
- `400` - Invalid request (missing fields, invalid position_mode, account not found)
- `401` - Authentication failed
- `500` - Internal server error

---

### POST /perpetual/leverage

**Handler**: `PostSetLeverageHandler` (Trading API). Sets **initial leverage** per account and market. **Order creation does not accept leverage**; Portfolio Manager resolves it from stored settings (`perps_account_market_settings.initial_leverage` / FSM) when matching orders. Use this endpoint (or the default, typically `10`) before trading if you need a specific multiplier.

**Position mode**: **ONE_WAY** and **HEDGE** use the **same** request body (`app_session_id`, `market`, `leverage` only — no `direction`).

**How to read current leverage**: `GET /perpetual/accounts` returns `initial_leverages` (market → leverage string). Positions and orders also expose an effective `leverage` field.

**Restrictions & behavior**:

- **Open orders**: Changing leverage is **rejected** while **any open order** exists for that **market** (`error` e.g. `order_exists`). Cancel or fill those orders first.
- **Open positions**: Changing leverage **is allowed**. The cluster applies the new leverage atomically: position `allocated_margin` is recalculated from notional ÷ leverage, and locked balance is adjusted. The call may still **fail** if the account does not have enough free collateral for a higher required margin, or if the new margin would fall **below maintenance** for a position (FSM error surfaced as `set_leverage_fsm_failed` or similar message from PM).
- **Validation**: Trading API requires `leverage >= 1`. PM enforces `leverage <= max_allowed_leverage` for that market (`leverage_exceeds_max`).

**Authentication**: Required (JWT or API Key). The wallet address must be present in the auth context; ownership of `app_session_id` is verified.

**Request Body**:

```json
{
  "app_session_id": "perp_session_123",
  "market": "BTCYTEST.USD-PERP",
  "leverage": "20"
}
```

**Request Fields**:

- `app_session_id` (string, required): Perpetuals account app session ID
- `market` (string, required): Market symbol (e.g., `BTCYTEST.USD-PERP`). Trimmed and uppercased by the server before the PM call
- `leverage` (number or string, required): Leverage multiplier, `>= 1` at the API layer, and `<= max_allowed_leverage` for the market on the PM side (e.g., `20` or `"20"`)

**Response** (200):

```json
{
  "success": true,
  "message": "leverage updated successfully",
  "market": "BTCYTEST.USD-PERP",
  "leverage": "20"
}
```

**Response Fields**:

- `success` (boolean): Whether the operation succeeded
- `message` (string): Human-readable message from PM
- `market` (string): Market symbol for which leverage was set
- `leverage` (string): Confirmed leverage (decimal string; same precision style as `POST /perpetual/position/margin` leverage fields)

**Error responses** (non-exhaustive; `success` is `false` when HTTP status is not 200):


| HTTP  | `error` (when present)                                                                                                                                     | Meaning                                                                                                                                                                                                                                                           |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `400` | `invalid_request_format`                                                                                                                                   | Malformed JSON                                                                                                                                                                                                                                                    |
| `400` | `missing_app_session_id`                                                                                                                                   | `app_session_id` empty                                                                                                                                                                                                                                            |
| `400` | `missing_market`                                                                                                                                           | `market` empty                                                                                                                                                                                                                                                    |
| `400` | `invalid_leverage_value`                                                                                                                                   | Leverage `< 1`                                                                                                                                                                                                                                                    |
| `400` | `no_permitted`                                                                                                                                             | Account not found or not accessible for this wallet                                                                                                                                                                                                               |
| `400` | *(from PM)* `leverage_exceeds_max`, `order_exists`, `insufficient_balance`, `leverage_exceeds_safe_margin`, `account_not_active`, `set_leverage_failed`, … | See `message` for detail; PM returns gRPC `error_code` as REST `error`. Some PM errors include `**placeholders`** (display-formatted decimal strings; see below). **Full list**: **Common Error Codes** → **Trading API Service** → `SetInitialLeverageResponse`. |
| `401` | `missing_wallet_address`                                                                                                                                   | Authenticated context has no wallet                                                                                                                                                                                                                               |
| `503` | `service_unavailable`                                                                                                                                      | Could not reach the correct Portfolio Manager node for this user                                                                                                                                                                                                  |
| `500` | `internal_error` / gRPC transport failures                                                                                                                 | See server logs; client may get a generic body for gRPC errors                                                                                                                                                                                                    |


**Error body** (400 from PM business rules; `placeholders` only when PM provides them):

```json
{
  "success": false,
  "error": "insufficient_balance",
  "message": "insufficient balance: insufficient available balance to apply new leverage (required extra margin: ..., available: ...)",
  "placeholders": {
    "available": "100.5",
    "required": "120"
  }
}
```

For `leverage_exceeds_safe_margin`, `placeholders` use `allocated` (allocated margin under the new leverage) and `required` (minimum margin including maintenance buffer).

**Status Codes**:

- `200` - Leverage set successfully
- `400` - Validation or business rule failure (see table)
- `401` - Not authenticated or missing wallet in context
- `503` - PM routing / connectivity failure
- `500` - Unexpected server or gRPC error

---

### POST /perpetual/margin-mode

Set the **margin mode** for a specific market on a perpetuals account. Margin mode controls how collateral is allocated for positions:

- `**cross`** (default): All available balance in the account backs positions. P&L from one position can offset losses in another.
- `**isolated**`: A fixed margin amount is assigned to each position. Losses are limited to that margin, but a position can be liquidated without affecting the rest of the account.

**Authentication**: Required (JWT or API Key, `trade_futures` scope). The wallet address must be present in the auth context; ownership of `app_session_id` is verified.

**Restrictions**: Changing margin mode may be rejected by Portfolio Manager while open positions or orders exist for the market.

**Request Body**:

```json
{
  "app_session_id": "perp_session_123",
  "market": "BTCYTEST.USD-PERP",
  "margin_mode": "isolated"
}
```

**Request Fields**:

- `app_session_id` (string, required): Perpetuals account app session ID
- `market` (string, required): Market symbol (e.g., `BTCYTEST.USD-PERP`)
- `margin_mode` (string, required): `"cross"` or `"isolated"` (case-insensitive)

**Response** (200):

```json
{
  "success": true,
  "market": "BTCYTEST.USD-PERP",
  "margin_mode": "isolated"
}
```

**Response Fields**:

- `success` (boolean): Whether the operation succeeded
- `market` (string): Market symbol for which the mode was set
- `margin_mode` (string): Confirmed margin mode (lowercased)

**Status Codes**:

- `200` - Margin mode set successfully
- `400` - Validation failure or account not found / access denied
- `401` - Authentication failed or missing wallet address
- `503` - Could not reach Portfolio Manager for this user
- `500` - Unexpected server or gRPC error

---

### POST /perpetual/position/margin

Add or remove **isolated/cross** margin on an **existing** position (distinct from `**POST /perpetual/leverage`**, which sets per-market **initial leverage**).

**Authentication**: Required

**Position mode**: `direction` is **always `long` or `short`**, and must match the position row from `**GET /perpetual/positions**` (or `positions` inside `**GET /perpetual/accounts**`). In **ONE_WAY**, use the net position’s `long`/`short`; **do not** send `both` (`both` is only for **placing** orders in one-way mode). In **HEDGE**, pick the **long** or **short** leg you are adjusting.

**Request Body**:

```json
{
  "app_session_id": "perp_session_123",
  "market": "BTCYTEST.USD-PERP",
  "direction": "long",
  "action": "add",
  "amount": "100",
  "asset": "USDT"
}
```

**Request Fields**:

- `app_session_id` (string, required)
- `market` (string, required)
- `direction` (string, required): `long` or `short` — target position row
- `action` (string, required): `add` or `reduce`
- `amount` (string, required): Positive decimal amount
- `asset` (string, required): Collateral asset symbol (e.g. `USDT`)

**Response** (200): On success, `success: true` with `position_id`, `market`, `direction`, `old_margin` / `new_margin`, `leverage_before` / `leverage_after`, liquidation prices, etc. (same shape as the Trading API JSON body).

**Status Codes**:

- `200` - Success (`success: true`) or PM rejection (`success: false` with `message`)
- `400` / `401` / `503` - Same classes of validation, auth, and routing errors as other perpetual endpoints

---

### GET /perpetual/position-history

Retrieve closed position history with pagination. Only returns positions that have been fully closed. Default ordering matches `**sort_by=closed_at`** and `**sort_dir=desc**`  with a stable secondary sort on internal row id.

**Authentication**: Required

**Query Parameters**:

- `app_session_id` (string, required): Perpetuals account app session ID
- `market` (string, optional): Exact market symbol filter; the API normalizes this value to **uppercase** before lookup (e.g. `btcytest.usd-perp` matches `BTCYTEST.USD-PERP`).
- `opened_from` (string, optional): Inclusive lower bound on `**opened_at`**. Accepts **RFC3339** or **RFC3339Nano** timestamps (same for other time filters below).
- `opened_to` (string, optional): Inclusive upper bound on `**opened_at`**.
- `closed_from` (string, optional): Inclusive lower bound on `**closed_at**`.
- `closed_to` (string, optional): Inclusive upper bound on `**closed_at**`.
- `sort_by` (string, optional): `opened_at` or `closed_at` (default: `closed_at`). Other values → `**400**`.
- `sort_dir` (string, optional): `asc` or `desc` (default: `desc`). Other values → `**400**`.
- `page` (integer, optional): Legacy offset page. If omitted with cursor mode, first cursor page is returned (`page` `0` in response).
- `page_size` (integer, optional): Number of positions per page. If **omitted**, defaults to `**50`**. If **present**, must be an integer from `**1`** to `**100**` (inclusive); otherwise `**400**`.
- `cursor` (string, optional): Phase A opaque cursor from `next_cursor`.
- `use_cursor` (boolean, optional): Force cursor pagination.
- `compute_total` (boolean, optional): Cursor path only — optionally compute `total` (expensive).

**Request**:

```http
GET /perpetual/position-history?app_session_id=perp_session_123&market=BTCYTEST.USD-PERP&opened_from=2024-01-01T00:00:00Z&closed_to=2024-01-31T23:59:59Z&sort_by=closed_at&sort_dir=desc&page=1&page_size=20
```

**Response**:

```json
{
  "positions": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "perps_account_id": "550e8400-e29b-41d4-a716-446655440001",
      "market": "BTCYTEST.USD-PERP",
      "direction": "long",
      "amount": "1.50000000",
      "entry_price": "35000.00000000",
      "leverage": "10.00000000",
      "allocated_margin": "5250.00000000",
      "realized_pnl": "225.50000000",
      "close_reason": "normal",
      "exit_price": "35150.00000000",
      "closed_quantity": "1.50000000",
      "max_held": "2.00000000",
      "initial_margin": "5000.00000000",
      "total_trading_fee": "12.50000000",
      "net_funding_fee": "-1.25000000",
      "liquidation_price": "0",
      "margin_mode": "cross",
      "pnl_ratio": "4.51000000",
      "opened_at": "2023-12-07T10:00:00.000000Z",
      "updated_at": "2023-12-07T11:30:00.000000Z",
      "closed_at": "2023-12-07T11:30:00.000000Z"
    }
  ],
  "total": 45,
  "page": 1,
  "page_size": 20,
  "next_cursor": "1701943800000:550e8400-e29b-41d4-a716-446655440000",
  "has_more": true
}
```

**Pagination**: Supports Phase A dual-mode pagination (see [Perpetual list pagination (Phase A dual-mode)](#perpetual-list-pagination-phase-a-dual-mode)). Malformed `cursor` → `400` / `invalid_cursor`.

**Response Fields**:

- `positions`: Array of closed position objects
  - `id`: Position ID (UUID)
  - `perps_account_id`: Perpetuals account ID (UUID)
  - `market`: Trading market
  - `direction`: Position direction (`long` or `short`)
  - `amount`: Position size (amount)
  - `entry_price`: Average entry price
  - `leverage`: Leverage multiplier used
  - `allocated_margin`: Margin that was allocated
  - `realized_pnl`: Final realized profit/loss
  - `close_reason`: Close reason (`normal`, `liquidated`, `adl`)
  - `exit_price`: Weighted-average close price over all close fills
  - `closed_quantity`: Total quantity closed for the position lifecycle
  - `max_held`: Peak absolute held quantity during the position lifecycle
  - `initial_margin`: Margin at first open (used for ROI)
  - `total_trading_fee`: Sum of linked trade fees for this position lifecycle
  - `net_funding_fee`: Signed cumulative funding for this position lifecycle (positive = received by user, negative = paid by user)
  - `liquidation_price`: Liquidation trigger snapshot for liquidated positions (else zero/empty)
  - `margin_mode`: Position margin mode (`cross` or `isolated`)
  - `pnl_ratio`: Realized ROE% = `realized_pnl / initial_margin * 100`
  - `opened_at`: Position open timestamp
  - `updated_at`: Last update timestamp
  - `closed_at`: Position close timestamp

Note: for legacy closed positions created before these fields were persisted, `max_held` and `initial_margin` can be `0`, which means historical values are unavailable rather than actual zero exposure.
Note: for positions that were already open when max-held/initial-margin persistence was introduced, `initial_margin` reflects migration-time/live-update value rather than the original open-time margin.

- `total`: Total number of closed positions matching criteria
- `page`: Current page number
- `page_size`: Number of positions per page

**Status Codes**:

- `200` - Position history retrieved successfully
- `400` - Invalid query parameters (including malformed `cursor`) (e.g. unparsable time strings, `opened_from` after `opened_to`, `closed_from` after `closed_to`, invalid `sort_by` / `sort_dir`, or `page_size` out of **1–100** when supplied)
- `401` - Authentication failed or missing wallet in context
- `403` - Permission denied (authenticated wallet is not the owner of `app_session_id`)
- `404` - Account not found
- `500` - Internal server error

---

### GET /perpetual/position-history/{id}

Retrieve **fill-level** history for a **single closed** position (one UUID from `GET /perpetual/position-history`). Results are **cursor-paginated** over the underlying `perps_trades` rows for that position (chronological order). Summary fields for the position itself are available on the list endpoint item with the same `id`.

**Authentication**: Required (JWT or API Key, `read:futures` scope). Owner is the authenticated wallet; must match the perpetuals account for `app_session_id`.

**Path Parameters**:

- `id` (string, required): Closed position UUID

**Query Parameters**:

- `app_session_id` (string, required): Perpetuals account app session ID
- `page_size` (integer, optional): Number of underlying trades loaded per request. If **omitted**, defaults to **200**. If supplied, must be a **positive** integer; values above **500** are **clamped to 500** by the service.
- `cursor` (string, optional): Opaque pagination cursor from the previous response’s `next_cursor`. Omit on the first page. Must match `**RFC3339Nano|uint64`** (pipe-separated) when present; malformed values typically surface as `**500**` from the backend rather than `**400**`.

**Pagination** (`cursor` / `next_cursor`):

- The server advances in **trade execution order** (`executed_at` ascending, then stable `id` tie-break).
- `next_cursor` is a tuple string: `**<RFC3339Nano>|<uint64 trade row id>`** (UTC in the timestamp part). Pass it back as `cursor` to fetch the next window.
- `has_more` is `true` when more trades exist after this page; when `false`, `next_cursor` is typically empty.

**Request** (first page):

```http
GET /perpetual/position-history/550e8400-e29b-41d4-a716-446655440000?app_session_id=perp_session_123&page_size=50
```

**Request** (next page):

```http
GET /perpetual/position-history/550e8400-e29b-41d4-a716-446655440000?app_session_id=perp_session_123&page_size=50&cursor=2023-12-07T10%3A05%3A10.000000000Z%7C12345
```

**Response**:

```json
{
  "fills": [
    {
      "id": "6c611f2f-8bf9-4d53-8dc0-bb8ce31ec6ff",
      "direction": "long",
      "amount": "1.00000000",
      "price": "34900.00000000",
      "pnl": "0",
      "fee": "4.20000000",
      "fee_currency": "USD",
      "exec_type": "trade",
      "kind": "open",
      "executed_at": "2023-12-07T10:05:10.000000Z"
    },
    {
      "id": "f9733de9-ec8d-4274-96f8-ec57d6fb986f",
      "direction": "long",
      "amount": "1.00000000",
      "price": "35200.00000000",
      "pnl": "300.00000000",
      "fee": "5.10000000",
      "fee_currency": "USD",
      "exec_type": "liquidation",
      "kind": "close",
      "executed_at": "2023-12-07T11:29:59.000000Z"
    }
  ],
  "next_cursor": "2023-12-07T11:29:59.000000000Z|67890",
  "has_more": false
}
```

**Response Fields**:

- `fills`: Array of **per-fill** rows derived from trades in the current window (open and close perspectives are combined in one list; each row’s `kind` is `open` or `close`).
  - `id`: Trade UUID (`perps_trades.uuid`)
  - `direction`: Position leg direction (`long` or `short`)
  - `amount`: Fill amount
  - `price`: Fill price
  - `pnl`: Fill realized PnL (`0` for `open` fills)
  - `fee`: Fee paid for that fill from this account perspective
  - `fee_currency`: Fee currency
  - `exec_type`: Execution source (`trade`, `liquidation`, `liquidation_takeover`, `adl`)
  - `kind`: Fill role (`open` or `close`)
  - `executed_at`: Fill execution timestamp (RFC3339 / RFC3339Nano string)
- `next_cursor`: Cursor for the **next** request (empty when there is no next page).
- `has_more`: Whether more trades exist after this page.

**Status Codes**:

- `200` - Position history detail retrieved successfully
- `400` - Missing `app_session_id`, missing path `id`, invalid `page_size` (non-positive when supplied), or invalid position UUID
- `401` - Authentication failed or missing wallet in context
- `403` - Permission denied (wallet not owner of `app_session_id`, or position belongs to another account)
- `404` - Position not found (or no history payload)
- `409` - Position is not closed (`failed_precondition` — open positions have no fill history on this route)
- `500` - Internal server error (including malformed `cursor` in some deployments)

---

### POST /perpetual/order

Create a new perpetuals order.

> **Known Issue**: Order placement intermittently fails (~80% of attempts) with `NOT_LEADER` error due to Raft cluster instability. Retry with delay is recommended.

**Authentication**: Required

If Kafka publish fails **after** PM lock succeeds, Trading API calls PM `UnlockPerpAsset` as compensation (best effort) so the temporary lock does not remain stuck. The HTTP **`400`** response body then uses **`error`: `publish_failed`**, a **`message`** with the publish error text, and **`rollback_status`**: `ok` or `failed` depending on whether unlock retries succeeded (see **Error response** below). This shape is **not** used when the lock step fails before publish.

**Request Body** (**HEDGE** example — `direction` is `long` or `short`):

```json
{
  "app_session_id": "perp_session_123",
  "market": "BTCYTEST.USD-PERP",
  "side": "buy",
  "direction": "long",
  "position_mode": "HEDGE",
  "type": "limit",
  "amount": "0.5",
  "price": "35000",
  "leverage": "10",
  "time_in_force": "gtc",
  "reduce_only": false
}
```

**Request Body** (**ONE_WAY** example — `direction` must be `both`; use `reduce_only` to close):

```json
{
  "app_session_id": "perp_session_123",
  "market": "BTCYTEST.USD-PERP",
  "side": "buy",
  "direction": "both",
  "position_mode": "ONE_WAY",
  "type": "limit",
  "amount": "0.5",
  "price": "35000",
  "leverage": "10",
  "time_in_force": "gtc",
  "reduce_only": false
}
```

**Position mode: HEDGE vs ONE_WAY**

- Per-market mode is stored on the account via `**POST /perpetual/position-mode`**. Align your order fields with the active mode so margin lock and matching semantics stay consistent.
- **HEDGE**
  - Set `direction` to `**long`** or `**short**` (the long or short leg). **Do not** use `both`.
  - Closing intent: besides `reduce_only`, you can use `**side` + `direction`** (e.g. `direction=long` and `side=sell` reduces a long).
- **ONE_WAY**
  - `direction` **must** be `**both`**. Portfolio Manager validates this in one-way mode; sending `long`/`short` fails (e.g. `ONE_WAY mode requires direction=both`).
  - There is a single net position per market; with `direction=both`, whether the order **closes** is mainly indicated by `**reduce_only: true`**, together with `side` vs net exposure (e.g. close long with `sell`, close short with `buy`).
  - Pass `**"position_mode": "ONE_WAY"**` explicitly on the request body. If omitted, normalization defaults `**HEDGE**` on the downstream command, which can disagree with the account’s actual mode.

**Request Parameters**:

- `app_session_id` (string, required): Perpetuals account app session ID
- `market` (string, required): Trading market (e.g., "BTCYTEST.USD-PERP")
- `side` (string, required): Order side - `buy` or `sell`
- `direction` (string, required): `**long`**, `**short**`, or `**both**`. Use `**long`/`short**` in **HEDGE** mode; use `**both`** in **ONE_WAY** mode only.
- `position_mode` (string, optional): `ONE_WAY` or `HEDGE`. If omitted, defaults to `**HEDGE`** in the API normalization layer; for **ONE_WAY** accounts, pass `**ONE_WAY`** explicitly.
- `type` (string, required): Order type - `limit`, `market`, `post_only`, or advanced/trigger types (see OpenAPI `PostCreatePerpetualOrderRequest`)
- `amount` (string, required): Order amount in base asset (decimal format)
- `price` (string, optional): Limit price (required for `limit` and `post_only` orders)
- `leverage` (string, required in schema): Sent with the request; effective margin rules still follow account/market settings from `POST /perpetual/leverage` and PM lock logic.
- `time_in_force` (string, optional): Time in force - `gtc` (Good-Till-Cancelled), `ioc` (Immediate-Or-Cancel), `fok` (Fill-Or-Kill). Defaults to `gtc` for `limit` and `post_only` orders.
- `reduce_only` (boolean, optional): If true, order can only reduce existing position size (default: false). In **ONE_WAY** with `direction=both`, this is the primary way to mark a closing order. Compatible with `post_only`.
- `client_order_id` (string, optional): Client-supplied order id
- `trigger_price` / `trigger_type` (optional): For trigger / stop / take-profit style orders (see OpenAPI). **Not allowed for `post_only`.**

> `**post_only` orders (perp)**: Maker-only limit order. Requires non-zero `price`; `trigger_price` / `trigger_type` are **not allowed**. If the order would immediately match on entry, Finex rejects it and pushes WebSocket `order.rejected` with `reason=post_only_would_match` (no fills, no resting, margin released).
>
> **Margin lock optimization**: Portfolio Manager locks the open-fee reserve at the **maker** fee rate (instead of taker) for `post_only` orders, since they are guaranteed to be makers. This can reduce required margin enough that a `post_only` order succeeds where an equivalent `limit` order at the same price would be rejected with `insufficient_margin`.

**Leverage**: Also configured via `POST /perpetual/leverage` before the first order. Inspect `GET /perpetual/accounts` (`initial_leverages`, `position_modes`) for current settings.

> **Margin mode**: Margin mode is determined by per-market settings (set via `POST /perpetual/margin-mode`). The default is `"cross"` if not configured. Passing `margin_mode` in the order request is no longer accepted.

**Response**:

```json
{
  "order_uuid": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Response Fields**:

- `order_uuid`: UUID of the created order

**Status Codes**:

- `200` - Order created successfully
- `400` - Invalid request parameters, validation failed, Portfolio Manager rejected the perp lock **before** publish, or **Kafka publish failed after a successful lock** (see publish-failure body below)
- `401` - Authentication failed
- `500` - Internal server error

**Error response** (typical for `400` — lock or validation, same family as other trading routes):

```json
{
  "error": "insufficient_margin",
  "message": "insufficient margin. Available:9.62, Required:9.6256",
  "placeholders": {
    "available": "9.62",
    "required": "9.6256"
  }
}
```

- `error`: Machine-readable code (`snake_case`). When the failure comes from the **perpetuals** Portfolio Manager **lock** step (`LockPerpAsset`), this equals the gRPC `error_code` (see **Common Error Codes** → **Trading API Service** → `LockPerpAssetResponse`). If PM returns `success=false` with an empty `error_code`, the gateway uses `lock_funds_rejected`. Other validation or gateway errors may still use `validation_failed`. **If PM lock succeeded but publishing the order command to Kafka failed**, the gateway returns **`error`: `publish_failed`** (not `validation_failed`) together with **`rollback_status`** (next subsection).
- `message`: Human-readable detail from PM or the gateway.
- `placeholders` (object, optional): Structured amounts from PM (`LockPerpAssetResponse.error_placeholders`). Keys match the table under **Common Error Codes** (e.g. `insufficient_margin` → `available`, `required`). Values use the **same display formatting as `message`** (asset-scaled decimal strings for UI/i18n); they are **not** guaranteed to be canonical full-precision internal decimals. Omitted when not applicable or empty.

**Error response** (`400` — **publish failed after successful PM lock**):

```json
{
  "error": "publish_failed",
  "message": "<underlying Kafka / publisher error>",
  "rollback_status": "ok"
}
```

- `rollback_status`: `ok` if `UnlockPerpAsset` compensation completed; `failed` if all configured unlock retries failed (temporary lock may still be held—treat as operational incident; retry or escalate).
- **`rollback_status` is not** included when the failure occurs **before** publish (e.g. lock rejection, request validation).

---

### DELETE /perpetual/order

Cancel an existing perpetuals order.

> **Known Issue**: The API returns `200` with success message, but the order may remain in `wait` state. Cancel functionality is not reliably working.

**Authentication**: Required

**Request Body**:

```json
{
  "app_session_id": "perp_session_123",
  "market": "BTCYTEST.USD-PERP",
  "order_uuid": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Request Parameters**:

- `app_session_id` (string, required): Perpetuals account app session ID
- `market` (string, required): Trading market
- `order_uuid` (string, required): UUID of the order to cancel

**Response**:

```json
{
  "message": "Order cancellation request sent successfully"
}
```

**Status Codes**:

- `200` - Cancellation request sent successfully
- `400` - Invalid request or missing parameters
- `401` - Authentication failed
- `500` - Internal server error

---

### DELETE /perpetual/orders

Cancel **all open** perpetuals orders for an account, optionally limited to one market.

**Authentication**: Required

**Request Body**:

```json
{
  "app_session_id": "perp_session_123",
  "market": "BTCYTEST.USD-PERP"
}
```

**Request Fields**:

- `app_session_id` (string, required): Perpetuals account app session ID
- `market` (string, optional): When empty or omitted, open orders in **all** markets are targeted; when set, only orders for that exact market string (as stored on open orders)

**Behavior**: The server lists open orders via the portfolio manager (`ListPerpetualsOpenOrders` with a large page size), then dispatches a `**CommandOrderCancelFinex`** to each order’s market Kafka topic. Trigger orders in `trigger_wait` / `trigger_triggered` also receive `**CommandPerpTriggerOrderCancel**` on the trigger command topic (same pattern as `**DELETE /perpetual/order**`).

**Response** (200):

```json
{
  "message": "Successfully dispatched 3 order cancellations",
  "total_orders": 3,
  "canceled_count": 3,
  "failed_count": 0
}
```

When there are no open orders:

```json
{
  "message": "No open orders to cancel",
  "canceled_count": 0
}
```

**Response Fields**:

- `message` (string): Summary
- `total_orders` (integer): Open orders found in the listing pass (omitted when count is zero in the “no open orders” response)
- `canceled_count` (integer): Cancel commands successfully published to Kafka for Finex
- `failed_count` (integer): Publish failures after `canceled_count` was incremented (best-effort loop; see server logs for details)

**Status Codes**:

- `200` - Listing and dispatch completed (check `failed_count` for partial publish failures)
- `400` - Invalid JSON or missing `app_session_id`
- `401` - Authentication failed
- `500` - Failed to list open orders

---

### GET /perpetual/orders

Retrieve perpetuals order history with pagination. Each order’s `direction` is stored as returned (e.g. **HEDGE** `long`/`short`; **ONE_WAY** placements often `both`) — same idea as `**GET /perpetual/open_orders`**.

**Authentication**: Required

**Query Parameters**:

- `app_session_id` (string, required): Perpetuals account app session ID
- `market` (string, optional): Exact filter by trading market
- `market_like` (string, optional): Fuzzy filter by market (contains, case-insensitive)
- `page` (integer, optional): Legacy offset page (default: 1)
- `page_size` (integer, optional): Number of orders per page (default: 50, max: 100)
- `cursor`, `use_cursor`, `compute_total` (optional): Phase A pagination — see [Perpetual list pagination (Phase A dual-mode)](#perpetual-list-pagination-phase-a-dual-mode)

**Request**:

```http
GET /perpetual/orders?app_session_id=perp_session_123&market=BTCYTEST.USD-PERP&page=1&page_size=20
```

**Response**:

```json
{
  "orders": [
    {
      "id": "1234567",
    "order_id": "550e8400-e29b-41d4-a716-446655440000",
    "app_session_id": "perp_session_123",
    "market": "BTCYTEST.USD-PERP",
    "price": "35000.00000000",
    "amount": "0.50000000",
    "origin_amount": "0.50000000",
    "fill_amount": "0.20000000",
    "notional": "17500.00000000",
    "side": "buy",
    "type": "limit",
    "state": "wait",
    "event": "",
    "reason": "",
    "leverage": "10.00000000",
    "created_at": "2023-12-07T10:30:00.000000Z",
    "updated_at": "2023-12-07T10:30:05.000000Z",
    "completed_at": ""
    }
  ],
  "total": 123,
  "page": 1,
  "page_size": 20,
  "next_cursor": "1234520",
  "has_more": true
}
```

**Response Fields**:

- `orders`: Array of order objects
- `total`: Total number of orders matching the filter (legacy offset path; optional on cursor path unless `compute_total=true`)
- `page`: Current page number (`0` in cursor mode)
- `page_size`: Number of items per page
- `next_cursor`, `has_more`: Phase A cursor pagination fields
- Order object fields:
  - `id`: Internal order record ID
  - `order_id`: Order UUID
  - `app_session_id`: Perpetuals account app session ID
  - `market`: Trading market
  - `price`: Order price
  - `amount`: Current order amount (remaining to fill)
  - `origin_amount`: Original order amount
  - `fill_amount`: Amount that has been filled
  - `notional`: Notional value (origin_amount x price)
  - `side`: Order side (`buy` or `sell`)
  - `type`: Order type (`limit`, `market`)
  - `state`: Order state (`pending`, `wait`, `done`, `canceled`)
  - `event`: Last order event
  - `reason`: Reason for last state change
  - `leverage`: Leverage multiplier
  - `created_at`: Order creation timestamp
  - `updated_at`: Last update timestamp
  - `completed_at`: Completion timestamp (empty for open orders)

**Status Codes**:

- `200` - Orders retrieved successfully
- `400` - Invalid query parameters (including malformed `cursor`)
- `401` - Authentication failed
- `500` - Internal server error

---

### GET /perpetual/open_orders

Retrieve open (unfilled) perpetuals orders. Returns only orders that are currently active in the order book. `**direction`** on each row follows stored order semantics (**HEDGE**: `long`/`short`; **ONE_WAY** placements often `**both`**), not position-list semantics.

**Authentication**: Required

**Query Parameters**:

- `app_session_id` (string, required): Perpetuals account app session ID
- `market` (string, optional): Exact filter by trading market
- `market_like` (string, optional): Fuzzy filter by market (contains, case-insensitive)
- `page` (integer, optional): Legacy offset page (default: 1)
- `page_size` (integer, optional): Number of orders per page (default: 50, max: 100)
- `cursor`, `use_cursor`, `compute_total` (optional): Phase A pagination — see [Perpetual list pagination (Phase A dual-mode)](#perpetual-list-pagination-phase-a-dual-mode)

**Request**:

```http
GET /perpetual/open_orders?app_session_id=perp_session_123&market=BTCYTEST.USD-PERP&page=1&page_size=20
```

**Response**:

```json
{
  "orders": [
    {
      "id": "1234567",
    "order_id": "550e8400-e29b-41d4-a716-446655440000",
    "app_session_id": "perp_session_123",
    "market": "BTCYTEST.USD-PERP",
    "price": "35000.00000000",
    "amount": "0.50000000",
    "origin_amount": "0.50000000",
    "fill_amount": "0.20000000",
    "notional": "17500.00000000",
    "side": "buy",
    "type": "limit",
    "state": "wait",
    "event": "",
    "reason": "",
    "leverage": "10.00000000",
    "created_at": "2023-12-07T10:30:00.000000Z",
    "updated_at": "2023-12-07T10:30:05.000000Z",
    "completed_at": ""
    }
  ],
  "total": 5,
  "page": 1,
  "page_size": 20,
  "next_cursor": "1234520",
  "has_more": false
}
```

**Response Fields**:

- `orders`: Array of open order objects (same format as `/perpetual/orders`)
- `total`: Total number of open orders matching the filter
- `page`: Current page number (`0` in cursor mode)
- `page_size`: Number of items per page
- `next_cursor`, `has_more`: Phase A cursor pagination fields
- Order object fields (same as `/perpetual/orders`):
  - `id`: Internal order record ID
  - `order_id`: Order UUID
  - `app_session_id`: Perpetuals account app session ID
  - `market`: Trading market
  - `price`: Order price
  - `amount`: Current order amount (remaining to fill)
  - `origin_amount`: Original order amount
  - `fill_amount`: Amount that has been filled
  - `notional`: Notional value (origin_amount x price)
  - `side`: Order side (`buy` or `sell`)
  - `type`: Order type (`limit`, `market`)
  - `state`: Order state (typically `wait` for open orders)
  - `event`: Last order event
  - `reason`: Reason for last state change
  - `leverage`: Leverage multiplier
  - `created_at`: Order creation timestamp
  - `updated_at`: Last update timestamp
  - `completed_at`: Completion timestamp (empty for open orders)

**Note**: This endpoint only returns orders with open states (`wait`, `pending`). For complete order history including filled and cancelled orders, use `/perpetual/orders`.

**Status Codes**:

- `200` - Open orders retrieved successfully
- `400` - Invalid query parameters (including malformed `cursor`)
- `401` - Authentication failed
- `500` - Internal server error

---

### GET /perpetual/trades

Retrieve perpetuals trade history with pagination. Returns only the user's own trades with privacy-focused response format.

**Authentication**: Required

**Query Parameters**:

- `app_session_id` (string, required): Perpetuals account app session ID
- `market` (string, optional): Exact filter by trading market
- `market_like` (string, optional): Fuzzy filter by market (contains, case-insensitive)
- `start_time` (string, optional): Filter executed_at >= start_time (RFC3339 or RFC3339Nano)
- `end_time` (string, optional): Filter executed_at <= end_time (RFC3339 or RFC3339Nano)
- `page` (integer, optional): Legacy offset page (default: 1)
- `page_size` (integer, optional): Number of trades per page (default: 50, max: 100)
- `cursor`, `use_cursor`, `compute_total` (optional): Phase A pagination — see [Perpetual list pagination (Phase A dual-mode)](#perpetual-list-pagination-phase-a-dual-mode)

**Request**:

```http
GET /perpetual/trades?app_session_id=perp_session_123&market=BTCYTEST.USD-PERP&page=1&page_size=50
```

**Response**:

```json
{
  "trades": [
    {
      "id": "5721092",
      "order_uuid": "550e8400-e29b-41d4-a716-446655440000",
      "market": "BTCYTEST.USD-PERP",
      "base_asset": "BTC",
      "quote_asset": "USD",
      "amount": "0.50000000",
      "price": "35000.00000000",
      "is_buyer": true,
      "is_maker": false,
      "fee": "0.75000000",
      "exec_type": "trade",
      "executed_at": "2023-12-07T10:30:00.000000Z",
      "created_at": "2023-12-07T10:30:01.000000Z"
    }
  ],
  "total": 456,
  "page": 1,
  "page_size": 50,
  "next_cursor": "1701943800000:5721092",
  "has_more": true
}
```

**Response Fields**:

- `trades`: Array of trade objects
- `total`: Total number of trades matching the filter (legacy; optional on cursor path unless `compute_total=true`)
- `page`: Current page number (`0` in cursor mode)
- `page_size`: Number of items per page
- `next_cursor`, `has_more`: Phase A cursor pagination fields
- Trade object fields:
  - `id`: Trade record ID (string)
  - `order_uuid`: User's own order UUID in this trade (string, matches create order response)
  - `market`: Trading market
  - `base_asset`: Base asset symbol (e.g., "BTC")
  - `quote_asset`: Quote asset symbol (e.g., "USD")
  - `amount`: Trade amount in base asset
  - `price`: Trade execution price
  - `is_buyer`: Whether the user was the buyer (boolean)
  - `is_maker`: Whether the user was the maker/liquidity provider (boolean)
  - `fee`: Trading fee charged to the user
  - `exec_type`: Execution source. One of:
    - `trade` — Normal matched trade (Finex)
    - `liquidation` — IOC liquidation fill or cross liquidation netting
    - `liquidation_takeover` — Position taken over by insurance fund
    - `adl` — ADL (auto-deleveraging) deleveraging
  - `executed_at`: Trade execution timestamp
  - `created_at`: Trade record creation timestamp

**Note**: The API response only exposes the user's own order UUID and does not expose counterparty information for privacy reasons. User-relative flags (`is_buyer`, `is_maker`) indicate the user's role in the trade.

**Status Codes**:

- `200` - Trades retrieved successfully
- `400` - Invalid query parameters (including malformed `cursor`)
- `401` - Authentication failed
- `500` - Internal server error

---

### GET /perpetual/funding-rate/:symbol

Retrieve the current funding rate for a specific perpetuals market, plus the previous funding interval when available.

**Authentication**: Not required (public endpoint)

**Path Parameters**:

- `symbol` (string, required): Trading symbol (e.g., "BTCYTEST.USD-PERP")

**Request**:

```http
GET /perpetual/funding-rate/BTCYTEST.USD-PERP
```

**Response**:

```json
{
  "current_funding_rate": {
    "market": "BTCYTEST.USD-PERP",
    "funding_rate": "0.0001",
    "premium_index": "0.00005",
    "mark_price": "35000.00000000",
    "index_price": "35010.00000000",
    "interval_start": "2023-12-07T10:00:00Z",
    "interval_end": "2023-12-07T14:00:00Z"
  },
  "previous_funding_rate": {
    "market": "BTCYTEST.USD-PERP",
    "funding_rate": "0.00008",
    "premium_index": "0.00004",
    "mark_price": "34950.00000000",
    "index_price": "34960.00000000",
    "interval_start": "2023-12-07T06:00:00Z",
    "interval_end": "2023-12-07T10:00:00Z"
  }
}
```

**Response Fields**:

- `current_funding_rate`: Current funding rate object
  - `market`: Trading market symbol
  - `funding_rate`: Current funding rate (decimal as string)
  - `premium_index`: Premium index value (decimal as string)
  - `mark_price`: Current mark price
  - `index_price`: Current index price
  - `interval_start`: Start of the current funding interval (ISO 8601 format)
  - `interval_end`: End of the current funding interval (ISO 8601 format)
  - `created_at`: When the current funding rate was recorded (ISO 8601 format)
- `previous_funding_rate`: Previous funding interval object (may be omitted if not available)
  - `market`: Trading market symbol
  - `funding_rate`: Previous interval funding rate (decimal as string)
  - `premium_index`: Previous interval premium index
  - `mark_price`: Mark price used for previous interval
  - `index_price`: Index price used for previous interval
  - `interval_start`: Start of the previous funding interval (ISO 8601 format)
  - `interval_end`: End of the previous funding interval (ISO 8601 format)
  - `created_at`: When the previous funding rate was recorded (ISO 8601 format)

**Status Codes**:

- `200` - Funding rate retrieved successfully
- `400` - Invalid symbol or validation failed
- `500` - Internal server error

---

### GET /perpetual/funding-rates

Retrieve funding rate history with Phase A dual-mode pagination.

**Authentication**: Not required (public endpoint)

**Query Parameters**:

- `symbol` (string, optional): Filter by trading market
- `page` (integer, optional): Legacy offset page (default: 1)
- `page_size` (integer, optional): Number of rates per page (default: 50, max: 100)
- `cursor`, `use_cursor`, `compute_total` (optional): See [Perpetual list pagination (Phase A dual-mode)](#perpetual-list-pagination-phase-a-dual-mode)

**Request** (cursor first page):

```http
GET /perpetual/funding-rates?symbol=BTCYTEST.USD-PERP&use_cursor=true&page_size=20
```

**Request** (legacy offset):

```http
GET /perpetual/funding-rates?symbol=BTCYTEST.USD-PERP&page=1&page_size=20
```

**Response**:

```json
{
  "funding_rates": [
    {
      "market": "BTCYTEST.USD-PERP",
      "funding_rate": "0.0001",
      "premium_index": "0.00005",
      "mark_price": "35000.00000000",
      "index_price": "35010.00000000",
      "interval_start": "2023-12-07T10:00:00Z",
      "interval_end": "2023-12-07T14:00:00Z"
    }
  ],
  "total": 100,
  "page": 1,
  "page_size": 20,
  "next_cursor": "1701943800000:42",
  "has_more": true
}
```

**Response Fields**:

- `funding_rates`: Array of funding rate objects (same structure as `/perpetual/funding-rate/:symbol`)
- `total`: Total number of funding rates matching criteria (legacy; optional on cursor path unless `compute_total=true`)
- `page`: Current page number (`0` in cursor mode)
- `page_size`: Number of rates per page
- `next_cursor`, `has_more`: Phase A cursor pagination fields

**Status Codes**:

- `200` - Funding rates retrieved successfully
- `400` - Invalid query parameters (including malformed `cursor`)
- `500` - Internal server error

---

### GET /perpetual/account/funding-payments

Retrieve funding payment history for the specified app session with pagination.

**Authentication**: Required

**Query Parameters**:

- `app_session_id` (string, required): App session identifier passed by frontend
- `interval_start` (string, optional): Filter by funding interval start time (ISO 8601 format)
- `page` (integer, optional): Page number (default: 1)
- `page_size` (integer, optional): Number of payments per page (default: 50, max: 100)

**Request**:

```http
GET /perpetual/account/funding-payments?app_session_id=perp_session_123&page=1&page_size=20
```

**Response**:

```json
{
  "funding_payments": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "market": "BTCYTEST.USD-PERP",
      "account_id": "550e8400-e29b-41d4-a716-446655440001",
      "position_id": "550e8400-e29b-41d4-a716-446655440002",
      "side": "long",
      "position_size": "1.50000000",
      "mark_price": "35000.00000000",
      "funding_rate": "0.0001",
      "funding_amount": "5.25000000",
      "interval_start": "2023-12-07T10:00:00Z",
      "interval_end": "2023-12-07T14:00:00Z",
      "created_at": "2023-12-07T14:00:00Z"
    }
  ],
  "total": 45,
  "page": 1,
  "page_size": 20
}
```

**Response Fields**:

- `funding_payments`: Array of funding payment objects
  - `id`: Funding payment ID (UUID)
  - `market`: Trading market
  - `account_id`: Perpetuals account ID (UUID)
  - `position_id`: Position ID (UUID)
  - `side`: Position side (`long` or `short`)
  - `position_size`: Position size at the time of funding payment
  - `mark_price`: Mark price at the time of funding payment
  - `funding_rate`: Funding rate applied
  - `funding_amount`: Funding payment amount (positive for long positions receiving payment, negative for paying)
  - `interval_start`: Start of the funding interval (ISO 8601 format)
  - `interval_end`: End of the funding interval (ISO 8601 format)
  - `created_at`: When the funding payment was recorded (ISO 8601 format)
- `total`: Total number of funding payments matching criteria
- `page`: Current page number
- `page_size`: Number of payments per page

**Status Codes**:

- `200` - Funding payments retrieved successfully
- `400` - Invalid query parameters (including malformed `cursor`) or missing app_session_id
- `401` - Authentication failed
- `500` - Internal server error

---

### GET /perpetual/position/funding-payments

Retrieve funding payment history for a specific position within the specified app session, with pagination.

**Authentication**: Required

**Query Parameters**:

- `app_session_id` (string, required): App session identifier passed by frontend
- `position_id` (string, required): Position ID (UUID)
- `interval_start` (string, optional): Filter by funding interval start time (ISO 8601 format)
- `page` (integer, optional): Page number (default: 1)
- `page_size` (integer, optional): Number of payments per page (default: 50, max: 100)

**Request**:

```http
GET /perpetual/position/funding-payments?app_session_id=perp_session_123&position_id=550e8400-e29b-41d4-a716-446655440002&page=1&page_size=20
```

**Response**: Same format as `/perpetual/account/funding-payments` (array of funding payment objects with pagination metadata)

**Status Codes**:

- `200` - Funding payments retrieved successfully
- `400` - Invalid query parameters (including malformed `cursor`) or missing position_id/app_session_id
- `401` - Authentication failed
- `500` - Internal server error

---

## Account Transfers

The account transfer API provides endpoints for transferring funds between spot and perpetuals accounts using an asynchronous Saga choreography pattern for reliability and eventual consistency.

### POST /accounts/transfer

Initiate an asynchronous fund transfer between spot and perpetuals accounts.

**Authentication**: Required (JWT or API Key)

**Request Body**:

```json
{
  "app_session_id": "user-session-123",
  "source_account_type": "spot",
  "dest_account_type": "perps",
  "asset_symbol": "USDT",
  "amount": "1000.50"
}
```

**Request Parameters**:

- `app_session_id` (string, required): User's session ID (shared between spot and perps accounts)
- `source_account_type` (string, required): Source account type - `spot` or `perps`
- `dest_account_type` (string, required): Destination account type - `spot` or `perps`
- `asset_symbol` (string, required): Stablecoin asset to transfer (e.g., "USDT"). Only stablecoin assets that are active on both spot and perps sides are allowed.
- `amount` (string, required): Amount to transfer (decimal format, must be positive)

**Asset Restrictions**:

- Only **stablecoin** assets can be transferred between spot and perps accounts.
- The asset must be active and marked as stablecoin on both the spot (`spot_assets.is_stablecoin = true`) and perps (`perps_assets.is_stablecoin = true`) sides.
- Use `GET /perpetual/transfer-assets` to fetch the list of transferable stablecoin assets.

**Perpetual → Spot (perps as source)**:

When `source_account_type` is `perps` and `dest_account_type` is `spot`, the Portfolio Manager applies a **per-request maximum debit** for user transfers (not the same as raw wallet `available_balance`):

`transferable_max = max(0, min(total_balance - locked_balance, available_balance)) × 80%`

- `available_balance` matches balance-query semantics (includes unrealized PnL where applicable).
- Applies to this API’s saga path and other user-facing perpetuals→spot debits. Internal operations (e.g. insurance fund debits, ADL shortfall) use a separate code path **without** this 80% multiplier.
- If `amount` exceeds `transferable_max`, the transfer fails at the source step with an insufficient-balance style error (same category as not having enough funds).

**Balance Notifications**:

- After a successful transfer, both the source and destination accounts publish balance update events.
- Spot side publishes `EventSpotBalanceUpdated` to the `spot_account.balance_update` WebSocket channel.
- Perps side publishes `EventPerpetualsBalanceUpdated` to the `perpetuals_account.balance_update` WebSocket channel.
- These notifications allow the frontend to update displayed balances in real-time without polling.

**Response** (202 Accepted):

```json
{
  "transfer_id": "550e8400-e29b-41d4-a716-446655440000",
  "message": "Transfer initiated successfully. You will be notified when the transfer completes."
}
```

**Response Fields**:

- `transfer_id`: Unique UUID for tracking the transfer
- `message`: Human-readable status message

**Status Codes**:

- `202` - Transfer request accepted and queued for processing
- `400` - Invalid request parameters (non-stablecoin asset, invalid amount, etc.)
- `401` - Authentication failed (missing or invalid JWT/API key)
- `500` - Failed to publish transfer command to Kafka
- `503` - Transfer validation service temporarily unavailable

**Error Responses**:

**400 Bad Request** - Invalid parameters:

```json
{
  "error": "invalid_amount",
  "message": "Amount must be positive"
}
```

Possible error codes:

- `invalid_request_format` - Malformed JSON or missing required fields
- `invalid_amount` - Amount is zero, negative, or invalid decimal
- `invalid_source_type` - Source account type must be 'spot' or 'perps'
- `invalid_dest_type` - Destination account type must be 'spot' or 'perps'
- `same_account_type` - Source and destination types cannot be the same
- `invalid_asset` - Asset is not supported for spot-perps transfer (not a stablecoin, not active, or not found)
- `amount_below_minimum` - Amount is below minimum transfer amount for the asset

**401 Unauthorized**:

```json
{
  "error": "missing_wallet_address",
  "message": "Wallet address not found in authentication context"
}
```

**500 Internal Server Error**:

```json
{
  "error": "transfer_failed",
  "message": "Failed to initiate transfer. Please try again or contact support."
}
```

**Transfer Flow**:

The transfer follows a 7-step asynchronous process:

1. **Submit Request**: Client sends transfer request to Trading API
2. **Command Published**: Trading API validates and publishes `CommandTransferFunds` to Kafka
3. **Source Processing**: Source Portfolio Manager (spot/perp) deducts funds via Raft FSM
4. **Event Propagation**: Source PM publishes `EventTransferUpdated(state=source_completed)`
5. **Destination Processing**: Destination PM receives event and adds funds
6. **Completion**: Destination PM publishes `EventTransferUpdated(state=dest_completed)`
7. **Notification**: User receives WebSocket notification on `transfer_updates` channel

**Transfer States**:


| State                 | Description                                              |
| --------------------- | -------------------------------------------------------- |
| `pending`             | Transfer initiated, waiting for source processing        |
| `source_completed`    | Funds deducted from source account                       |
| `dest_completed`      | Funds added to destination account (final success state) |
| `failed`              | Transfer failed before any funds were moved              |
| `compensation_needed` | Destination failed, initiating rollback                  |
| `compensated`         | Source account refunded after destination failure        |
| `compensation_failed` | Compensation failed (requires manual intervention)       |


**WebSocket Notification**:

When the transfer completes (or fails), you receive a real-time notification:

```json
{
  "channel": "transfer_updates",
  "data": {
    "transfer_id": "550e8400-e29b-41d4-a716-446655440000",
    "owner_address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
    "app_session_id": "user-session-123",
    "source_type": "spot",
    "dest_type": "perps",
    "asset_symbol": "USDT",
    "amount": "1000.50",
    "state": "dest_completed",
    "completed_at": "2026-02-16T10:30:45.123456Z"
  }
}
```

**Important Notes**:

- **Stablecoin Only**: Only stablecoin assets (where `is_stablecoin = true` and `is_active = true` on both spot and perps) can be transferred
- **Perp → Spot cap**: Outgoing perpetuals transfers are limited per request to 80% of `min(total - locked, available)` (see **Perpetual → Spot** above); plan `amount` accordingly or split into multiple transfers
- **Asynchronous Processing**: The API returns immediately (202 Accepted) while the transfer processes in the background
- **Eventual Consistency**: The transfer follows a Saga pattern with automatic compensation on failure
- **Idempotency**: Duplicate requests with the same parameters are safely handled
- **Real-time Updates**: Subscribe to WebSocket `transfer_updates` channel for completion notifications. Balance update notifications are also pushed to `spot_account.balance_update` and `perpetuals_account.balance_update` channels.
- **Automatic Rollback**: If the destination fails, the system automatically refunds the source account
- **Source and destination types must be different** (cannot transfer within the same account type)

**Example Request**:

```bash
curl -X POST https://api.neodax.com/accounts/transfer \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "app_session_id": "user-session-123",
    "source_account_type": "spot",
    "dest_account_type": "perps",
    "asset_symbol": "USDT",
    "amount": "1000.00"
  }'
```

---

### POST /accounts/send

Send funds from your spot account to another user's spot account.

**Authentication**: Required (JWT or API Key)

**Request Body**:

```json
{
  "app_session_id": "sender-session-123",
  "recipient_app_session_id": "recipient-session-456",
  "asset_symbol": "USDT",
  "amount": "100.50"
}
```

**Request Parameters**:

- `app_session_id` (string, required): Sender's session ID
- `recipient_app_session_id` (string, required): Recipient's session ID
- `asset_symbol` (string, required): Asset to send (e.g., "USDT", "BTC"). Case-insensitive (uppercased server-side).
- `amount` (string, required): Amount to send (decimal format, must be positive)

**Response** (200 OK):

```json
{
  "send_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Response Fields**:

- `send_id`: Unique UUID identifying this send operation

**Status Codes**:

- `200` - Send completed successfully
- `400` - Invalid request parameters (invalid amount, send to self, missing fields)
- `401` - Authentication failed (missing or invalid JWT/API key)
- `403` - Not authorized to send from this account
- `404` - Sender or recipient account not found
- `409` - Insufficient available balance
- `500` - Internal server error

**Error Responses**:

**400 Bad Request** - Invalid parameters:

```json
{
  "error": "invalid_amount",
  "message": "Amount must be positive"
}
```

```json
{
  "error": "invalid_request",
  "message": "cannot send funds to yourself"
}
```

Possible error codes:

- `invalid_request_format` - Malformed JSON or missing required fields
- `invalid_amount` - Amount is zero or negative
- `invalid_request` - Send to self or other invalid argument

**403 Forbidden**:

```json
{
  "error": "permission_denied",
  "message": "Not authorized to send from this account"
}
```

**404 Not Found**:

```json
{
  "error": "account_not_found",
  "message": "Sender or recipient account not found"
}
```

**409 Conflict**:

```json
{
  "error": "insufficient_balance",
  "message": "Insufficient available balance"
}
```

**Important Notes**:

- **Synchronous Processing**: Unlike transfers between spot and perps accounts, sends are processed synchronously. The response indicates final success or failure.
- **Atomic**: Both the sender debit and recipient credit are applied in a single atomic Raft batch. Either both succeed or neither does.
- **No Fees**: There are currently no fees on sends.
- **Balance Notifications**: Both sender and recipient receive a `balance_updates` WebSocket notification after a successful send.

**Example Request**:

```bash
curl -X POST https://api.neodax.com/accounts/send \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "app_session_id": "sender-session-123",
    "recipient_app_session_id": "recipient-session-456",
    "asset_symbol": "USDT",
    "amount": "100.00"
  }'
```

---

### GET /accounts/send_history

Retrieve send history for the authenticated user (both sent and received) with pagination.

**Authentication**: Required (JWT or API Key)

**Query Parameters**:

- `app_session_id` (string, required): User's session ID
- `page` (integer, optional): Page number (default: 1)
- `page_size` (integer, optional): Number of records per page (default: 20, max: 100)

**Request**:

```http
GET /accounts/send_history?app_session_id=user-session-123&page=1&page_size=20
```

**Response** (200 OK):

```json
{
  "sends": [
    {
      "send_id": "550e8400-e29b-41d4-a716-446655440000",
      "sender_app_session_id": "user-session-123",
      "sender_owner": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
      "recipient_app_session_id": "recipient-session-456",
      "recipient_owner": "0xAnotherWalletAddress",
      "asset_symbol": "USDT",
      "amount": "100.00",
      "created_at": "2026-03-18T12:00:00Z"
    }
  ],
  "total": 1
}
```

**Response Fields**:

- `sends`: Array of send records
  - `send_id`: Unique UUID of the send
  - `sender_app_session_id`: Sender's session ID
  - `sender_owner`: Sender's wallet address
  - `recipient_app_session_id`: Recipient's session ID
  - `recipient_owner`: Recipient's wallet address
  - `asset_symbol`: Sent asset symbol
  - `amount`: Send amount (decimal string)
  - `created_at`: When the send was executed (RFC 3339 format)
- `total`: Total number of matching sends (for pagination)

**Status Codes**:

- `200` - Send history retrieved successfully
- `400` - Missing `app_session_id`
- `401` - Authentication failed
- `404` - Account not found or not authorized to view (permission denied is mapped to 404 to avoid leaking account existence)
- `500` - Internal server error

**Example Request**:

```bash
curl -X GET "https://api.neodax.com/accounts/send_history?app_session_id=user-session-123&page=1&page_size=20" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

---

### GET /accounts/transfers

Retrieve transfer history for the authenticated user with pagination and optional filters.

**Authentication**: Required (JWT or API Key)

**Status**: NOT YET IMPLEMENTED -- planned for upcoming release.

**Query Parameters**:

- `app_session_id` (string, required): User's session ID (from auth context)
- `asset_symbol` (string, optional): Filter by asset (e.g., "YTEST.USD")
- `state` (string, optional): Filter by state (`pending`, `dest_completed`, `failed`, `compensated`)
- `limit` (integer, optional): Number of records per page (default: 20, max: 100)
- `offset` (integer, optional): Pagination offset (default: 0)

**Request**:

```http
GET /accounts/transfers?app_session_id=user-session-123&asset_symbol=YTEST.USD&limit=20&offset=0
```

**Response** (200 OK):

```json
{
  "transfers": [
    {
      "transfer_id": "550e8400-e29b-41d4-a716-446655440000",
      "owner_address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
      "app_session_id": "user-session-123",
      "source_type": "spot",
      "dest_type": "perps",
      "asset_symbol": "YTEST.USD",
      "amount": "1000.00",
      "state": "dest_completed",
      "created_at": "2026-02-24T10:00:00Z",
      "source_completed_at": "2026-02-24T10:00:01Z",
      "dest_completed_at": "2026-02-24T10:00:02Z",
      "completed_at": "2026-02-24T10:00:02Z"
    },
    {
      "transfer_id": "660e8400-e29b-41d4-a716-446655440001",
      "owner_address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
      "app_session_id": "user-session-123",
      "source_type": "perps",
      "dest_type": "spot",
      "asset_symbol": "YTEST.USD",
      "amount": "500.00",
      "state": "dest_completed",
      "created_at": "2026-02-24T11:00:00Z",
      "source_completed_at": "2026-02-24T11:00:01Z",
      "dest_completed_at": "2026-02-24T11:00:02Z",
      "completed_at": "2026-02-24T11:00:02Z"
    }
  ],
  "total": 42
}
```

**Response Fields**:

- `transfers`: Array of transfer records
  - `transfer_id`: Unique UUID of the transfer
  - `owner_address`: User's wallet address
  - `app_session_id`: User's session ID
  - `source_type`: Source account type (`spot` or `perps`)
  - `dest_type`: Destination account type (`spot` or `perps`)
  - `asset_symbol`: Transferred asset symbol
  - `amount`: Transfer amount
  - `state`: Current transfer state (see Transfer States in POST section above)
  - `failure_reason`: Reason for failure (only present if state is `failed` or `compensation_failed`)
  - `created_at`: When the transfer was initiated
  - `source_completed_at`: When source deduction completed
  - `dest_completed_at`: When destination credit completed
  - `completed_at`: When the transfer fully completed
- `total`: Total number of matching transfers

**Status Codes**:

- `200` - Transfer history retrieved successfully
- `400` - Invalid query parameters (including malformed `cursor`)
- `401` - Authentication failed
- `500` - Internal server error

**Example Request**:

```bash
curl -X GET "https://api.neodax.com/accounts/transfers?app_session_id=user-session-123&limit=20" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

---

## Clearnet (Blockchain Integration)

The Clearnet API provides endpoints for managing blockchain transactions, deposits, and app sessions. These endpoints handle the integration between the exchange and the blockchain layer.

### GET /clearnet/sessions

Retrieve all clearnet app sessions for the authenticated user.

**Authentication**: Required

**Response**:

```json
[
  {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "owner": "0x1234567890abcdef1234567890abcdef12345678",
    "channel_id": "0xchannel123",
    "state": "active",
    "created_at": "2023-12-01T08:00:00.000000Z"
  }
]
```

**Response Fields**:

- `id`: App session ID (UUID)
- `owner`: Owner's Ethereum wallet address
- `channel_id`: Channel ID associated with this session
- `state`: Session state (e.g., `active`, `closed`)
- `created_at`: Session creation timestamp

**Status Codes**:

- `200` - Sessions retrieved successfully
- `401` - Authentication failed
- `500` - Internal server error

---

### GET /clearnet/sessions/:session_id

Retrieve details for a specific clearnet app session.

**Authentication**: Required

**Path Parameters**:

- `session_id` (string, required): App session ID (UUID)

**Request**:

```http
GET /clearnet/sessions/550e8400-e29b-41d4-a716-446655440000
```

**Response**:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "owner": "0x1234567890abcdef1234567890abcdef12345678",
  "channel_id": "0xchannel123",
  "state": "active",
  "created_at": "2023-12-01T08:00:00.000000Z"
}
```

**Status Codes**:

- `200` - Session retrieved successfully
- `401` - Authentication failed
- `404` - Session not found
- `500` - Internal server error

---

### GET /clearnet/transactions

Retrieve blockchain transactions by transaction IDs.

**Authentication**: Required

**Query Parameters**:

- `ids` (string, required): Comma-separated list of transaction IDs

**Request**:

```http
GET /clearnet/transactions?ids=tx1,tx2,tx3
```

**Response**:

```json
[
  {
    "id": "tx1",
    "session_id": "550e8400-e29b-41d4-a716-446655440000",
    "type": "deposit",
    "asset_symbol": "USDT",
    "amount": "1000.00000000",
    "status": "completed",
    "transaction_hash": "0xabcdef1234567890",
    "created_at": "2023-12-07T10:30:00.000000Z",
    "completed_at": "2023-12-07T10:31:00.000000Z"
  }
]
```

**Response Fields**:

- `id`: Transaction ID
- `session_id`: App session ID (UUID)
- `type`: Transaction type (`deposit`, `withdrawal`)
- `asset_symbol`: Asset symbol
- `amount`: Transaction amount
- `status`: Transaction status (`pending`, `completed`, `failed`)
- `transaction_hash`: Blockchain transaction hash
- `created_at`: Transaction creation timestamp
- `completed_at`: Transaction completion timestamp

**Status Codes**:

- `200` - Transactions retrieved successfully
- `400` - Missing or invalid IDs parameter
- `401` - Authentication failed
- `500` - Internal server error

---

### GET /clearnet/sessions/:session_id/transactions

Retrieve all blockchain transactions for a specific app session.

**Authentication**: Required

**Path Parameters**:

- `session_id` (string, required): App session ID (UUID)

**Request**:

```http
GET /clearnet/sessions/550e8400-e29b-41d4-a716-446655440000/transactions
```

**Response**: Same format as `/clearnet/transactions`

**Status Codes**:

- `200` - Transactions retrieved successfully
- `401` - Authentication failed
- `404` - Session not found
- `500` - Internal server error

---

### GET /clearnet/session/payload

Get the payload data required for creating a new clearnet app session on the blockchain.

**Authentication**: Required

**Query Parameters**:

- `account_type` (string, required): Account type - `spot` or `perps`
- `nonce` (integer, required): Nonce for transaction ordering

**Request**:

```http
GET /clearnet/session/payload?account_type=spot&nonce=1
```

**Response**:

```json
{
  "payload": "0x...",
  "expires_at": "2023-12-07T10:35:00Z"
}
```

**Response Fields**:

- `payload`: Hex-encoded payload data to be signed
- `expires_at`: When the payload expires

**Status Codes**:

- `200` - Payload generated successfully
- `400` - Invalid parameters
- `401` - Authentication failed
- `500` - Internal server error

---

### POST /clearnet/session

Create a new clearnet app session on the blockchain.

**Authentication**: Required

**Request Body**:

```json
{
  "account_type": "spot",
  "nonce": 1,
  "signature": "0x1234567890abcdef..."
}
```

**Request Parameters**:

- `account_type` (string, required): Account type - `spot` or `perps`
- `nonce` (integer, required): Nonce used in payload generation
- `signature` (string, required): Ethereum signature of the payload

**Response**:

```json
{
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "channel_id": "0xchannel123",
  "status": "pending",
  "message": "App session creation request submitted"
}
```

**Response Fields**:

- `session_id`: Created app session ID (UUID)
- `channel_id`: Channel ID for this session
- `status`: Initial status (`pending`)
- `message`: Confirmation message

**Status Codes**:

- `200` - Session creation request accepted
- `400` - Invalid request or signature verification failed
- `401` - Authentication failed
- `500` - Internal server error

---

### GET /clearnet/sessions/:session_id/deposit/payload

Get the payload data required for depositing funds to a clearnet app session.

**Authentication**: Required

**Path Parameters**:

- `session_id` (string, required): App session ID (UUID)

**Query Parameters**:

- `asset_symbol` (string, required): Asset to deposit (e.g., "USDT", "BTC")
- `amount` (string, required): Amount to deposit (decimal format)
- `nonce` (integer, required): Nonce for transaction ordering

**Request**:

```http
GET /clearnet/sessions/550e8400-e29b-41d4-a716-446655440000/deposit/payload?asset_symbol=USDT&amount=1000.00&nonce=1
```

**Response**:

```json
{
  "payload": "0x...",
  "expires_at": "2023-12-07T10:35:00Z"
}
```

**Response Fields**:

- `payload`: Hex-encoded payload data to be signed
- `expires_at`: When the payload expires

**Status Codes**:

- `200` - Payload generated successfully
- `400` - Invalid parameters
- `401` - Authentication failed
- `404` - Session not found
- `500` - Internal server error

---

### POST /clearnet/sessions/:session_id/deposit

Request a deposit to a clearnet app session.

**Authentication**: Required

**Path Parameters**:

- `session_id` (string, required): App session ID (UUID)

**Request Body**:

```json
{
  "asset_symbol": "USDT",
  "amount": "1000.00000000",
  "nonce": 1,
  "signature": "0x1234567890abcdef..."
}
```

**Request Parameters**:

- `asset_symbol` (string, required): Asset to deposit
- `amount` (string, required): Amount to deposit (decimal format)
- `nonce` (integer, required): Nonce used in payload generation
- `signature` (string, required): Ethereum signature of the payload

**Response**:

```json
{
  "deposit_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "pending",
  "message": "Deposit request submitted to blockchain"
}
```

**Response Fields**:

- `deposit_id`: Deposit transaction ID (UUID)
- `status`: Initial status (`pending`)
- `message`: Confirmation message

**Status Codes**:

- `200` - Deposit request accepted
- `400` - Invalid request or signature verification failed
- `401` - Authentication failed
- `404` - Session not found
- `500` - Internal server error

---

## Error Handling

All endpoints return consistent error responses with the following structure:

### Authentication Service Errors:

```json
{
  "error": "error_code",
  "message": "Human-readable error message",
  "code": 400
}
```

### Trading API Service Errors:

```json
{
  "error": "error_code",
  "message": "Detailed error description",
  "placeholders": {
    "available": "9.62",
    "required": "9.6256"
  }
}
```

- `placeholders` (object, optional): Populated for some `**POST /perpetual/order**`, `**POST /perpetual/leverage**`, and `**POST /spot/order**` (async) failures when the PM lock response includes `error_placeholders` or the gateway forwards the same shape. Keys are snake_case; values are **display-formatted** decimal strings (aligned with `message`), not guaranteed canonical internal precision. Omitted when there is no structured context.

### Common Error Codes:

**Authentication Service:**

- `invalid_request` - Invalid request body format
- `invalid_wallet_address` - Invalid Ethereum wallet address
- `internal_error` - Internal server error
- `challenge_expired` - Challenge not found or expired
- `invalid_signature` - Signature verification failed
- `invalid_refresh_token` - Refresh token invalid or expired
- `missing_token` - Authorization header missing
- `invalid_token_format` - Invalid Authorization header format
- `unauthorized` - Authentication required

**Trading API Service:**

*Portfolio Manager gRPC `error_code*`: On responses from spot/perp PM (`LockSpotAssetResponse`, `UnlockSpotAssetResponse`, `LockPerpAssetResponse`, `SetInitialLeverageResponse` in `internal/protocol/proto/spot_pm.proto` and `perp_pm.proto`), when `success` is `false`, the `error_code` field is a snake_case string (empty when `success` is `true`). Trading API copies it to REST JSON `error` for order lock (`POST /spot/order`, `POST /perpetual/order`) and leverage (`POST /perpetual/leverage`). If PM returns `success=false` with an empty `error_code` on lock, the gateway uses `lock_funds_rejected`.

*Portfolio Manager gRPC `error_placeholders*`: On `LockPerpAssetResponse` and `SetInitialLeverageResponse`, PM may attach a string-to-string map (`error_placeholders` in proto). When non-empty, Trading API adds REST JSON field `**placeholders**` with the same map. Keys are snake_case; values are **display-formatted** decimal strings (same rules as the human-readable `message`), for clients that build localized copy. The field is **omitted** when PM sends no placeholders.

*Per-endpoint `error` → `placeholders` keys* (when PM provides them):


| REST `error` (from PM)                | `placeholders` keys                                                                          |
| ------------------------------------- | -------------------------------------------------------------------------------------------- |
| `insufficient_margin`                 | `available`, `required`                                                                      |
| `insufficient_margin_for_fees`        | `available`, `required`                                                                      |
| `insufficient_balance` (set leverage) | `available`, `required`                                                                      |
| `leverage_exceeds_safe_margin`        | `allocated`, `required` (`required` = minimum margin threshold including maintenance buffer) |


*Gateway and other Trading API codes:*

- `missing_parameter` - Required parameter is missing
- `invalid_request_format` - Request body format is invalid
- `validation_failed` - Request validation failed (common on perpetual order and other endpoints when the failure is not a typed PM lock rejection)
- `order_creation_failed` - Spot order creation failed for reasons other than a typed PM lock rejection (message carries detail)
- `lock_funds_rejected` - PM returned `success=false` on `LockSpotAsset` or `LockPerpAsset` without a specific `error_code` (fallback)
- `missing_channel_id` - Channel ID parameter is required
- `missing_symbol` - Symbol parameter is required
- `symbol_not_found` - Requested symbol not found
- `missing_order_uuid` - Order UUID is required
- `invalid_order_uuid` - Order UUID format is invalid
- `internal_error` - Internal server error occurred

`*LockSpotAssetResponse` (`error_code`):*

- `service_unavailable` - Spot service not initialized on PM
- `invalid_amount` - Amount string invalid or non-positive
- `invalid_price` - Price string invalid
- `invalid_side` - Side is not buy/sell
- `no_permitted` - Caller is not the account owner
- `account_not_active` - Spot account is not active
- `account_not_exists` - Account not found
- `insufficient_balance` - Not enough available balance to lock
- `market_price_unavailable` - No usable market price (e.g. market order) or reference price issue
- `market_not_found` - Unknown market symbol
- `unsupported_order_type` - Order type not supported for spot lock
- `missing_limit_price` - Limit order requires a non-zero price
- `limit_price_deviation_exceeded` - Limit price outside max deviation vs reference
- `fee_rate_unavailable` - Could not resolve maker/taker fee rate
- `lock_failed` - Lock failed (validation/Raft/other; see `message`)

*`UnlockSpotAssetResponse` (`error_code`):*

- `service_unavailable` - Spot service not initialized on PM
- `invalid_amount` - Amount invalid, non-positive, or bad unlock delta
- `insufficient_locked_balance` - Cannot unlock more than locked
- `unlock_failed` - Unlock failed (validation/Raft/other; see `message`)

*`LockPerpAssetResponse` (`error_code`; optional `error_placeholders` → REST `placeholders`):*

- `service_unavailable` - Perpetuals service unavailable on PM
- `invalid_amount` - Amount invalid, non-positive, or truncated to zero
- `invalid_leverage_value` - Request leverage not a number ≥ 1
- `leverage_exceeds_max` - Leverage above market maximum
- `invalid_side` - Side is not buy/sell
- `invalid_direction` - Direction not long/short/both
- `account_not_exists` - Account not found
- `no_permitted` - Caller is not the account owner
- `account_locked_for_liquidation` - Cross account locked for liquidation
- `fee_rate_unavailable` - Could not load maker/taker fee rate
- `hedge_direction_required` - HEDGE mode: direction must be long or short, not both
- `oneway_direction_both_required` - ONE_WAY mode: direction must be both
- `no_position_to_close` - Closing order but no position
- `position_locked_for_liquidation` - Isolated position locked for liquidation
- `insufficient_position` - Close size exceeds available position
- `mark_price_unavailable` - Mark price needed but unavailable
- `invalid_limit_price` - Limit price missing or non-positive
- `limit_price_deviation_exceeded` - Limit price outside max deviation vs reference (used for the same reference as the check: mark price if available, otherwise last trade). Human-readable `message` is one of: `Order price cannot be more than {negative_dev} below the mark price ({mark_price}).` when the limit is too low, or `Order price cannot be more than {positive_dev} above the mark price ({mark_price}).` when too high. `placeholders`: too low → `negative_dev`, `mark_price` (ratio limits as display percentages, e.g. `25%`; `mark_price` uses quote display formatting); too high → `positive_dev`, `mark_price`.
- `insufficient_margin_for_fees` - Not enough margin for closing fees (placeholders: `available`, `required`)
- `lock_position_failed` - Failed to lock position quantity
- `initial_margin_calculation_failed` - Initial margin computation failed
- `insufficient_margin` - Not enough margin for open (margin + fees) (placeholders: `available`, `required`)
- `lock_margin_failed` - Failed to reserve margin

*`SetInitialLeverageResponse` (`error_code`; optional `error_placeholders` → REST `placeholders`):*

- `missing_app_session_id` - `app_session_id` empty
- `missing_market` - `market` empty
- `missing_user_address` - `user_address` empty
- `service_unavailable` - Perpetuals service unavailable
- `invalid_leverage_value` - Leverage not a number ≥ 1
- `leverage_exceeds_max` - Above market max leverage
- `account_not_exists` - Account missing
- `no_permitted` - Not the account owner
- `account_not_active` - Account not active
- `open_orders_check_failed` - Could not verify open orders
- `order_exists` - Open orders exist for this market
- `insufficient_balance` - Not enough available collateral to apply higher leverage when positions require extra margin (placeholders: `available`, `required`)
- `leverage_exceeds_safe_margin` - New leverage would put allocated margin below maintenance buffer for a position (placeholders: `allocated`, `required`)
- `set_leverage_failed` - FSM update or DB persist failed (generic; usually no placeholders)

### HTTP Status Codes:

- `200` - Success
- `400` - Bad Request (client error)
- `401` - Unauthorized (authentication required/failed)
- `404` - Not Found (resource doesn't exist)
- `500` - Internal Server Error

---

## WebSocket API

The trading API also supports WebSocket connections for real-time data and RPC calls. WebSocket endpoints provide:

- Real-time position updates
- Order status notifications
- Market data streams
- RPC methods for account operations
- **Margin call notifications** (perpetuals)

WebSocket documentation is available separately.

### Perpetuals Liquidation Warning Notifications

For cross-margin perpetual accounts, the system pushes real-time liquidation warnings by threshold levels.

**Channel**: `private.{userAddress}`

**Liquidation Warning** (`perpetuals_account.liquidation_warning`):

Sent when danger ratio reaches configured thresholds (`80%`, `90%`, `95%`).

```json
{
  "type": "perpetuals_account.liquidation_warning",
  "app_session_id": "user-session-123",
  "owner": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  "warning_level": "final",
  "reminder_type": "first_cross",
  "margin_ratio": "95.0000",
  "threshold": "95",
  "triggered_at": "2026-04-06T10:00:00Z",
  "next_remind_at": "2026-04-06T11:00:00Z"
}
```

**Fields**:

- `warning_level`: `early` | `strong` | `final`
- `reminder_type`: `first_cross` | `hourly`
- `margin_ratio`: danger ratio percent (`maintenance_margin / equity * 100`)
- `threshold`: triggered threshold (`80`, `90`, `95`)

**Trigger Rules**:

1. `>= 80%`: first cross per natural day triggers once
2. `>= 90%`: first cross per natural day triggers once
3. `>= 95%`: first cross per natural day triggers once, then hourly reminders while still `>= 95%`
4. If ratio drops below `95%`, hourly reminders stop
5. If ratio reaches `>= 100%`, warning popup is suppressed (liquidation path takes priority)

---

## Rate Limiting

Rate limiting is enforced to ensure fair usage:

- Authentication middleware includes rate limiting functionality
- Limits are configurable per endpoint and user type
- Rate limit headers are included in responses

---

## Data Types

### Decimal Values

All monetary and quantity values are represented as strings in decimal format to maintain precision:

```json
{
  "amount": "1.50000000",
  "price": "35000.00000000"
}
```

### Timestamps

Timestamps are provided in milliseconds since Unix epoch:

```json
{
  "server_time": 1640995200000
}
```

### UUIDs

Order UUIDs follow standard UUID format:

```json
{
  "order_uuid": "550e8400-e29b-41d4-a716-446655440000"
}
```

---

## Implementation Notes

### Event-Driven Architecture

The NeoDAX platform uses an event-driven architecture with Kafka as the primary message bus:

**Spot Trading:**

- Spot order commands are published to Kafka topics
- Order execution events are consumed from Kafka
- Full Kafka integration completed

**Perpetuals Trading:**

- Full Kafka Integration:
  - All perpetual order commands use Kafka CommandBus
  - WebSocket/RPC endpoints: Kafka
  - REST API endpoints (`POST /perpetual/order`, `DELETE /perpetual/order`): Kafka
  - All event flows (trade execution, position updates, etc.) use Kafka
  - Migration from Redis Stream completed

**Clearnet Integration:**

- Blockchain transactions are handled asynchronously
- Deposit and withdrawal events are published to Kafka
- Status updates are propagated through event streams

### Pagination Defaults

All paginated endpoints follow consistent defaults:


| Parameter   | Default | Maximum |
| ----------- | ------- | ------- |
| `page`      | 1       | N/A     |
| `page_size` | 50      | 100     |
| `limit`     | 50      | 100     |
| `offset`    | 0       | N/A     |


**Note:** Transfer endpoints use `limit`/`offset` pagination, while other endpoints use `page`/`page_size`.

### Service Ports Reference

For local development:


| Service        | HTTP Port | WebSocket Port |
| -------------- | --------- | -------------- |
| Auth Service   | 8081      | N/A            |
| Trading API    | 8086      | 8086           |
| Quote Service  | 8084      | N/A            |
| Ledger Service | 8083      | N/A            |


### Known Limitations

1. **Clearnet Session Management:** App session creation requires blockchain interaction with associated latency
