# Spot Trading API

The spot trading API provides endpoints for managing spot trading accounts, placing and canceling orders, and retrieving trade history.

## GET /spot/exchangeInfo

Retrieve exchange information including available spot markets and their details.

{% hint style="info" %}
**Authentication:** Not required
{% endhint %}

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

| Field                             | Type    | Description                                |
| --------------------------------- | ------- | ------------------------------------------ |
| `timezone`                        | string  | Exchange timezone                          |
| `server_time`                     | integer | Current server timestamp in milliseconds   |
| `symbols`                         | array   | Array of available spot trading symbols    |
| `symbols[].symbol`                | string  | Trading symbol name (e.g., "BTCYTEST.USD") |
| `symbols[].status`                | string  | Market status (e.g., "active")             |
| `symbols[].base_asset`            | string  | Base asset symbol (e.g., "BTC")            |
| `symbols[].base_asset_precision`  | integer | Decimal precision for base asset           |
| `symbols[].quote_asset`           | string  | Quote asset symbol (e.g., "YTEST.USD")     |
| `symbols[].quote_asset_precision` | integer | Decimal precision for quote asset          |

**Status Codes**:

* `200` - Exchange information retrieved successfully
* `500` - Internal server error

***

## GET /spot/networks

Retrieve supported blockchain networks and the tokens available for deposit and withdrawal on each network.

{% hint style="info" %}
**Authentication:** Not required
{% endhint %}

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

| Field                                  | Type    | Description                                                              |
| -------------------------------------- | ------- | ------------------------------------------------------------------------ |
| `networks`                             | array   | Array of supported blockchain networks                                   |
| `networks[].chain_id`                  | integer | EVM chain ID (e.g., 1 for Ethereum mainnet, 42161 for Arbitrum)          |
| `networks[].name`                      | string  | Human-readable network name                                              |
| `networks[].custody_address`           | string  | Custody contract address on this network (deposit destination)           |
| `networks[].tokens`                    | array   | Array of tokens available on this network                                |
| `networks[].tokens[].symbol`           | string  | Asset symbol (e.g., "ETH", "USDC")                                       |
| `networks[].tokens[].contract_address` | string  | Token ERC-20 contract address (`0x000...000` for native ETH)             |
| `networks[].tokens[].decimals`         | integer | On-chain token decimals                                                  |
| `networks[].tokens[].can_deposit`      | boolean | Whether deposits are currently enabled for this token on this network    |
| `networks[].tokens[].can_withdraw`     | boolean | Whether withdrawals are currently enabled for this token on this network |

**Status Codes**:

* `200` - Networks retrieved successfully

***

## GET /spot/accounts

Retrieve all spot accounts for the authenticated user.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

**Response**:

```json
[
  {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "app_session_id": "your-session-id",
    "owner": "0x1234567890abcdef1234567890abcdef12345678",
    "balances": [
      {
        "asset_symbol": "BTC",
        "asset_name": "Bitcoin",
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
        "asset_name": "Tether USD",
        "total_balance": "10000.00000000",
        "available_balance": "8500.00000000",
        "locked_balance": "1500.00000000",
        "total_balance_usd": "10000.00000000",
        "available_balance_usd": "8500.00000000",
        "locked_balance_usd": "1500.00000000",
        "last_updated": "2023-12-07T10:30:00.000000Z"
      }
    ],
    "usdt_account_value": "179343.931544",
    "state": "active",
    "opened_at": "2023-12-01T08:00:00.000000Z"
  }
]
```

**Response Fields**:

