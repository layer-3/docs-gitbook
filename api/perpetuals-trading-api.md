# Perpetuals Trading API

The perpetuals trading API provides endpoints for managing cross-margin perpetual futures accounts, placing and canceling orders, monitoring positions, and retrieving trade history. All perpetuals accounts use cross-margin model where collateral is shared across all positions.

## GET /perpetual/exchangeInfo

Retrieve exchange information for perpetuals markets including available markets and their details.

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
      "symbol": "BTCYTEST.USD-PERP",
      "status": "active",
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

| Field                                | Type           | Description                                                                                                                                                   |
| ------------------------------------ | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `timezone`                           | string         | Exchange timezone                                                                                                                                             |
| `server_time`                        | integer        | Current server timestamp in milliseconds                                                                                                                      |
| `symbols`                            | array          | Array of available perpetuals markets                                                                                                                         |
| `symbols[].symbol`                   | string         | Trading symbol name (e.g., "BTCYTEST.USD-PERP")                                                                                                               |
| `symbols[].status`                   | string         | Market status (e.g., "active")                                                                                                                                |
| `symbols[].base_asset_name`          | string         | Display name for base asset                                                                                                                                   |
| `symbols[].base_asset`               | string         | Base asset symbol (e.g., "BTC")                                                                                                                               |
| `symbols[].quote_asset_name`         | string         | Display name for quote asset                                                                                                                                  |
| `symbols[].quote_asset`              | string         | Quote asset symbol (e.g., "USD")                                                                                                                              |
| `symbols[].amount_precision`         | integer        | Decimal precision for amount (base asset)                                                                                                                     |
| `symbols[].price_precision`          | integer        | Decimal precision for price (quote asset)                                                                                                                     |
| `symbols[].amount_display_precision` | integer        | Display precision for amount in UI                                                                                                                            |
| `symbols[].price_display_precision`  | integer        | Display precision for price in UI                                                                                                                             |
| `symbols[].maker_fee_rate`           | decimal string | Maker fee rate (e.g., "0.001")                                                                                                                                |
| `symbols[].taker_fee_rate`           | decimal string | Taker fee rate (e.g., "0.002")                                                                                                                                |
| `symbols[].max_allowed_leverage`     | decimal string | Max allowed leverage (e.g., "100")                                                                                                                            |
| `symbols[].maintenance_margin_rate`  | decimal string | Maintenance margin rate (MMR) for this market (e.g., `"0.007"` = 0.7%). Tiered maintenance on positions may still compute `maintenance_margin` from brackets. |
| `symbols[].filters`                  | array          | Trading rules (PRICE\_FILTER, LOT\_SIZE, MIN\_NOTIONAL, DEPTH\_MERGE)                                                                                         |

**Status Codes**:

* `200` - Exchange information retrieved successfully
* `500` - Internal server error

***

## Perpetuals Calculation Formulas

For frontend display (margin ratio, break-even price, ROI %, etc.), see `perpetuals-frontend-formulas-from-code.md`. Key formulas:

* **Cross margin ratio**: `margin_ratio = total_account_equity / total_maintenance_margin` (from `GET /perpetual/account` or `GET /perpetual/accounts`)
* **Break-even price**: Long `entry_price - realized_pnl/amount`, Short `entry_price + realized_pnl/amount` (derived, not returned)
* **Liquidation price**: Returned by API and WebSocket `position_update` for cross-margin positions

***

## List pagination

Perpetual **list** endpoints use opaque **cursor** pagination.

{% stepper %}
{% step %}
### First request

Send `use_cursor=true` (optionally with `page_size`). The response returns the first page plus a `next_cursor` token and a `has_more` flag.
{% endstep %}

{% step %}
### Next pages

While `has_more` is `true`, send the previous response's `next_cursor` value as the `cursor` query parameter to fetch the following page.
{% endstep %}

{% step %}
### End

When `has_more` is `false`, `next_cursor` is empty and there are no more pages.
{% endstep %}
{% endstepper %}

**Query parameters** (where noted on each endpoint):

| Parameter    | Type    | Required | Options           | Description                                                                                  |
| ------------ | ------- | -------- | ----------------- | -------------------------------------------------------------------------------------------- |
| `use_cursor` | boolean | No       | `true`            | Set `true` to use cursor pagination (on the first request)                                   |
| `cursor`     | string  | No       | —                 | Opaque continuation token from the previous response's `next_cursor`. Omit on the first page |
| `page_size`  | integer | No       | see each endpoint | Items per page                                                                               |

**Response fields**:

| Field         | Type    | Description                                                           |
| ------------- | ------- | --------------------------------------------------------------------- |
| `next_cursor` | string  | Token to pass as `cursor` for the next page; empty when no more pages |
| `has_more`    | boolean | Whether another page exists                                           |
| `page_size`   | integer | Items per page                                                        |

**Errors**: Malformed `cursor` → `400` with `"error": "invalid_cursor"`.

***

## GET /perpetual/account/market-fee-rate

Retrieve the effective maker/taker fee rate for a single market on a perpetuals app session.

{% hint style="info" %}
**Authentication:** Required (read:futures scope)
{% endhint %}

**Query Parameters**:

| Parameter        | Type   | Required | Options | Description                       |
| ---------------- | ------ | -------- | ------- | --------------------------------- |
| `app_session_id` | string | Yes      | —       | Perpetuals account app session ID |
| `market`         | string | Yes      | —       | Market symbol (e.g., `BTCUSDT`)   |

**Response**:

```json
[
  {
    "app_session_id": "your-session-id",
    "market": "BTCUSDT",
    "maker_fee_rate": "0.0002",
    "taker_fee_rate": "0.0005",
    "source": "fee_tier",
    "fee_tier_version": 3
  }
]
```

**Response Fields**:

The response is a JSON array containing a single object for the requested market:

| Field              | Type           | Description                                             |
| ------------------ | -------------- | ------------------------------------------------------- |
| `app_session_id`   | string         | Application session identifier (from request parameter) |
| `market`           | string         | Market symbol                                           |
| `maker_fee_rate`   | decimal string | Effective maker fee rate                                |
| `taker_fee_rate`   | decimal string | Effective taker fee rate                                |
| `source`           | string         | Origin of the effective fee rate (e.g., `fee_tier`)     |
| `fee_tier_version` | integer        | Version of the fee tier configuration applied           |

**Status Codes**:

* `200` - Account fee rate retrieved successfully
* `400` - Missing `app_session_id` or `market` parameter
* `401` - Authentication failed

***

## GET /perpetual/accounts