| Field                              | Type           | Description                                                        |
| ---------------------------------- | -------------- | ------------------------------------------------------------------ |
| `id`                               | string         | Unique spot account identifier (UUID)                              |
| `app_session_id`                   | string         | Application session identifier                                     |
| `owner`                            | string         | Owner's Ethereum wallet address                                    |
| `balances`                         | array          | Array of balance objects                                           |
| `balances[].asset_symbol`          | string         | Asset symbol (e.g., "BTC", "USDT")                                 |
| `balances[].asset_name`            | string         | Human-readable asset display name (e.g., "Bitcoin")                |
| `balances[].total_balance`         | decimal string | Total balance (available + locked)                                 |
| `balances[].available_balance`     | decimal string | Balance available for trading                                      |
| `balances[].locked_balance`        | decimal string | Balance locked in open orders                                      |
| `balances[].total_balance_usd`     | decimal string | Total balance valued in stablecoin (USD/USDT); `"0"` when no price |
| `balances[].available_balance_usd` | decimal string | Available balance valued in stablecoin; `"0"` when no price        |
| `balances[].locked_balance_usd`    | decimal string | Locked balance valued in stablecoin; `"0"` when no price           |
| `balances[].last_updated`          | string         | Last balance update timestamp                                      |
| `usdt_account_value`               | decimal string | Total account value in USDT                                        |
| `state`                            | string         | Account state (`active`, `closed`)                                 |
| `opened_at`                        | string         | Account creation timestamp                                         |

**Status Codes**:

* `200` - Accounts retrieved successfully
* `401` - Authentication failed
* `500` - Internal server error

## GET /spot/account

Retrieve information for a specific spot account.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

**Query Parameters**:

| Parameter        | Type   | Required | Options | Description                                                   |
| ---------------- | ------ | -------- | ------- | ------------------------------------------------------------- |
| `app_session_id` | string | Yes      | —       | Spot account app session ID                                   |
| `asset`          | string | No       | —       | Exact asset symbol match on `asset_symbol`                    |
| `asset_like`     | string | No       | —       | Case-insensitive substring (contains) match on `asset_symbol` |

**Request**:

```http
GET /spot/account?app_session_id=your-session-id
```

**Response**:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "app_session_id": "your-session-id",
  "owner": "0x1234567890abcdef1234567890abcdef12345678",
  "balances": [
    {
      "asset_symbol": "BTC",
      "asset_name": "Bitcoin",
      "total_balance": "1.50000000",
      "available_balance": "1.20000000",
      "locked_balance": "0.30000000",
      "total_balance_usd": "169343.931544",
      "available_balance_usd": "135475.145235",
      "locked_balance_usd": "33868.786309",
      "last_updated": "2023-12-07T10:30:00.000000Z"
    }
  ],
  "usdt_account_value": "169343.931544",
  "state": "active",
  "opened_at": "2023-12-01T08:00:00.000000Z"
}
```

**Status Codes**:

* `200` - Account retrieved successfully
* `400` - Missing app\_session\_id parameter
* `401` - Authentication failed
* `404` - Account not found
* `500` - Internal server error

## GET /spot/account/market-fee-rate

Retrieve the effective spot maker/taker fee rates for a single market on a given account.

{% hint style="info" %}
**Authentication:** Required (read:spot scope)
{% endhint %}

**Query Parameters**:

| Parameter        | Type   | Required | Options | Description                      |
| ---------------- | ------ | -------- | ------- | -------------------------------- |
| `app_session_id` | string | Yes      | —       | Spot account app session ID      |
| `market`         | string | Yes      | —       | Trading market (e.g., "BTCUSDT") |

**Request**:

```http
GET /spot/account/market-fee-rate?app_session_id=your-session-id&market=BTCUSDT
```

**Response**:

```json
[
  {
    "app_session_id": "your-session-id",
    "market": "BTCUSDT",
    "maker_fee_rate": "0.001",
    "taker_fee_rate": "0.0015",
    "source": "fee_tier",
    "fee_tier_version": 3
  }
]
```

**Response Fields**:

The response is a JSON array containing a single fee-rate object:

| Field              | Type           | Description                                       |
| ------------------ | -------------- | ------------------------------------------------- |
| `app_session_id`   | string         | Spot account app session ID                       |
| `market`           | string         | Trading market                                    |
| `maker_fee_rate`   | decimal string | Effective maker fee rate                          |
| `taker_fee_rate`   | decimal string | Effective taker fee rate                          |
| `source`           | string         | Origin of the applied fee rate (e.g., `fee_tier`) |
| `fee_tier_version` | integer        | Version of the fee tier configuration applied     |

**Status Codes**:

* `200` - Fee rates retrieved successfully
* `400` - Missing `app_session_id` or `market` parameter
* `401` - Authentication failed
* `500` - Internal server error

## POST /spot/order

Create a new spot trading order.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

**Request Body**:

```json
{
  "app_session_id": "your-session-id",
  "market": "BTCUSDT",
  "side": "buy",
  "type": "limit",
  "amount": "0.5",
  "price": "35000",
  "time_in_force": "gtc",
  "trigger_price": "34000"
}
```

**Request Parameters**:

| Parameter        | Type   | Required | Options                                                                                                                                         | Description                                                            |
| ---------------- | ------ | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `app_session_id` | string | Yes      | —                                                                                                                                               | Spot account app session ID                                            |
| `market`         | string | Yes      | —                                                                                                                                               | Trading market (e.g., "BTCUSDT", "ETHUSDC")                            |
| `side`           | string | Yes      | `buy`, `sell`                                                                                                                                   | Order side                                                             |
| `type`           | string | Yes      | `limit`, `market`                                                                                                                               | Order type                                                             |
| `amount`         | string | Yes      | —                                                                                                                                               | Order amount in base asset (decimal format)                            |
| `price`          | string | No       | —                                                                                                                                               | Limit price (required for `limit` orders, ignored for `market` orders) |
| `time_in_force`  | string | No       | `gtc` (Good-Till-Cancelled), `ioc` (Immediate-Or-Cancel), `fok` (Fill-Or-Kill); defaults to `GTC` for `limit` orders, `IOC` for `market` orders | Time in force                                                          |
| `trigger_price`  | string | No       | —                                                                                                                                               | Trigger price for stop/trigger order types (decimal string)            |

**Response**:

```json
{
  "order_uuid": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Response Fields**:

| Field        | Type   | Description               |
| ------------ | ------ | ------------------------- |
| `order_uuid` | string | UUID of the created order |

**Status Codes**:

* `200` - Order created successfully
* `400` - Invalid request parameters or validation failed (for example, insufficient balance)
* `401` - Authentication failed
* `500` - Internal server error

**Error response** (typical for `400`):

```json
{
  "error": "insufficient_balance",
  "message": "insufficient available balance"
}
```

* `error`: Machine-readable error code (`snake_case`), e.g. `insufficient_balance`.
* `message`: Human-readable detail.

## DELETE /spot/order

Cancel an existing spot order.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

**Request Body**:

```json
{
  "app_session_id": "your-session-id",
  "market": "BTCUSDT",
  "order_uuid": "550e8400-e29b-41d4-a716-446655440000",
  "type": "limit"
}
```

**Request Parameters**:

| Parameter        | Type   | Required | Options | Description                 |
| ---------------- | ------ | -------- | ------- | --------------------------- |
| `app_session_id` | string | Yes      | —       | Spot account app session ID |
| `market`         | string | Yes      | —       | Trading market              |
| `order_uuid`     | string | Yes      | —       | UUID of the order to cancel |
| `type`           | string | Yes      | —       | Order type                  |

**Response**:

```json
{
  "message": "Spot order cancellation request sent successfully"
}
```

**Status Codes**:

* `200` - Cancellation request sent successfully
* `400` - Invalid request or missing parameters
* `401` - Authentication failed
* `500` - Internal server error

## DELETE /spot/orders

Cancel all open spot orders for an account, optionally limited to a single market.

{% hint style="info" %}
**Authentication:** Required (trade:spot scope)
{% endhint %}

**Request Body**:

```json
{
  "app_session_id": "your-session-id",
  "market": "BTCUSDT"
}
```

**Request Parameters**:

| Parameter        | Type   | Required | Options | Description                                                                                                                                |
| ---------------- | ------ | -------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `app_session_id` | string | Yes      | —       | Spot account app session ID                                                                                                                |
| `market`         | string | No       | —       | Trading market. When provided, only open orders for that market are cancelled; when omitted, open orders across all markets are cancelled. |

**Response**:

```json
{
  "message": "Cancel requests sent for 3 orders in market BTCUSDT",
  "canceled_count": 3,
  "failed_count": 0
}
```

When there are no open orders to cancel:

```json
{
  "message": "No open orders to cancel",
  "canceled_count": 0
}
```

**Response Fields**:

| Field            | Type    | Description                                                                                            |
| ---------------- | ------- | ------------------------------------------------------------------------------------------------------ |
| `message`        | string  | Human-readable summary of the cancellation request                                                     |
| `canceled_count` | integer | Number of orders for which a cancellation request was sent                                             |
| `failed_count`   | integer | Number of orders whose cancellation request could not be sent (omitted when there were no open orders) |

**Status Codes**:

* `200` - Cancellation requests sent (or no open orders to cancel)
* `400` - Invalid request body or missing `app_session_id`
* `401` - Authentication failed
* `403` - Not authorized to cancel orders for this account
* `404` - Spot account not found
* `500` - Internal server error

## GET /spot/open\_orders

Retrieve open spot orders with cursor pagination.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

{% hint style="info" %}
**Pagination:** Spot list endpoints use cursor-based pagination. See [Pagination](/broken/pages/b93db56a4d5e4b2656bac7bec7c7292840b07205#pagination) for the shared model.
{% endhint %}

**Query Parameters**:

| Parameter        | Type    | Required | Options                 | Description                                                                                   |
| ---------------- | ------- | -------- | ----------------------- | --------------------------------------------------------------------------------------------- |
| `app_session_id` | string  | Yes      | —                       | Spot account app session ID                                                                   |
| `asset`          | string  | No       | —                       | Case-insensitive fuzzy filter on `asset_symbol` (contains match)                              |
| `market`         | string  | No       | —                       | Filter by trading market                                                                      |
| `use_cursor`     | boolean | No       | —                       | Set `true` to use cursor pagination (use on the first request).                               |
| `cursor`         | string  | No       | —                       | Opaque continuation token from the previous response's `next_cursor`. Omit on the first page. |
| `page_size`      | integer | No       | default `50`, max `100` | Number of orders per page                                                                     |

**Request** (first page):

```http
GET /spot/open_orders?app_session_id=your-session-id&market=BTCUSDT&use_cursor=true&page_size=20
```

**Request** (next page):

```http
GET /spot/open_orders?app_session_id=your-session-id&market=BTCUSDT&cursor=eyJpZCI6MTIzNH0&page_size=20
```

**Response**:

```json
{
  "orders": [
    {
      "id": "1234",
      "order_id": "550e8400-e29b-41d4-a716-446655440000",
      "channel_id": "your-session-id",
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
  "next_cursor": "eyJpZCI6MTIzNH0",
  "has_more": true,
  "page_size": 20
}
```

**Response Fields**:

| Field                    | Type           | Description                                                           |
| ------------------------ | -------------- | --------------------------------------------------------------------- |
| `orders`                 | array          | Array of order objects                                                |
| `orders[].id`            | string         | Internal order record ID                                              |
| `orders[].order_id`      | string         | Order UUID                                                            |
| `orders[].channel_id`    | string         | Spot account app session ID                                           |
| `orders[].market`        | string         | Trading market                                                        |
| `orders[].price`         | decimal string | Order price                                                           |
| `orders[].amount`        | decimal string | Current order amount (may be partially filled)                        |
| `orders[].origin_amount` | decimal string | Original order amount                                                 |
| `orders[].notional`      | decimal string | Notional value (amount x price)                                       |
| `orders[].side`          | string         | Order side (`buy` or `sell`)                                          |
| `orders[].type`          | string         | Order type (`limit` or `market`)                                      |
| `orders[].state`         | string         | Order state (`wait`, `done`, `canceled`)                              |
| `orders[].event`         | string         | Last order event (empty string)                                       |
| `orders[].reason`        | string         | Reason for last state change (empty string)                           |
| `orders[].created_at`    | string         | Order creation timestamp                                              |
| `orders[].updated_at`    | string         | Last update timestamp                                                 |
| `orders[].completed_at`  | string         | Completion timestamp (empty for open orders)                          |
| `next_cursor`            | string         | Token to pass as `cursor` for the next page; empty when no more pages |
| `has_more`               | boolean        | Whether another page exists                                           |
| `page_size`              | integer        | Number of orders per page                                             |

**Status Codes**:

* `200` - Orders retrieved successfully
* `400` - Invalid query parameters (including malformed `cursor`)
* `401` - Authentication failed
* `500` - Internal server error

## GET /spot/orders

Retrieve spot order history with cursor pagination.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

{% hint style="info" %}
**Pagination:** This endpoint uses cursor-based pagination. See [Pagination](/broken/pages/b93db56a4d5e4b2656bac7bec7c7292840b07205#pagination).
{% endhint %}

**Query Parameters**:

| Parameter        | Type    | Required | Options                 | Description                                                                                   |
| ---------------- | ------- | -------- | ----------------------- | --------------------------------------------------------------------------------------------- |
| `app_session_id` | string  | Yes      | —                       | Spot account app session ID                                                                   |
| `market`         | string  | No       | —                       | Filter by trading market                                                                      |
| `use_cursor`     | boolean | No       | —                       | Set `true` to use cursor pagination (use on the first request).                               |
| `cursor`         | string  | No       | —                       | Opaque continuation token from the previous response's `next_cursor`. Omit on the first page. |
| `page_size`      | integer | No       | default `50`, max `100` | Number of orders per page                                                                     |

**Request** (first page):

```http
GET /spot/orders?app_session_id=your-session-id&use_cursor=true&page_size=50
```

**Request** (next page):

```http
GET /spot/orders?app_session_id=your-session-id&cursor=eyJpZCI6OTl9&page_size=50
```

**Response**: Same format as `/spot/open_orders` but includes all orders (open, filled, and cancelled)

**Status Codes**:

* `200` - Orders retrieved successfully
* `400` - Invalid query parameters (including malformed `cursor`)
* `401` - Authentication failed
* `500` - Internal server error

## GET /spot/trades

Retrieve spot trade history with cursor pagination.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

{% hint style="info" %}
**Pagination:** This endpoint uses cursor-based pagination. See [Pagination](/broken/pages/b93db56a4d5e4b2656bac7bec7c7292840b07205#pagination).
{% endhint %}

**Query Parameters**:

| Parameter        | Type    | Required | Options                 | Description                                                                                   |
| ---------------- | ------- | -------- | ----------------------- | --------------------------------------------------------------------------------------------- |
| `app_session_id` | string  | Yes      | —                       | Spot account app session ID                                                                   |
| `market`         | string  | No       | —                       | Filter by trading market                                                                      |
| `use_cursor`     | boolean | No       | —                       | Set `true` to use cursor pagination (use on the first request).                               |
| `cursor`         | string  | No       | —                       | Opaque continuation token from the previous response's `next_cursor`. Omit on the first page. |
| `page_size`      | integer | No       | default `50`, max `100` | Number of trades per page                                                                     |

**Request** (first page):

```http
GET /spot/trades?app_session_id=your-session-id&market=BTCUSDT&use_cursor=true&page_size=50
```

**Request** (next page):

```http
GET /spot/trades?app_session_id=your-session-id&market=BTCUSDT&cursor=eyJpZCI6NTcyMTA5Mn0&page_size=50
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
  "next_cursor": "eyJpZCI6NTcyMTA5Mn0",
  "has_more": true,
  "page_size": 50
}
```

**Response Fields**:

| Field                  | Type           | Description                                                           |
| ---------------------- | -------------- | --------------------------------------------------------------------- |
| `trades`               | array          | Array of trade objects                                                |
| `trades[].id`          | string         | Internal trade record ID                                              |
| `trades[].order_id`    | string         | User's own order ID in this trade (numeric)                           |
| `trades[].market`      | string         | Trading market                                                        |
| `trades[].amount`      | decimal string | Trade amount in base asset                                            |
| `trades[].price`       | decimal string | Trade execution price                                                 |
| `trades[].is_buyer`    | boolean        | Whether the user was the buyer in this trade                          |
| `trades[].is_maker`    | boolean        | Whether the user was the maker (liquidity provider) in this trade     |
| `trades[].executed_at` | string         | Trade execution timestamp                                             |
| `trades[].created_at`  | string         | Trade record creation timestamp                                       |
| `next_cursor`          | string         | Token to pass as `cursor` for the next page; empty when no more pages |
| `has_more`             | boolean        | Whether another page exists                                           |
| `page_size`            | integer        | Number of trades per page                                             |

**Note**: The API response only exposes the user's own order ID and does not expose counterparty information for privacy reasons. It provides user-relative flags (`is_buyer`, `is_maker`) to indicate the user's role in the trade.

**Status Codes**:

* `200` - Trades retrieved successfully
* `400` - Invalid query parameters (including malformed `cursor`)
* `401` - Authentication failed
* `500` - Internal server error

## GET /spot/deposits

Retrieve spot deposit history with cursor pagination.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

{% hint style="info" %}
**Pagination:** This endpoint uses cursor-based pagination. See [Pagination](/broken/pages/b93db56a4d5e4b2656bac7bec7c7292840b07205#pagination).
{% endhint %}

**Query Parameters**:

| Parameter        | Type    | Required | Options                 | Description                                                                                   |
| ---------------- | ------- | -------- | ----------------------- | --------------------------------------------------------------------------------------------- |
| `app_session_id` | string  | Yes      | —                       | Spot account app session ID                                                                   |
| `use_cursor`     | boolean | No       | —                       | Set `true` to use cursor pagination (use on the first request).                               |
| `cursor`         | string  | No       | —                       | Opaque continuation token from the previous response's `next_cursor`. Omit on the first page. |
| `page_size`      | integer | No       | default `50`, max `100` | Number of deposits per page                                                                   |

**Request** (first page):

```http
GET /spot/deposits?app_session_id=your-session-id&use_cursor=true&page_size=50
```

**Request** (next page):

```http
GET /spot/deposits?app_session_id=your-session-id&cursor=eyJpZCI6Nn0&page_size=50
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
  "next_cursor": "eyJpZCI6Nn0",
  "has_more": true,
  "page_size": 50
}
```

**Response Fields**:

| Field                         | Type           | Description                                                                                   |
| ----------------------------- | -------------- | --------------------------------------------------------------------------------------------- |
| `deposits`                    | array          | Array of deposit objects                                                                      |
| `deposits[].spot_account_id`  | string         | Spot account ID (UUID)                                                                        |
| `deposits[].asset_symbol`     | string         | Asset symbol (e.g., "USDT", "BTC")                                                            |
| `deposits[].amount`           | decimal string | Deposit amount                                                                                |
| `deposits[].transaction_hash` | string         | On-chain transaction hash                                                                     |
| `deposits[].created_at`       | string         | Record creation timestamp                                                                     |
| `deposits[].deposited_at`     | string         | Timestamp when deposit occurred                                                               |
| `deposits[].chain_id`         | integer        | Blockchain chain ID where the deposit was received (e.g., 1 for Ethereum, 42161 for Arbitrum) |
| `next_cursor`                 | string         | Token to pass as `cursor` for the next page; empty when no more pages                         |
| `has_more`                    | boolean        | Whether another page exists                                                                   |
| `page_size`                   | integer        | Number of deposits per page                                                                   |

**Status Codes**:

* `200` - Deposits retrieved successfully
* `400` - Missing app\_session\_id parameter
* `401` - Authentication failed
* `500` - Internal server error

## GET /spot/withdrawals

Retrieve spot withdrawal history with cursor pagination.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

{% hint style="info" %}
**Pagination:** This endpoint uses cursor-based pagination. See [Pagination](/broken/pages/b93db56a4d5e4b2656bac7bec7c7292840b07205#pagination).
{% endhint %}

**Query Parameters**:

| Parameter        | Type    | Required | Options                 | Description                                                                                   |
| ---------------- | ------- | -------- | ----------------------- | --------------------------------------------------------------------------------------------- |
| `app_session_id` | string  | Yes      | —                       | Spot account app session ID                                                                   |
| `use_cursor`     | boolean | No       | —                       | Set `true` to use cursor pagination (use on the first request).                               |
| `cursor`         | string  | No       | —                       | Opaque continuation token from the previous response's `next_cursor`. Omit on the first page. |
| `page_size`      | integer | No       | default `50`, max `100` | Number of withdrawals per page                                                                |

**Request** (first page):

```http
GET /spot/withdrawals?app_session_id=your-session-id&use_cursor=true&page_size=50
```

**Request** (next page):

```http
GET /spot/withdrawals?app_session_id=your-session-id&cursor=eyJpZCI6MTB9&page_size=50
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
  "next_cursor": "eyJpZCI6MTB9",
  "has_more": true,
  "page_size": 50
}
```

**Response Fields**:

| Field                            | Type           | Description                                                                                      |
| -------------------------------- | -------------- | ------------------------------------------------------------------------------------------------ |
| `withdrawals`                    | array          | Array of withdrawal objects                                                                      |
| `withdrawals[].id`               | string         | Withdrawal ID (UUID)                                                                             |
| `withdrawals[].spot_account_id`  | string         | Spot account ID (UUID)                                                                           |
| `withdrawals[].asset_symbol`     | string         | Asset symbol (e.g., "USDT", "BTC")                                                               |
| `withdrawals[].amount`           | decimal string | Withdrawal amount                                                                                |
| `withdrawals[].status`           | string         | Withdrawal status (`pending`, `completed`, `failed`)                                             |
| `withdrawals[].transaction_hash` | string         | On-chain transaction hash (when completed)                                                       |
| `withdrawals[].failure_reason`   | string         | Reason for failure (when status is `failed`)                                                     |
| `withdrawals[].requested_at`     | string         | Timestamp when withdrawal was requested                                                          |
| `withdrawals[].completed_at`     | string         | Timestamp when withdrawal completed (or failed)                                                  |
| `withdrawals[].created_at`       | string         | Record creation timestamp                                                                        |
| `withdrawals[].chain_id`         | integer        | Blockchain chain ID where the withdrawal was executed (e.g., 1 for Ethereum, 42161 for Arbitrum) |
| `next_cursor`                    | string         | Token to pass as `cursor` for the next page; empty when no more pages                            |
| `has_more`                       | boolean        | Whether another page exists                                                                      |
| `page_size`                      | integer        | Number of withdrawals per page                                                                   |

**Status Codes**:

* `200` - Withdrawals retrieved successfully
* `400` - Missing app\_session\_id parameter
* `401` - Authentication failed
* `500` - Internal server error

## POST /spot/withdrawal

Request a new spot withdrawal.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

{% hint style="warning" %}
**Known Issue**: Server returns `500` instead of `400` when withdrawal amount exceeds available balance.
{% endhint %}

**Request Body**:

```json
{
  "app_session_id": "your-session-id",
  "asset_symbol": "USDT",
  "amount": "5000.00000000",
  "chain_id": 1
}
```

**Request Parameters**:

| Parameter        | Type    | Required | Options                                  | Description                                                                                                                                                                                                                                                              |
| ---------------- | ------- | -------- | ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `app_session_id` | string  | Yes      | —                                        | Spot account app session ID                                                                                                                                                                                                                                              |
| `asset_symbol`   | string  | Yes      | —                                        | Asset to withdraw (e.g., "USDT", "BTC")                                                                                                                                                                                                                                  |
| `amount`         | string  | Yes      | must be positive                         | Amount to withdraw (decimal format)                                                                                                                                                                                                                                      |
| `chain_id`       | integer | No       | default: server-configured default chain | Target blockchain chain ID for the withdrawal (e.g., 1 for Ethereum, 42161 for Arbitrum). Use `GET /spot/networks` to discover available chains. When omitted, falls back to the server-configured default chain. Returns `400` if omitted and no default is configured. |

**Response**:

```json
{
  "withdrawal_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "pending",
  "message": "Withdrawal request accepted and funds reserved"
}
```

**Response Fields**:

| Field           | Type   | Description                    |
| --------------- | ------ | ------------------------------ |
| `withdrawal_id` | string | UUID of the withdrawal request |
| `status`        | string | Initial status (`pending`)     |
| `message`       | string | Confirmation message           |

**Status Codes**:

* `200` - Withdrawal request accepted
* `400` - Invalid request or insufficient funds
* `401` - Authentication failed
* `500` - Internal server error

***