Retrieve all perpetuals accounts for the authenticated user.

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
        "uuid": "550e8400-e29b-41d4-a716-446655440000",
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
        "locked_amount": "0.50000000",
        "available_amount": "1.00000000"
      }
    ]
  }
]
```

**Response Fields**:

| Field                              | Type           | Description                                                                                                                                                                                                                                                          |
| ---------------------------------- | -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`                               | string         | Unique perpetuals account identifier (UUID)                                                                                                                                                                                                                          |
| `app_session_id`                   | string         | Application session identifier                                                                                                                                                                                                                                       |
| `owner`                            | string         | Owner's Ethereum wallet address                                                                                                                                                                                                                                      |
| `state`                            | string         | Account state (`active`, `closed`)                                                                                                                                                                                                                                   |
| `opened_at`                        | string         | Account creation timestamp                                                                                                                                                                                                                                           |
| `total_account_balance`            | decimal string | Sum of all collateral balances                                                                                                                                                                                                                                       |
| `total_unrealized_pnl`             | decimal string | Total unrealized PnL across all positions                                                                                                                                                                                                                            |
| `total_account_equity`             | decimal string | Total account equity (balance + unrealized PnL)                                                                                                                                                                                                                      |
| `total_allocated_margin`           | decimal string | Total margin allocated to positions (historical name; same numeric value as `total_locked_balance`)                                                                                                                                                                  |
| `total_locked_balance`             | decimal string | Sum of locked collateral across all assets (orders + position margin); same value as `total_allocated_margin`                                                                                                                                                        |
| `total_maintenance_margin`         | decimal string | Total maintenance margin required                                                                                                                                                                                                                                    |
| `available_balance`                | decimal string | (Account-level.) Balance available for new positions or withdrawals. Includes unrealized PnL (total balance − allocated/locked + unrealized PnL). Not the max you can transfer perp→spot; use `transferable_balances` or `balances[].max_transfer_out` for that cap. |
| `transferable_balances`            | object         | (optional) Map of `asset_symbol` → decimal string: max amount the user may transfer perpetuals → spot for that collateral asset (closing-fee reserve; 80% cap when cross-margin positions exist). Omitted when there are no balance rows.                            |
| `balances`                         | array          | Array of collateral balances                                                                                                                                                                                                                                         |
| `balances[].asset_symbol`          | string         | Collateral asset symbol (e.g., "USDT")                                                                                                                                                                                                                               |
| `balances[].asset_name`            | string         | Display name (when known)                                                                                                                                                                                                                                            |
| `balances[].total_balance`         | decimal string | Total balance for this asset                                                                                                                                                                                                                                         |
| `balances[].allocated_balance`     | decimal string | Margin allocated to positions (historical name; same numeric value as `locked_balance`)                                                                                                                                                                              |
| `balances[].locked_balance`        | decimal string | Locked collateral for this asset (orders + position margin); same value as `allocated_balance`                                                                                                                                                                       |
| `balances[].available_balance`     | decimal string | (Per-asset.) Available for this asset. Includes unrealized PnL (total − locked + unrealized PnL; in single-collateral the PnL is attributed to that asset).                                                                                                          |
| `balances[].max_transfer_out`      | decimal string | Max user perp→spot transfer for this asset (may be lower than `available_balance` when open positions require a closing-fee reserve or cross-margin transfer cap applies).                                                                                           |
| `balances[].total_balance_usd`     | decimal string | Total balance valued in stablecoin (USD/USDT); `"0"` when no price                                                                                                                                                                                                   |
| `balances[].allocated_balance_usd` | decimal string | Allocated/locked margin valued in stablecoin; `"0"` when no price                                                                                                                                                                                                    |
| `balances[].locked_balance_usd`    | decimal string | Same as `allocated_balance_usd` (locked collateral in USD terms)                                                                                                                                                                                                     |
| `balances[].available_balance_usd` | decimal string | Available balance valued in stablecoin; `"0"` when no price                                                                                                                                                                                                          |
| `balances[].last_updated`          | string         | Last balance update timestamp                                                                                                                                                                                                                                        |
| `initial_leverages`                | object         | (optional) Map of market → leverage string (e.g. `"10"`). Only present when the account has per-market settings.                                                                                                                                                     |
| `positions`                        | array          | Array of open positions. May include two rows for the same `market` (one `long`, one `short`), since long and short legs are tracked separately.                                                                                                                     |
| `positions[].uuid`                 | string         | Unique position identifier (always present)                                                                                                                                                                                                                          |
| `positions[].market`               | string         | Trading market (e.g., "BTCYTEST.USD-PERP")                                                                                                                                                                                                                           |
| `positions[].direction`            | string         | Position direction (`long` or `short`)                                                                                                                                                                                                                               |
| `positions[].amount`               | decimal string | Position size (always positive)                                                                                                                                                                                                                                      |
| `positions[].entry_price`          | decimal string | Average entry price                                                                                                                                                                                                                                                  |
| `positions[].mark_price`           | decimal string | Current mark price                                                                                                                                                                                                                                                   |
| `positions[].notional_value`       | decimal string | Current notional value (amount x mark\_price)                                                                                                                                                                                                                        |
| `positions[].leverage`             | decimal string | Leverage multiplier                                                                                                                                                                                                                                                  |
| `positions[].unrealized_pnl`       | decimal string | Unrealized profit/loss                                                                                                                                                                                                                                               |
| `positions[].realized_pnl`         | decimal string | Realized profit/loss from this position                                                                                                                                                                                                                              |
| `positions[].total_pnl`            | decimal string | Total profit/loss (unrealized + realized)                                                                                                                                                                                                                            |
| `positions[].allocated_margin`     | decimal string | Margin allocated to this position                                                                                                                                                                                                                                    |
| `positions[].maintenance_margin`   | decimal string | Maintenance margin requirement                                                                                                                                                                                                                                       |
| `positions[].liquidation_price`    | decimal string | Liquidation price (calculated by backend for cross-margin positions)                                                                                                                                                                                                 |
| `positions[].margin_asset`         | string         | Collateral asset for margin calculation                                                                                                                                                                                                                              |
| `positions[].locked_amount`        | decimal string | Amount currently locked                                                                                                                                                                                                                                              |
| `positions[].available_amount`     | decimal string | Amount available                                                                                                                                                                                                                                                     |

**Status Codes**:

* `200` - Accounts retrieved successfully
* `401` - Authentication failed
* `500` - Internal server error

***

## POST /perpetual/account

Create a new perpetuals account with a specified app\_session\_id.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

> **Account creation completes shortly after this call returns `200`.** The account does not exist immediately; poll `GET /perpetual/account` until it returns a result (typically a few seconds).

**Request Body**:

```json
{
  "app_session_id": "your-session-id",
  "owner": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"
}
```

**Request Parameters**:

| Parameter        | Type   | Required | Options                     | Description                                                                              |
| ---------------- | ------ | -------- | --------------------------- | ---------------------------------------------------------------------------------------- |
| `app_session_id` | string | Yes      | —                           | Application session ID for the new perpetual account (typically same as spot account ID) |
| `owner`          | string | No       | default: authenticated user | Owner wallet address                                                                     |

**Response**:

```json
{
  "app_session_id": "your-session-id",
  "owner": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  "message": "Perpetuals account creation initiated"
}
```

**Response Fields**:

| Field            | Type   | Description                               |
| ---------------- | ------ | ----------------------------------------- |
| `app_session_id` | string | The app session ID of the created account |
| `owner`          | string | Owner wallet address                      |
| `message`        | string | Status message                            |

**Status Codes**:

* `200` - Account creation initiated successfully
* `400` - Invalid request parameters (missing app\_session\_id, invalid owner address)
* `401` - Authentication failed
* `403` - Forbidden (owner mismatch - trying to create account for different wallet)
* `500` - Internal server error

**Notes**:

* The `app_session_id` should typically match the user's spot account ID to allow seamless transfers
* The account becomes available shortly after this request completes
* Attempting to create an account with an existing `app_session_id` will return an error

***

## GET /perpetual/account

Retrieve detailed information for a specific perpetuals account including all balances, positions, and account-level metrics.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

**Query Parameters**:

| Parameter        | Type   | Required | Options | Description                       |
| ---------------- | ------ | -------- | ------- | --------------------------------- |
| `app_session_id` | string | Yes      | —       | Perpetuals account app session ID |

**Request**:

```http
GET /perpetual/account?app_session_id=your-session-id
```

**Response**:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "app_session_id": "your-session-id",
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

**Response Fields**: Same as `/perpetual/accounts` response (single account object), including optional `initial_leverages` and `transferable_balances`.

**Note on `available_balance` vs transfers**: The name appears in two places—(1) at the **account root**: aggregate balance available for new positions; (2) inside each `balances[]` element: available for that asset. **Both include unrealized PnL** (formula: total − allocated/locked + unrealized PnL). For **perp→spot transfer** limits, use `transferable_balances` (account root) or `balances[].max_transfer_out` per asset—these match WebSocket `perpetuals_account.account_update` and account for closing-fee reserve and cross-margin transfer caps.

**Status Codes**:

* `200` - Account retrieved successfully
* `400` - Missing app\_session\_id parameter
* `401` - Authentication failed
* `404` - Account not found
* `500` - Internal server error

***

## POST /perpetual/warning/mute

Mute this perpetuals account's liquidation warnings for 24 hours.

{% hint style="info" %}
**Authentication:** Required (trade:futures scope)
{% endhint %}

**Request Body**:

```json
{
  "app_session_id": "your-session-id"
}
```

**Request Fields**:

| Parameter        | Type   | Required | Options | Description                       |
| ---------------- | ------ | -------- | ------- | --------------------------------- |
| `app_session_id` | string | Yes      | —       | Perpetuals account app session ID |

**Response** (200):

```json
{
  "app_session_id": "your-session-id",
  "muted_until": "2026-06-11T10:30:00Z"
}
```

**Response Fields**:

| Field            | Type   | Description                                                      |
| ---------------- | ------ | ---------------------------------------------------------------- |
| `app_session_id` | string | Application session identifier (from request body)               |
| `muted_until`    | string | RFC3339 UTC timestamp until which liquidation warnings are muted |

**Status Codes**:

* `200` - Liquidation warnings muted successfully
* `400` - Invalid request body or missing `app_session_id`
* `401` - Authentication failed
* `404` - Account not found
* `500` - Internal server error
* `503` - Service temporarily unavailable

***

## GET /perpetual/balance

Retrieve collateral balances for a specific perpetuals account.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

**Query Parameters**:

| Parameter        | Type   | Required | Options | Description                       |
| ---------------- | ------ | -------- | ------- | --------------------------------- |
| `app_session_id` | string | Yes      | —       | Perpetuals account app session ID |

**Request**:

```http
GET /perpetual/balance?app_session_id=your-session-id
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

| Field                   | Type           | Description                                                                                                     |
| ----------------------- | -------------- | --------------------------------------------------------------------------------------------------------------- |
| `asset_symbol`          | string         | Collateral asset symbol                                                                                         |
| `asset_name`            | string         | Display name (when known)                                                                                       |
| `total_balance`         | decimal string | Total balance for this asset                                                                                    |
| `allocated_balance`     | decimal string | Margin allocated to positions (historical name; same numeric value as `locked_balance`)                         |
| `locked_balance`        | decimal string | Locked collateral for this asset (orders + position margin); same value as `allocated_balance`                  |
| `available_balance`     | decimal string | Balance available for new positions (includes unrealized PnL share for this asset)                              |
| `max_transfer_out`      | decimal string | Max user perp→spot transfer for this asset (same as `transferable_balances[asset]` on `GET /perpetual/account`) |
| `total_balance_usd`     | decimal string | Total balance valued in stablecoin (USD/USDT); `"0"` when no price                                              |
| `allocated_balance_usd` | decimal string | Allocated/locked margin valued in stablecoin; `"0"` when no price                                               |
| `locked_balance_usd`    | decimal string | Same as `allocated_balance_usd` (locked collateral in USD terms)                                                |
| `available_balance_usd` | decimal string | Available balance valued in stablecoin; `"0"` when no price                                                     |
| `last_updated`          | string         | Last balance update timestamp                                                                                   |

**Status Codes**:

* `200` - Balances retrieved successfully
* `400` - Missing app\_session\_id parameter
* `401` - Authentication failed
* `404` - Account not found
* `500` - Internal server error

***

## GET /perpetual/transfer-assets

Retrieve the **system-level** perpetual transfer asset configuration.

This endpoint returns which assets are configured for spot/perps transfer at the platform level.\
Only assets that are both **active** and marked as **stablecoin** are returned.\
It does **not** represent user-level transferability (e.g., user balance availability).

{% hint style="info" %}
**Authentication:** Not required (public endpoint)
{% endhint %}

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

| Field       | Type    | Description                                                        |
| ----------- | ------- | ------------------------------------------------------------------ |
| `symbol`    | string  | Asset symbol                                                       |
| `name`      | string  | Display name                                                       |
| `decimals`  | integer | Decimal precision for min transfer amount logic (`10^(-decimals)`) |
| `is_active` | boolean | Whether this asset is currently enabled for transfer               |

Only assets that are both active and stablecoin are included (stablecoin flag is not repeated in the JSON).

**Status Codes**:

* `200` - Transfer assets retrieved successfully
* `500` - Internal server error

***

## GET /perpetual/transaction/history

Retrieve unified perpetual transaction history for a wallet + app session.

This endpoint aggregates balance-changing events (funding payments, internal transfers, fees, realized PnL, liquidations, ADL, etc.) into a single paginated history from the **user's perspective**. Each row is one `(transaction_id, asset)` pair:

* Positive `amount` → balance increase
* Negative `amount` → balance decrease

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

**Query Parameters**:

| Parameter        | Type    | Required | Options                                                                      | Description                                                                                                            |
| ---------------- | ------- | -------- | ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `app_session_id` | string  | Yes      | —                                                                            | Perpetuals account app session ID                                                                                      |
| `type`           | string  | No       | `""`, `funding_fee`, `transfer`, `fee`, `realized_pnl`, `liquidation`, `adl` | High-level transaction type filter (see mapping below)                                                                 |
| `market`         | string  | No       | —                                                                            | Market filter (e.g. `BTC-USDT-PERP`)                                                                                   |
| `asset`          | string  | No       | —                                                                            | Asset symbol to filter by (e.g. margin asset symbol)                                                                   |
| `start_time`     | integer | No       | Unix timestamp in seconds (UTC)                                              | Inclusive range start. See Time range below.                                                                           |
| `end_time`       | integer | No       | Unix timestamp in seconds (UTC)                                              | Inclusive range end. See Time range below.                                                                             |
| `use_cursor`     | boolean | No       | —                                                                            | Set `true` to use cursor pagination (first request). See [List pagination](perpetuals-trading-api.md#list-pagination). |
| `cursor`         | string  | No       | —                                                                            | Opaque continuation token from the previous response's `next_cursor`. Omit on the first page.                          |
| `page_size`      | integer | No       | default `50`, max `100`                                                      | Number of items per page                                                                                               |

`type` filter mapping:

* `""` (empty / omitted): User-facing types only (excludes internal order bookkeeping such as `ORDER_*`, `DEPOSIT`, `WITHDRAWAL`, etc.)
* `"funding_fee"`: Funding payments (maps to `FUNDING_FEE_IN`, `FUNDING_FEE_OUT`)
* `"transfer"`: Internal asset transfers (maps to `TRANSFER_IN`, `TRANSFER_OUT`)
* `"fee"`: Trading fee-related events (maps to `FEE`, `FEE_WAIVER_DISTRESSED_CLOSE`, `POSITION_OPEN_FEE`, `POSITION_CLOSE_FEE`)
* `"realized_pnl"`: Realized PnL events (maps to `REALIZED_PNL`)
* `"liquidation"`: Liquidation events (maps to `LIQUIDATION`, `LIQUIDATION_COMPLETE`)
* `"adl"`: ADL events (maps to `ADL_COUNTERPARTY_CLOSE`, `ADL_TAKEN_OVER_CLOSE`)
* Compatibility note: ADL events are intentionally separated from the `liquidation` filter; clients that want all forced-deleveraging events should query both `type=liquidation` and `type=adl`.

**Time range**:

`start_time` and `end_time` are applied unchanged. When bounds are omitted (or sent as `0`), the backend applies a default window of the **last 30 days** ending at now (UTC):

| Client sends                        | Effective window                                                                          |
| ----------------------------------- | ----------------------------------------------------------------------------------------- |
| Neither `start_time` nor `end_time` | `[now − 30d, now]`                                                                        |
| `start_time` only                   | `[start_time, now]`                                                                       |
| `end_time` only                     | `[end_time − 30d, end_time]`                                                              |
| Both                                | `[start_time, end_time]` (if `start_time > end_time`, start is reset to `end_time − 30d`) |

The time window is applied on the transaction time of each event.

**Request** (explicit time range):

```http
GET /perpetual/transaction/history?app_session_id=your-session-id&type=funding_fee&market=BTC-USDT-PERP&asset=USDT&start_time=1704067200&end_time=1704153600&page_size=20
```

**Request** (default last 30 days — omit `start_time` and `end_time`; cursor first page):

```http
GET /perpetual/transaction/history?app_session_id=your-session-id&use_cursor=true&page_size=20
```

**Request** (cursor page 2):

```http
GET /perpetual/transaction/history?app_session_id=your-session-id&use_cursor=true&cursor=1701943800000:5721092:USDT&page_size=20
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
  "has_more": false,
  "next_cursor": "",
  "page_size": 20
}
```

**Pagination**: Cursor only. Send `use_cursor=true` on the first request, then follow `next_cursor` while `has_more` is `true` (see [List pagination](perpetuals-trading-api.md#list-pagination)). Malformed `cursor` → `400` / `"error": "invalid_cursor"`.

**Response Fields**:

| Field                      | Type           | Description                                                                                                                         |
| -------------------------- | -------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `items`                    | array          | Array of transaction history records                                                                                                |
| `items[].transaction_id`   | integer        | Internal transaction identifier                                                                                                     |
| `items[].transaction_time` | string         | Transaction timestamp in RFC3339 format                                                                                             |
| `items[].transaction_type` | string         | Low-level transaction type (e.g., `FUNDING_FEE_IN`, `TRANSFER_OUT`, `FEE`, `REALIZED_PNL`, `LIQUIDATION`, `ADL_COUNTERPARTY_CLOSE`) |
| `items[].market`           | string         | Related market symbol when applicable; empty string for non-market-specific events (e.g. some transfers)                            |
| `items[].asset`            | string         | Asset symbol impacted by the transaction                                                                                            |
| `items[].amount`           | decimal string | Signed amount from the user's perspective (positive = balance increase, negative = balance decrease)                                |
| `has_more`                 | boolean        | Whether another page exists                                                                                                         |
| `next_cursor`              | string         | Opaque next-page token (empty when no more pages)                                                                                   |
| `page_size`                | integer        | Number of items requested per page                                                                                                  |

**Status Codes**:

* `200` - Transaction history retrieved successfully
* `400` - Invalid query parameters (including malformed `cursor`) (e.g., missing `app_session_id`, invalid `type`, malformed `cursor`)
* `401` - Authentication failed
* `500` - Internal server error

***

## GET /perpetual/positions

Retrieve all open positions for a specific perpetuals account.

**Position mode**: Each row’s `direction` is `long` or `short` (stored leg). A market may return two rows (one `long`, one `short`), since long and short legs are tracked separately.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

**Query Parameters**:

| Parameter        | Type   | Required | Options | Description                       |
| ---------------- | ------ | -------- | ------- | --------------------------------- |
| `app_session_id` | string | Yes      | —       | Perpetuals account app session ID |

**Request**:

```http
GET /perpetual/positions?app_session_id=your-session-id
```

**Response**:

```json
[
  {
    "uuid": "550e8400-e29b-41d4-a716-446655440000",
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
    "margin_mode": "cross",
    "locked_amount": "0.50000000",
    "available_amount": "1.00000000"
  }
]
```

**Response Fields**:

| Field                | Type           | Description                                                          |
| -------------------- | -------------- | -------------------------------------------------------------------- |
| `uuid`               | string         | Unique position identifier (always present)                          |
| `market`             | string         | Trading market (e.g., "BTCYTEST.USD-PERP")                           |
| `direction`          | string         | Position direction (`long` or `short`)                               |
| `amount`             | decimal string | Position size (always positive)                                      |
| `entry_price`        | decimal string | Average entry price                                                  |
| `mark_price`         | decimal string | Current mark price used for PnL calculation                          |
| `notional_value`     | decimal string | Current notional value (amount x mark\_price)                        |
| `leverage`           | decimal string | Leverage multiplier                                                  |
| `unrealized_pnl`     | decimal string | Current unrealized profit/loss                                       |
| `realized_pnl`       | decimal string | Realized profit/loss from partial closes                             |
| `total_pnl`          | decimal string | Total profit/loss (unrealized + realized)                            |
| `allocated_margin`   | decimal string | Margin allocated to this position                                    |
| `maintenance_margin` | decimal string | Maintenance margin requirement                                       |
| `liquidation_price`  | decimal string | Liquidation price (calculated by backend for cross-margin positions) |
| `margin_asset`       | string         | Collateral asset (e.g., "USDT")                                      |
| `margin_mode`        | string         | Margin mode (always `cross`)                                         |
| `locked_amount`      | decimal string | Amount currently locked                                              |
| `available_amount`   | decimal string | Amount available                                                     |

**Status Codes**:

* `200` - Positions retrieved successfully
* `400` - Missing app\_session\_id parameter
* `401` - Authentication failed
* `404` - Account not found
* `500` - Internal server error

***

## POST /perpetual/positions/close

Batch-submit **market IOC** close orders for every open position leg on the account that has **available** size greater than zero. Each leg uses the same path as `POST /perpetual/order`.

* If `market` is omitted or empty: all matching legs across markets are closed.
* If `market` is set: only positions for that market (case-insensitive match; symbol is normalized to uppercase for validation against exchange info).
* Legs are processed with bounded parallelism (worker pool) for lower latency.
* Safety cap: at most **50** closable legs are submitted per request; when more legs exist, the response includes `remaining_count` and `partial=true`.

**Position mode**: Each closable leg uses `direction` `long` or `short` with `reduce_only: true`. A market may have separate long and short legs, which are closed independently.

{% hint style="info" %}
**Authentication:** Required (trade scope). Ownership of `app_session_id` is verified.
{% endhint %}

**Request Body**:

```json
{
  "app_session_id": "your-session-id",
  "market": "BTCYTEST.USD-PERP"
}
```

**Request Fields**:

| Parameter        | Type   | Required | Options | Description                                                                                                                       |
| ---------------- | ------ | -------- | ------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `app_session_id` | string | Yes      | —       | Perpetuals account app session ID                                                                                                 |
| `market`         | string | No       | —       | When present, only this market's positions are closed; must exist in perpetuals exchange info when the market cache is configured |

**Response** (200):

```json
{
  "message": "dispatched 2 close order(s)",
  "positions_submitted": 2,
  "failed_count": 0,
  "remaining_count": 0,
  "partial": false,
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

| Field                  | Type    | Description                                                                |
| ---------------------- | ------- | -------------------------------------------------------------------------- |
| `message`              | string  | Summary for the client                                                     |
| `positions_submitted`  | integer | Number of close orders successfully dispatched                             |
| `failed_count`         | integer | Number of legs that failed validation or submission (see `failed`)         |
| `remaining_count`      | integer | Number of additional closable legs not processed due to per-request cap    |
| `partial`              | boolean | `true` when `remaining_count > 0` (caller should invoke again to continue) |
| `orders`               | array   | Successful items                                                           |
| `orders[].market`      | string  | Market of the submitted close order (from the position row)                |
| `orders[].direction`   | string  | Position direction (`long` or `short`)                                     |
| `orders[].margin_mode` | string  | Margin mode (always `cross`)                                               |
| `orders[].order_uuid`  | string  | UUID of the submitted order                                                |
| `failed`               | array   | Per-leg failures                                                           |
| `failed[].market`      | string  | Market of the failed leg                                                   |
| `failed[].direction`   | string  | (optional) Position direction (`long` or `short`)                          |
| `failed[].margin_mode` | string  | (optional) Margin mode (always `cross`)                                    |
| `failed[].error`       | string  | Failure description                                                        |
| `failed[].error_code`  | string  | (optional) Client-visible error code, same idea as `POST /perpetual/order` |

When there are no closable legs (all `available_amount` is zero), the endpoint returns `200` with `positions_submitted: 0` and an explanatory `message`.

**Status Codes**:

* `200` - Request completed (check `failed` for partial failures)
* `400` - Invalid JSON, missing `app_session_id`, or unknown `market` (when provided and exchange info is available)
* `401` - Authentication failed
* `403` - Permission denied for `(wallet_address, app_session_id)`
* `404` - Perpetual account not found
* `502` - Backend request failed
* `503` - Service temporarily unavailable

***

## POST /perpetual/leverage

Sets **initial leverage** per account and market. **Order creation does not accept leverage**; it is resolved from stored settings when matching orders. Use this endpoint (or the default, typically `10`) before trading if you need a specific multiplier.

**How to read current leverage**: `GET /perpetual/accounts` returns `initial_leverages` (market → leverage string). Positions and orders also expose an effective `leverage` field.

**Restrictions & behavior**:

* **Open orders**: Changing leverage is **rejected** while **any open order** exists for that **market** (`error` e.g. `order_exists`). Cancel or fill those orders first.
* **Open positions**: Changing leverage is **allowed**. The new leverage is applied atomically: position `allocated_margin` is recalculated from notional ÷ leverage, and locked balance is adjusted. The call may still **fail** if the account does not have enough free collateral for a higher required margin, or if the new margin would fall **below maintenance** for a position (`set_leverage_failed` or a similar message).
* **Validation**: `leverage >= 1` is required, and `leverage <= max_allowed_leverage` for that market (`leverage_exceeds_max`).

{% hint style="info" %}
**Authentication:** Required (JWT or API Key). The wallet address must be present in the auth context; ownership of `app_session_id` is verified.
{% endhint %}

**Request Body**:

```json
{
  "app_session_id": "your-session-id",
  "market": "BTCYTEST.USD-PERP",
  "leverage": "20"
}
```

**Request Fields**:

| Parameter        | Type             | Required | Options                                             | Description                                                                     |
| ---------------- | ---------------- | -------- | --------------------------------------------------- | ------------------------------------------------------------------------------- |
| `app_session_id` | string           | Yes      | —                                                   | Perpetuals account app session ID                                               |
| `market`         | string           | Yes      | —                                                   | Market symbol (e.g., `BTCYTEST.USD-PERP`). Trimmed and uppercased by the server |
| `leverage`       | number or string | Yes      | `>= 1` and `<= max_allowed_leverage` for the market | Leverage multiplier (e.g., `20` or `"20"`)                                      |

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

| Field      | Type           | Description                              |
| ---------- | -------------- | ---------------------------------------- |
| `success`  | boolean        | Whether the operation succeeded          |
| `message`  | string         | Human-readable message                   |
| `market`   | string         | Market symbol for which leverage was set |
| `leverage` | decimal string | Confirmed leverage                       |

**Error responses** (non-exhaustive; `success` is `false` when HTTP status is not 200):

| HTTP  | `error` (when present)                                                                                                                         | Meaning                                             |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| `400` | `invalid_request_format`                                                                                                                       | Malformed JSON                                      |
| `400` | `missing_app_session_id`                                                                                                                       | `app_session_id` empty                              |
| `400` | `missing_market`                                                                                                                               | `market` empty                                      |
| `400` | `invalid_leverage_value`                                                                                                                       | Leverage `< 1`                                      |
| `400` | `no_permitted`                                                                                                                                 | Account not found or not accessible for this wallet |
| `400` | `leverage_exceeds_max`, `order_exists`, `insufficient_balance`, `leverage_exceeds_safe_margin`, `account_not_active`, `set_leverage_failed`, … | See `message` for detail.                           |
| `401` | `missing_wallet_address`                                                                                                                       | Authenticated context has no wallet                 |
| `503` | `service_unavailable`                                                                                                                          | Service temporarily unavailable for this user       |
| `500` | `internal_error`                                                                                                                               | Internal server error                               |

**Error body** (400 from business rules):

```json
{
  "success": false,
  "error": "insufficient_balance",
  "message": "insufficient balance: insufficient available balance to apply new leverage (required extra margin: ..., available: ...)"
}
```

**Status Codes**:

* `200` - Leverage set successfully
* `400` - Validation or business rule failure (see table)
* `401` - Not authenticated or missing wallet in context
* `503` - Service temporarily unavailable
* `500` - Internal server error

***

## GET /perpetual/position-history

Retrieve closed position history with pagination. Only returns positions that have been fully closed. Default ordering matches `sort_by=closed_at` and `sort_dir=desc` with a stable secondary sort on internal row id.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

**Query Parameters**:

| Parameter        | Type    | Required | Options                                        | Description                                                                                                              |
| ---------------- | ------- | -------- | ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `app_session_id` | string  | Yes      | —                                              | Perpetuals account app session ID                                                                                        |
| `market`         | string  | No       | —                                              | Exact market symbol filter; normalized to uppercase before lookup (e.g. `btcytest.usd-perp` matches `BTCYTEST.USD-PERP`) |
| `opened_from`    | string  | No       | RFC3339 or RFC3339Nano                         | Inclusive lower bound on `opened_at`                                                                                     |
| `opened_to`      | string  | No       | RFC3339 or RFC3339Nano                         | Inclusive upper bound on `opened_at`                                                                                     |
| `closed_from`    | string  | No       | RFC3339 or RFC3339Nano                         | Inclusive lower bound on `closed_at`                                                                                     |
| `closed_to`      | string  | No       | RFC3339 or RFC3339Nano                         | Inclusive upper bound on `closed_at`                                                                                     |
| `sort_by`        | string  | No       | `opened_at`, `closed_at` (default `closed_at`) | Sort field; other values → `400`                                                                                         |
| `sort_dir`       | string  | No       | `asc`, `desc` (default `desc`)                 | Sort direction; other values → `400`                                                                                     |
| `use_cursor`     | boolean | No       | —                                              | Set `true` to use cursor pagination (first request). See [List pagination](perpetuals-trading-api.md#list-pagination).   |
| `cursor`         | string  | No       | —                                              | Opaque continuation token from the previous response's `next_cursor`. Omit on the first page.                            |
| `page_size`      | integer | No       | `1`–`100` (default `50`)                       | Number of positions per page; out-of-range when supplied → `400`                                                         |

**Request**:

```http
GET /perpetual/position-history?app_session_id=your-session-id&market=BTCYTEST.USD-PERP&opened_from=2024-01-01T00:00:00Z&closed_to=2024-01-31T23:59:59Z&sort_by=closed_at&sort_dir=desc&use_cursor=true&page_size=20
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
  "page_size": 20,
  "next_cursor": "1701943800000:550e8400-e29b-41d4-a716-446655440000",
  "has_more": true
}
```

**Pagination**: Cursor only (see [List pagination](perpetuals-trading-api.md#list-pagination)). Malformed `cursor` → `400` / `invalid_cursor`.

**Response Fields**:

| Field                           | Type           | Description                                                                                                  |
| ------------------------------- | -------------- | ------------------------------------------------------------------------------------------------------------ |
| `positions`                     | array          | Array of closed position objects                                                                             |
| `positions[].id`                | string         | Position ID (UUID)                                                                                           |
| `positions[].perps_account_id`  | string         | Perpetuals account ID (UUID)                                                                                 |
| `positions[].market`            | string         | Trading market                                                                                               |
| `positions[].direction`         | string         | Position direction (`long` or `short`)                                                                       |
| `positions[].amount`            | decimal string | Position size (amount)                                                                                       |
| `positions[].entry_price`       | decimal string | Average entry price                                                                                          |
| `positions[].leverage`          | decimal string | Leverage multiplier used                                                                                     |
| `positions[].allocated_margin`  | decimal string | Margin that was allocated                                                                                    |
| `positions[].realized_pnl`      | decimal string | Final realized profit/loss                                                                                   |
| `positions[].close_reason`      | string         | Close reason (`normal`, `liquidated`, `adl`)                                                                 |
| `positions[].exit_price`        | decimal string | Weighted-average close price over all close fills                                                            |
| `positions[].closed_quantity`   | decimal string | Total quantity closed for the position lifecycle                                                             |
| `positions[].max_held`          | decimal string | Peak absolute held quantity during the position lifecycle                                                    |
| `positions[].initial_margin`    | decimal string | Margin at first open (used for ROI)                                                                          |
| `positions[].total_trading_fee` | decimal string | Sum of linked trade fees for this position lifecycle                                                         |
| `positions[].net_funding_fee`   | decimal string | Signed cumulative funding for this position lifecycle (positive = received by user, negative = paid by user) |
| `positions[].liquidation_price` | decimal string | Liquidation trigger snapshot for liquidated positions (else zero/empty)                                      |
| `positions[].margin_mode`       | string         | Margin mode (always `cross`)                                                                                 |
| `positions[].pnl_ratio`         | decimal string | Realized ROE% = `realized_pnl / initial_margin * 100`                                                        |
| `positions[].opened_at`         | string         | Position open timestamp                                                                                      |
| `positions[].updated_at`        | string         | Last update timestamp                                                                                        |
| `positions[].closed_at`         | string         | Position close timestamp                                                                                     |
| `next_cursor`                   | string         | Opaque next-page token (empty when no more pages)                                                            |
| `has_more`                      | boolean        | Whether another page exists                                                                                  |
| `page_size`                     | integer        | Number of positions per page                                                                                 |

Note: for legacy closed positions created before these fields were persisted, `max_held` and `initial_margin` can be `0`, which means historical values are unavailable rather than actual zero exposure.\
Note: for positions that were already open when max-held/initial-margin persistence was introduced, `initial_margin` reflects migration-time/live-update value rather than the original open-time margin.

**Status Codes**:

* `200` - Position history retrieved successfully
* `400` - Invalid query parameters (including malformed `cursor`) (e.g. unparsable time strings, `opened_from` after `opened_to`, `closed_from` after `closed_to`, invalid `sort_by` / `sort_dir`, or `page_size` out of **1–100** when supplied)
* `401` - Authentication failed or missing wallet in context
* `403` - Permission denied (authenticated wallet is not the owner of `app_session_id`)
* `404` - Account not found
* `500` - Internal server error

***

## GET /perpetual/position-history/{id}

Retrieve **fill-level** history for a **single closed** position (one UUID from `GET /perpetual/position-history`). Results are **cursor-paginated** over the underlying trade fills for that position (chronological order). Summary fields for the position itself are available on the list endpoint item with the same `id`.

{% hint style="info" %}
**Authentication:** Required (JWT or API Key, `read:futures` scope). Owner is the authenticated wallet; must match the perpetuals account for `app_session_id`.
{% endhint %}

**Path Parameters**:

| Parameter | Type   | Required | Options | Description          |
| --------- | ------ | -------- | ------- | -------------------- |
| `id`      | string | Yes      | —       | Closed position UUID |

**Query Parameters**:

| Parameter        | Type    | Required | Options                                                              | Description                                                                                                                                                                  |
| ---------------- | ------- | -------- | -------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `app_session_id` | string  | Yes      | —                                                                    | Perpetuals account app session ID                                                                                                                                            |
| `page_size`      | integer | No       | positive integer, default `200`, values above `500` clamped to `500` | Number of underlying trades loaded per request                                                                                                                               |
| `cursor`         | string  | No       | `RFC3339Nano\|uint64` (pipe-separated)                               | Opaque pagination cursor from the previous response's `next_cursor`. Omit on the first page. Malformed values typically surface as `500` from the backend rather than `400`. |

**Pagination** (`cursor` / `next_cursor`):

* The server advances in **trade execution order** (`executed_at` ascending, then stable `id` tie-break).
* `next_cursor` is a tuple string: `<RFC3339Nano>|<uint64 trade row id>` (UTC in the timestamp part). Pass it back as `cursor` to fetch the next window.
* `has_more` is `true` when more trades exist after this page; when `false`, `next_cursor` is typically empty.

**Request** (first page):

```http
GET /perpetual/position-history/550e8400-e29b-41d4-a716-446655440000?app_session_id=your-session-id&page_size=50
```

**Request** (next page):

```http
GET /perpetual/position-history/550e8400-e29b-41d4-a716-446655440000?app_session_id=your-session-id&page_size=50&cursor=2023-12-07T10%3A05%3A10.000000000Z%7C12345
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

| Field                  | Type           | Description                                                                                                                                            |
| ---------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `fills`                | array          | Per-fill rows derived from trades in the current window (open and close perspectives are combined in one list; each row's `kind` is `open` or `close`) |
| `fills[].id`           | string         | Trade UUID                                                                                                                                             |
| `fills[].direction`    | string         | Position leg direction (`long` or `short`)                                                                                                             |
| `fills[].amount`       | decimal string | Fill amount                                                                                                                                            |
| `fills[].price`        | decimal string | Fill price                                                                                                                                             |
| `fills[].pnl`          | decimal string | Fill realized PnL (`0` for `open` fills)                                                                                                               |
| `fills[].fee`          | decimal string | Fee paid for that fill from this account perspective                                                                                                   |
| `fills[].fee_currency` | string         | Fee currency                                                                                                                                           |
| `fills[].exec_type`    | string         | Execution source (`trade`, `liquidation`, `liquidation_takeover`, `adl`)                                                                               |
| `fills[].kind`         | string         | Fill role (`open` or `close`)                                                                                                                          |
| `fills[].executed_at`  | string         | Fill execution timestamp (RFC3339 / RFC3339Nano)                                                                                                       |
| `next_cursor`          | string         | Cursor for the next request (empty when there is no next page)                                                                                         |
| `has_more`             | boolean        | Whether more trades exist after this page                                                                                                              |

**Status Codes**:

* `200` - Position history detail retrieved successfully
* `400` - Missing `app_session_id`, missing path `id`, invalid `page_size` (non-positive when supplied), or invalid position UUID
* `401` - Authentication failed or missing wallet in context
* `403` - Permission denied (wallet not owner of `app_session_id`, or position belongs to another account)
* `404` - Position not found (or no history payload)
* `409` - Position is not closed (`failed_precondition` — open positions have no fill history on this route)
* `500` - Internal server error (including malformed `cursor` in some deployments)

***

## POST /perpetual/order

Create a new perpetuals order.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

**Request Body** (`direction` is `long` or `short`):

```json
{
  "app_session_id": "your-session-id",
  "market": "BTCYTEST.USD-PERP",
  "side": "buy",
  "direction": "long",
  "type": "limit",
  "amount": "0.5",
  "price": "35000",
  "leverage": "10",
  "time_in_force": "gtc",
  "reduce_only": false
}
```

> **Position legs**: Long and short legs are tracked separately. Set `direction` to the leg you are trading and use `reduce_only` to close.

**Request Parameters**:

| Parameter         | Type    | Required | Options                                                | Description                                                                                                        |
| ----------------- | ------- | -------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------ |
| `app_session_id`  | string  | Yes      | —                                                      | Perpetuals account app session ID                                                                                  |
| `market`          | string  | Yes      | —                                                      | Trading market (e.g., "BTCYTEST.USD-PERP")                                                                         |
| `side`            | string  | Yes      | `buy`, `sell`                                          | Order side                                                                                                         |
| `direction`       | string  | Yes      | `long`, `short`                                        | Position leg direction                                                                                             |
| `type`            | string  | Yes      | `limit`, `market`, `trigger_limit`, `trigger_market`   | Order type                                                                                                         |
| `amount`          | string  | Yes      | —                                                      | Order amount in base asset (decimal format)                                                                        |
| `price`           | string  | No       | required for `limit` orders                            | Limit price                                                                                                        |
| `leverage`        | string  | Yes      | —                                                      | Sent with the request; effective margin rules still follow account/market settings from `POST /perpetual/leverage` |
| `time_in_force`   | string  | No       | `gtc`, `ioc`, `fok` (default `gtc` for `limit` orders) | Time in force — Good-Till-Cancelled, Immediate-Or-Cancel, Fill-Or-Kill                                             |
| `reduce_only`     | boolean | No       | default `false`                                        | If true, order can only reduce/close an existing position                                                          |
| `client_order_id` | string  | No       | —                                                      | Client-supplied order id                                                                                           |
| `trigger_price`   | string  | No       | —                                                      | For trigger / stop / take-profit style orders (see OpenAPI)                                                        |
| `trigger_type`    | string  | No       | —                                                      | For trigger / stop / take-profit style orders (see OpenAPI)                                                        |

**Leverage**: Also configured via `POST /perpetual/leverage` before the first order. Inspect `GET /perpetual/accounts` (`initial_leverages`) for current settings.

> **Margin mode**: All perpetual positions use cross margin. Passing `margin_mode` in the order request is not accepted.

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
* `400` - Invalid request parameters, validation failed, or the order was rejected (e.g. insufficient margin)
* `401` - Authentication failed
* `500` - Internal server error

**Error response** (typical for `400` — validation or rejection, same family as other trading routes):

```json
{
  "error": "insufficient_margin",
  "message": "insufficient margin. Available:9.62, Required:9.6256"
}
```

* `error`: Machine-readable code (`snake_case`), e.g. `insufficient_margin`, `lock_funds_rejected`, `validation_failed`.
* `message`: Human-readable detail.

***

## DELETE /perpetual/order

Cancel an existing perpetuals order.

> **Known Issue**: The API returns `200` with success message, but the order may remain in `wait` state. Cancel functionality is not reliably working.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

**Request Body**:

```json
{
  "app_session_id": "your-session-id",
  "market": "BTCYTEST.USD-PERP",
  "order_uuid": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Request Parameters**:

| Parameter        | Type   | Required | Options | Description                       |
| ---------------- | ------ | -------- | ------- | --------------------------------- |
| `app_session_id` | string | Yes      | —       | Perpetuals account app session ID |
| `market`         | string | Yes      | —       | Trading market                    |
| `order_uuid`     | string | Yes      | —       | UUID of the order to cancel       |

**Response**:

```json
{
  "message": "Order cancellation request sent successfully"
}
```

**Status Codes**:

* `200` - Cancellation request sent successfully
* `400` - Invalid request or missing parameters
* `401` - Authentication failed
* `500` - Internal server error

***

## DELETE /perpetual/orders

Cancel **all open** perpetuals orders for an account, optionally limited to one market.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

**Request Body**:

```json
{
  "app_session_id": "your-session-id",
  "market": "BTCYTEST.USD-PERP"
}
```

**Request Fields**:

| Parameter        | Type   | Required | Options | Description                                                                                                                                   |
| ---------------- | ------ | -------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `app_session_id` | string | Yes      | —       | Perpetuals account app session ID                                                                                                             |
| `market`         | string | No       | —       | When empty or omitted, open orders in all markets are targeted; when set, only orders for that exact market string (as stored on open orders) |

**Behavior**: The server lists open orders, then dispatches a cancellation for each one. Trigger orders in `trigger_wait` / `trigger_triggered` are also cancelled (same pattern as `DELETE /perpetual/order`).

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

| Field            | Type    | Description                                                                                         |
| ---------------- | ------- | --------------------------------------------------------------------------------------------------- |
| `message`        | string  | Summary                                                                                             |
| `total_orders`   | integer | Open orders found in the listing pass (omitted when count is zero in the "no open orders" response) |
| `canceled_count` | integer | Cancel requests successfully dispatched                                                             |
| `failed_count`   | integer | Cancel requests that failed to dispatch                                                             |

**Status Codes**:

* `200` - Listing and dispatch completed (check `failed_count` for partial publish failures)
* `400` - Invalid JSON or missing `app_session_id`
* `401` - Authentication failed
* `500` - Failed to list open orders

***

## GET /perpetual/orders

Retrieve perpetuals order history with pagination. Each order’s `direction` is stored as returned (`long` or `short`) — same idea as `GET /perpetual/open_orders`.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

**Query Parameters**:

| Parameter        | Type    | Required | Options                 | Description                                                                                                            |
| ---------------- | ------- | -------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `app_session_id` | string  | Yes      | —                       | Perpetuals account app session ID                                                                                      |
| `market`         | string  | No       | —                       | Exact filter by trading market                                                                                         |
| `market_like`    | string  | No       | —                       | Fuzzy filter by market (contains, case-insensitive)                                                                    |
| `use_cursor`     | boolean | No       | —                       | Set `true` to use cursor pagination (first request). See [List pagination](perpetuals-trading-api.md#list-pagination). |
| `cursor`         | string  | No       | —                       | Opaque continuation token from the previous response's `next_cursor`. Omit on the first page.                          |
| `page_size`      | integer | No       | default `50`, max `100` | Number of orders per page                                                                                              |

**Request**:

```http
GET /perpetual/orders?app_session_id=your-session-id&market=BTCYTEST.USD-PERP&use_cursor=true&page_size=20
```

**Response**:

```json
{
  "orders": [
    {
      "id": "1234567",
    "order_id": "550e8400-e29b-41d4-a716-446655440000",
    "app_session_id": "your-session-id",
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
  "page_size": 20,
  "next_cursor": "1234520",
  "has_more": true
}
```

**Response Fields**:

| Field                     | Type           | Description                                         |
| ------------------------- | -------------- | --------------------------------------------------- |
| `orders`                  | array          | Array of order objects                              |
| `orders[].id`             | string         | Internal order record ID                            |
| `orders[].order_id`       | string         | Order UUID                                          |
| `orders[].app_session_id` | string         | Perpetuals account app session ID                   |
| `orders[].market`         | string         | Trading market                                      |
| `orders[].price`          | decimal string | Order price                                         |
| `orders[].amount`         | decimal string | Current order amount (remaining to fill)            |
| `orders[].origin_amount`  | decimal string | Original order amount                               |
| `orders[].fill_amount`    | decimal string | Amount that has been filled                         |
| `orders[].notional`       | decimal string | Notional value (origin\_amount x price)             |
| `orders[].side`           | string         | Order side (`buy` or `sell`)                        |
| `orders[].type`           | string         | Order type (`limit`, `market`)                      |
| `orders[].state`          | string         | Order state (`pending`, `wait`, `done`, `canceled`) |
| `orders[].event`          | string         | Last order event                                    |
| `orders[].reason`         | string         | Reason for last state change                        |
| `orders[].leverage`       | decimal string | Leverage multiplier                                 |
| `orders[].created_at`     | string         | Order creation timestamp                            |
| `orders[].updated_at`     | string         | Last update timestamp                               |
| `orders[].completed_at`   | string         | Completion timestamp (empty for open orders)        |
| `page_size`               | integer        | Number of items per page                            |
| `next_cursor`             | string         | Cursor pagination token (empty when no more pages)  |
| `has_more`                | boolean        | Whether another page exists                         |

**Status Codes**:

* `200` - Orders retrieved successfully
* `400` - Invalid query parameters (including malformed `cursor`)
* `401` - Authentication failed
* `500` - Internal server error

***

## GET /perpetual/open\_orders

Retrieve open (unfilled) perpetuals orders. Returns only orders that are currently active in the order book. `direction` on each row is `long` or `short`.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

**Query Parameters**:

| Parameter        | Type    | Required | Options                 | Description                                                                                                            |
| ---------------- | ------- | -------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `app_session_id` | string  | Yes      | —                       | Perpetuals account app session ID                                                                                      |
| `market`         | string  | No       | —                       | Exact filter by trading market                                                                                         |
| `market_like`    | string  | No       | —                       | Fuzzy filter by market (contains, case-insensitive)                                                                    |
| `use_cursor`     | boolean | No       | —                       | Set `true` to use cursor pagination (first request). See [List pagination](perpetuals-trading-api.md#list-pagination). |
| `cursor`         | string  | No       | —                       | Opaque continuation token from the previous response's `next_cursor`. Omit on the first page.                          |
| `page_size`      | integer | No       | default `50`, max `100` | Number of orders per page                                                                                              |

**Request**:

```http
GET /perpetual/open_orders?app_session_id=your-session-id&market=BTCYTEST.USD-PERP&use_cursor=true&page_size=20
```

**Response**:

```json
{
  "orders": [
    {
      "id": "1234567",
    "order_id": "550e8400-e29b-41d4-a716-446655440000",
    "app_session_id": "your-session-id",
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
  "page_size": 20,
  "next_cursor": "1234520",
  "has_more": false
}
```

**Response Fields**:

| Field                     | Type           | Description                                                      |
| ------------------------- | -------------- | ---------------------------------------------------------------- |
| `orders`                  | array          | Array of open order objects (same format as `/perpetual/orders`) |
| `orders[].id`             | string         | Internal order record ID                                         |
| `orders[].order_id`       | string         | Order UUID                                                       |
| `orders[].app_session_id` | string         | Perpetuals account app session ID                                |
| `orders[].market`         | string         | Trading market                                                   |
| `orders[].price`          | decimal string | Order price                                                      |
| `orders[].amount`         | decimal string | Current order amount (remaining to fill)                         |
| `orders[].origin_amount`  | decimal string | Original order amount                                            |
| `orders[].fill_amount`    | decimal string | Amount that has been filled                                      |
| `orders[].notional`       | decimal string | Notional value (origin\_amount x price)                          |
| `orders[].side`           | string         | Order side (`buy` or `sell`)                                     |
| `orders[].type`           | string         | Order type (`limit`, `market`)                                   |
| `orders[].state`          | string         | Order state (typically `wait` for open orders)                   |
| `orders[].event`          | string         | Last order event                                                 |
| `orders[].reason`         | string         | Reason for last state change                                     |
| `orders[].leverage`       | decimal string | Leverage multiplier                                              |
| `orders[].created_at`     | string         | Order creation timestamp                                         |
| `orders[].updated_at`     | string         | Last update timestamp                                            |
| `orders[].completed_at`   | string         | Completion timestamp (empty for open orders)                     |
| `page_size`               | integer        | Number of items per page                                         |
| `next_cursor`             | string         | Cursor pagination token (empty when no more pages)               |
| `has_more`                | boolean        | Whether another page exists                                      |

**Note**: This endpoint only returns orders with open states (`wait`, `pending`). For complete order history including filled and cancelled orders, use `/perpetual/orders`.

**Status Codes**:

* `200` - Open orders retrieved successfully
* `400` - Invalid query parameters (including malformed `cursor`)
* `401` - Authentication failed
* `500` - Internal server error

***

## GET /perpetual/trades

Retrieve perpetuals trade history with pagination. Returns only the user's own trades with privacy-focused response format.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

**Query Parameters**:

| Parameter        | Type    | Required | Options                 | Description                                                                                                            |
| ---------------- | ------- | -------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `app_session_id` | string  | Yes      | —                       | Perpetuals account app session ID                                                                                      |
| `market`         | string  | No       | —                       | Exact filter by trading market                                                                                         |
| `market_like`    | string  | No       | —                       | Fuzzy filter by market (contains, case-insensitive)                                                                    |
| `start_time`     | string  | No       | RFC3339 or RFC3339Nano  | Filter executed\_at >= start\_time                                                                                     |
| `end_time`       | string  | No       | RFC3339 or RFC3339Nano  | Filter executed\_at <= end\_time                                                                                       |
| `use_cursor`     | boolean | No       | —                       | Set `true` to use cursor pagination (first request). See [List pagination](perpetuals-trading-api.md#list-pagination). |
| `cursor`         | string  | No       | —                       | Opaque continuation token from the previous response's `next_cursor`. Omit on the first page.                          |
| `page_size`      | integer | No       | default `50`, max `100` | Number of trades per page                                                                                              |

**Request**:

```http
GET /perpetual/trades?app_session_id=your-session-id&market=BTCYTEST.USD-PERP&use_cursor=true&page_size=50
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
  "page_size": 50,
  "next_cursor": "1701943800000:5721092",
  "has_more": true
}
```

**Response Fields**:

| Field                  | Type           | Description                                                                                                                                                                                                    |
| ---------------------- | -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `trades`               | array          | Array of trade objects                                                                                                                                                                                         |
| `trades[].id`          | string         | Trade record ID                                                                                                                                                                                                |
| `trades[].order_uuid`  | string         | User's own order UUID in this trade (matches create order response)                                                                                                                                            |
| `trades[].market`      | string         | Trading market                                                                                                                                                                                                 |
| `trades[].base_asset`  | string         | Base asset symbol (e.g., "BTC")                                                                                                                                                                                |
| `trades[].quote_asset` | string         | Quote asset symbol (e.g., "USD")                                                                                                                                                                               |
| `trades[].amount`      | decimal string | Trade amount in base asset                                                                                                                                                                                     |
| `trades[].price`       | decimal string | Trade execution price                                                                                                                                                                                          |
| `trades[].is_buyer`    | boolean        | Whether the user was the buyer                                                                                                                                                                                 |
| `trades[].is_maker`    | boolean        | Whether the user was the maker/liquidity provider                                                                                                                                                              |
| `trades[].fee`         | decimal string | Trading fee charged to the user                                                                                                                                                                                |
| `trades[].exec_type`   | string         | Execution source: `trade` (normal matched trade), `liquidation` (IOC liquidation fill or cross liquidation netting), `liquidation_takeover` (position taken over by insurance fund), `adl` (auto-deleveraging) |
| `trades[].executed_at` | string         | Trade execution timestamp                                                                                                                                                                                      |
| `trades[].created_at`  | string         | Trade record creation timestamp                                                                                                                                                                                |
| `page_size`            | integer        | Number of items per page                                                                                                                                                                                       |
| `next_cursor`          | string         | Cursor pagination token (empty when no more pages)                                                                                                                                                             |
| `has_more`             | boolean        | Whether another page exists                                                                                                                                                                                    |

**Note**: The API response only exposes the user's own order UUID and does not expose counterparty information for privacy reasons. User-relative flags (`is_buyer`, `is_maker`) indicate the user's role in the trade.

**Status Codes**:

* `200` - Trades retrieved successfully
* `400` - Invalid query parameters (including malformed `cursor`)
* `401` - Authentication failed
* `500` - Internal server error

***

## GET /perpetual/funding-rate/:symbol

Retrieve the current funding rate for a specific perpetuals market, plus the previous funding interval when available.

{% hint style="info" %}
**Authentication:** Not required (public endpoint)
{% endhint %}

**Path Parameters**:

| Parameter | Type   | Required | Options | Description                                |
| --------- | ------ | -------- | ------- | ------------------------------------------ |
| `symbol`  | string | Yes      | —       | Trading symbol (e.g., "BTCYTEST.USD-PERP") |

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

| Field                                  | Type           | Description                                                        |
| -------------------------------------- | -------------- | ------------------------------------------------------------------ |
| `current_funding_rate`                 | object         | Current funding rate object                                        |
| `current_funding_rate.market`          | string         | Trading market symbol                                              |
| `current_funding_rate.funding_rate`    | decimal string | Current funding rate                                               |
| `current_funding_rate.premium_index`   | decimal string | Premium index value                                                |
| `current_funding_rate.mark_price`      | decimal string | Current mark price                                                 |
| `current_funding_rate.index_price`     | decimal string | Current index price                                                |
| `current_funding_rate.interval_start`  | string         | Start of the current funding interval (ISO 8601 format)            |
| `current_funding_rate.interval_end`    | string         | End of the current funding interval (ISO 8601 format)              |
| `current_funding_rate.created_at`      | string         | When the current funding rate was recorded (ISO 8601 format)       |
| `previous_funding_rate`                | object         | Previous funding interval object (may be omitted if not available) |
| `previous_funding_rate.market`         | string         | Trading market symbol                                              |
| `previous_funding_rate.funding_rate`   | decimal string | Previous interval funding rate                                     |
| `previous_funding_rate.premium_index`  | decimal string | Previous interval premium index                                    |
| `previous_funding_rate.mark_price`     | decimal string | Mark price used for previous interval                              |
| `previous_funding_rate.index_price`    | decimal string | Index price used for previous interval                             |
| `previous_funding_rate.interval_start` | string         | Start of the previous funding interval (ISO 8601 format)           |
| `previous_funding_rate.interval_end`   | string         | End of the previous funding interval (ISO 8601 format)             |
| `previous_funding_rate.created_at`     | string         | When the previous funding rate was recorded (ISO 8601 format)      |

**Status Codes**:

* `200` - Funding rate retrieved successfully
* `400` - Invalid symbol or validation failed
* `500` - Internal server error

***

## GET /perpetual/funding-rates

Retrieve funding rate history with pagination.

{% hint style="info" %}
**Authentication:** Not required (public endpoint)
{% endhint %}

**Query Parameters**:

| Parameter    | Type    | Required | Options                 | Description                                                                                                            |
| ------------ | ------- | -------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `symbol`     | string  | No       | —                       | Filter by trading market                                                                                               |
| `use_cursor` | boolean | No       | —                       | Set `true` to use cursor pagination (first request). See [List pagination](perpetuals-trading-api.md#list-pagination). |
| `cursor`     | string  | No       | —                       | Opaque continuation token from the previous response's `next_cursor`. Omit on the first page.                          |
| `page_size`  | integer | No       | default `50`, max `100` | Number of rates per page                                                                                               |

**Request** (cursor first page):

```http
GET /perpetual/funding-rates?symbol=BTCYTEST.USD-PERP&use_cursor=true&page_size=20
```

**Request** (next page):

```http
GET /perpetual/funding-rates?symbol=BTCYTEST.USD-PERP&cursor=1701943800000:42&page_size=20
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
  "page_size": 20,
  "next_cursor": "1701943800000:42",
  "has_more": true
}
```

**Response Fields**:

| Field           | Type    | Description                                                                         |
| --------------- | ------- | ----------------------------------------------------------------------------------- |
| `funding_rates` | array   | Array of funding rate objects (same structure as `/perpetual/funding-rate/:symbol`) |
| `page_size`     | integer | Number of rates per page                                                            |
| `next_cursor`   | string  | Cursor pagination token (empty when no more pages)                                  |
| `has_more`      | boolean | Whether another page exists                                                         |

**Status Codes**:

* `200` - Funding rates retrieved successfully
* `400` - Invalid query parameters (including malformed `cursor`)
* `500` - Internal server error

***

## GET /perpetual/account/funding-payments

Retrieve funding payment history for the specified app session with pagination.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

**Query Parameters**:

| Parameter        | Type    | Required | Options                 | Description                                                                                                            |
| ---------------- | ------- | -------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `app_session_id` | string  | Yes      | —                       | App session identifier passed by frontend                                                                              |
| `interval_start` | string  | No       | ISO 8601 format         | Filter by funding interval start time                                                                                  |
| `use_cursor`     | boolean | No       | —                       | Set `true` to use cursor pagination (first request). See [List pagination](perpetuals-trading-api.md#list-pagination). |
| `cursor`         | string  | No       | —                       | Opaque continuation token from the previous response's `next_cursor`. Omit on the first page.                          |
| `page_size`      | integer | No       | default `50`, max `100` | Number of payments per page                                                                                            |

**Request**:

```http
GET /perpetual/account/funding-payments?app_session_id=your-session-id&use_cursor=true&page_size=20
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
  "page_size": 20,
  "next_cursor": "1701943800000:550e8400-e29b-41d4-a716-446655440000",
  "has_more": true
}
```

**Response Fields**:

| Field                               | Type           | Description                                                                                 |
| ----------------------------------- | -------------- | ------------------------------------------------------------------------------------------- |
| `funding_payments`                  | array          | Array of funding payment objects                                                            |
| `funding_payments[].id`             | string         | Funding payment ID (UUID)                                                                   |
| `funding_payments[].market`         | string         | Trading market                                                                              |
| `funding_payments[].account_id`     | string         | Perpetuals account ID (UUID)                                                                |
| `funding_payments[].position_id`    | string         | Position ID (UUID)                                                                          |
| `funding_payments[].side`           | string         | Position side (`long` or `short`)                                                           |
| `funding_payments[].position_size`  | decimal string | Position size at the time of funding payment                                                |
| `funding_payments[].mark_price`     | decimal string | Mark price at the time of funding payment                                                   |
| `funding_payments[].funding_rate`   | decimal string | Funding rate applied                                                                        |
| `funding_payments[].funding_amount` | decimal string | Funding payment amount (positive for long positions receiving payment, negative for paying) |
| `funding_payments[].interval_start` | string         | Start of the funding interval (ISO 8601 format)                                             |
| `funding_payments[].interval_end`   | string         | End of the funding interval (ISO 8601 format)                                               |
| `funding_payments[].created_at`     | string         | When the funding payment was recorded (ISO 8601 format)                                     |
| `next_cursor`                       | string         | Opaque next-page token (empty when no more pages)                                           |
| `has_more`                          | boolean        | Whether another page exists                                                                 |
| `page_size`                         | integer        | Number of payments per page                                                                 |

**Status Codes**:

* `200` - Funding payments retrieved successfully
* `400` - Invalid query parameters (including malformed `cursor`) or missing app\_session\_id
* `401` - Authentication failed
* `500` - Internal server error

***

## GET /perpetual/position/funding-payments

Retrieve funding payment history for a specific position within the specified app session, with pagination.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

**Query Parameters**:

| Parameter        | Type    | Required | Options                 | Description                                                                                                            |
| ---------------- | ------- | -------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `app_session_id` | string  | Yes      | —                       | App session identifier passed by frontend                                                                              |
| `position_id`    | string  | Yes      | —                       | Position ID (UUID)                                                                                                     |
| `interval_start` | string  | No       | ISO 8601 format         | Filter by funding interval start time                                                                                  |
| `use_cursor`     | boolean | No       | —                       | Set `true` to use cursor pagination (first request). See [List pagination](perpetuals-trading-api.md#list-pagination). |
| `cursor`         | string  | No       | —                       | Opaque continuation token from the previous response's `next_cursor`. Omit on the first page.                          |
| `page_size`      | integer | No       | default `50`, max `100` | Number of payments per page                                                                                            |

**Request**:

```http
GET /perpetual/position/funding-payments?app_session_id=your-session-id&position_id=550e8400-e29b-41d4-a716-446655440002&use_cursor=true&page_size=20
```

**Response**: Same format as `/perpetual/account/funding-payments` (array of funding payment objects with pagination metadata)

**Status Codes**:

* `200` - Funding payments retrieved successfully
* `400` - Invalid query parameters (including malformed `cursor`) or missing position\_id/app\_session\_id
* `401` - Authentication failed
* `500` - Internal server error
