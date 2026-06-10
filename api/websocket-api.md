# WebSocket Api

The Yellow WebSocket API provides real-time data streams, order notifications, account updates, and authenticated subscription access. All connections use a single endpoint:

```
wss://trade.api.yellow.pro/ws
```

Both **authenticated** and **unauthenticated** connections are supported. Unauthenticated connections can subscribe to public market data; authenticated connections additionally receive private account, order, and balance notifications. All messages are JSON.

## Connecting

Every WebSocket connection — authenticated or not — **must** send a `connect` command immediately after the socket opens. This handshake establishes the session before any subscriptions or notifications are delivered.

### Connection Requirements

* **Protocol**: WebSocket (RFC 6455)
* **Authentication**: Optional (token-based or header-based)
* **Reconnection**: Automatic with exponential backoff (see [Connection management](websocket-api.md#connection-management))
* **Message format**: JSON (the server may batch multiple JSON objects per frame, see [Message batching](websocket-api.md#message-batching))

### Connect Handshake (Required)

Send an unauthenticated `connect` command right after the socket opens:

```json
{"id": 1, "connect": {}}
```

**Success response**:

```json
{
  "id": 1,
  "connect": {
    "client": "3d43bec7-0e0c-429c-a29f-8830bef64437",
    "data": {},
    "ping": 300,
    "pong": true
  }
}
```

{% hint style="danger" %}
A bare WebSocket connection that does not send the `connect` handshake will be closed by the server with error `3501 (bad request)`.
{% endhint %}

## Authentication

To receive private notifications, authenticate the connection. Two methods are available.

{% tabs %}
{% tab title="Token-based (recommended)" %}
Pass your JWT in the `connect` payload after the socket opens. This method works in all environments, including browsers.

```json
{
  "id": 1,
  "connect": {
    "token": "<JWT_TOKEN>"
  }
}
```

**Success response** — the server auto-subscribes you to your private channel and includes it in `subs`:

```json
{
  "id": 1,
  "connect": {
    "client": "3d43bec7-0e0c-429c-a29f-8830bef64437",
    "data": {},
    "subs": {
      "private.0x1234567890abcdef1234567890abcdef12345678": {
        "recoverable": true,
        "epoch": "bQsD",
        "positioned": true
      }
    },
    "ping": 300,
    "pong": true
  }
}
```
{% endtab %}

{% tab title="Header-based (Node.js only)" %}
Pass the JWT as an `Authorization: Bearer` header when constructing the WebSocket, then still send the `connect` command after the socket opens.

```javascript
const ws = new WebSocket('wss://trade.api.yellow.pro/ws', [], {
  headers: {
    'Authorization': 'Bearer <JWT_TOKEN>'
  }
});

// Still must send the connect command once the connection is open
ws.on('open', () => {
  ws.send(JSON.stringify({ "id": 1, "connect": {} }));
});
```

{% hint style="warning" %}
Custom WebSocket headers only work with Node.js (e.g. the `ws` library). Browser WebSocket APIs do not support custom headers — use the token-based method in browsers.
{% endhint %}
{% endtab %}
{% endtabs %}

## Message format

All WebSocket messages follow a consistent JSON structure.

### Outbound messages (client → server)

```json
{
  "id": 1,
  "subscribe": {
    "channel": "channel_name"
  }
}
```

Each outbound command carries a unique, incrementing `id` and a single action object (`connect`, `subscribe`, or `unsubscribe`).

### Inbound messages (server → client)

**Response messages** echo the request `id`:

```json
{
  "id": 1,
  "subscribe": {}
}
```

**Push notifications** carry no `id` and are wrapped in a `push` envelope:

```json
{
  "push": {
    "channel": "channel_name",
    "pub": {
      "data": {}
    }
  }
}
```

| Field           | Type   | Description                                         |
| --------------- | ------ | --------------------------------------------------- |
| `push.channel`  | string | Channel the message belongs to                      |
| `push.pub.data` | object | Notification payload (shape depends on the channel) |

### Message batching

The server may send multiple JSON objects in a single WebSocket frame, separated by newlines (NDJSON / newline-delimited JSON):

```
{"push":{"channel":"public.tickers.24h","pub":{"data":{}}}}
{"push":{"channel":"public.kline.BTCUSD.1m","pub":{"data":{}}}}
```

{% hint style="warning" %}
Calling `JSON.parse(message)` directly on a batched frame throws a `SyntaxError`. Split the frame by newline and parse each line individually:

```javascript
function parseMessages(raw) {
  return raw.trim().split('\n')
    .filter(line => line.trim())
    .map(line => JSON.parse(line));
}
```
{% endhint %}

## Subscriptions

After a successful authenticated handshake, you are **automatically subscribed** to your private notification channel — no explicit subscribe is required. Public channels must be subscribed to explicitly (see [Subscription management](websocket-api.md#subscription-management)).

### Private channel name format

* **Pattern**: `private.{wallet_address}`
* **Example**: `private.0x1234567890abcdef1234567890abcdef12345678`

All private notifications (account, order, balance, transfer events) are delivered on this single channel; the event `type` inside `data.header` distinguishes them.

## Private channels

Private notifications are pushed on `private.{wallet_address}` for authenticated connections. Each notification carries a `header` with metadata and a `type` identifying the event.

### Perpetuals account update

Account-level aggregate metrics for perpetuals (cross-margin). Pushed on balance/position changes (event-triggered) and periodically (timer-driven, default every 3s) for price-driven equity updates.

**Notification type**: `perpetuals_account.account_update`

```json
{
  "push": {
    "channel": "private.0x1234567890abcdef1234567890abcdef12345678",
    "pub": {
      "data": {
        "header": {
          "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
          "scope": "private",
          "type": "perpetuals_account.account_update",
          "user_address": "0x1234567890abcdef1234567890abcdef12345678",
          "app_session_id": "your-session-id",
          "created_at": "2025-09-30T15:40:54.455631184Z"
        },
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "app_session_id": "your-session-id",
        "owner": "0x1234567890abcdef1234567890abcdef12345678",
        "state": "active",
        "total_account_balance": "10000.00000000",
        "total_unrealized_pnl": "150.00000000",
        "total_account_equity": "10150.00000000",
        "total_allocated_margin": "5265.00000000",
        "total_locked_balance": "5265.00000000",
        "total_maintenance_margin": "2632.50000000",
        "available_balance": "4885.00000000",
        "total_balance_usd": "10150.00000000",
        "allocated_balance_usd": "5265.00000000",
        "locked_balance_usd": "5265.00000000",
        "available_balance_usd": "4885.00000000",
        "transferable_balances": { "USDT": "3908.00000000" },
        "position_modes": { "BTCUSD-PERP": "HEDGE" },
        "initial_leverages": { "BTCUSD-PERP": "10" }
      }
    }
  }
}
```

| Field                           | Type   | Description                                                                         |
| ------------------------------- | ------ | ----------------------------------------------------------------------------------- |
| `data.header`                   | object | Event metadata (`type`, `user_address`, `app_session_id`, `created_at`)             |
| `data.id`                       | string | Perpetuals account ID                                                               |
| `data.app_session_id`           | string | App session ID of the perpetuals account                                            |
| `data.owner`                    | string | Wallet address of the account owner                                                 |
| `data.state`                    | string | Account state (`active`, `closing`, `closed`)                                       |
| `data.total_account_balance`    | string | Total balance across all collateral assets                                          |
| `data.total_unrealized_pnl`     | string | Sum of unrealized PnL across all positions                                          |
| `data.total_account_equity`     | string | Total equity (balance + unrealized PnL)                                             |
| `data.total_allocated_margin`   | string | Total margin allocated to open positions (same value as `total_locked_balance`)     |
| `data.total_locked_balance`     | string | Sum of locked collateral across all assets (orders + position margin)               |
| `data.total_maintenance_margin` | string | Total maintenance margin required                                                   |
| `data.available_balance`        | string | Account-level balance available for new positions (includes unrealized PnL)         |
| `data.total_balance_usd`        | string | Total balance in stablecoin equivalent; `"0"` when no price                         |
| `data.allocated_balance_usd`    | string | Allocated/locked margin in stablecoin equivalent; `"0"` when no price               |
| `data.locked_balance_usd`       | string | Locked balance in stablecoin equivalent; `"0"` when no price                        |
| `data.available_balance_usd`    | string | Available balance in stablecoin equivalent; `"0"` when no price                     |
| `data.transferable_balances`    | object | Optional. Map of collateral asset → max perp→spot transfer amount (decimal strings) |
| `data.position_modes`           | object | Optional. Map of market → position mode (`ONE_WAY` or `HEDGE`)                      |
| `data.initial_leverages`        | object | Optional. Map of market → leverage string (e.g. `"10"`)                             |

### Perpetuals balance update

Real-time updates on perpetuals account balance changes for a specific collateral asset.

**Notification type**: `perpetuals_account.balance_update`

```json
{
  "push": {
    "channel": "private.0x1234567890abcdef1234567890abcdef12345678",
    "pub": {
      "data": {
        "header": {
          "id": "df444db3-6db8-40e0-9e32-4b499c24d955",
          "scope": "private",
          "type": "perpetuals_account.balance_update",
          "user_address": "0x1234567890abcdef1234567890abcdef12345678",
          "app_session_id": "your-session-id",
          "created_at": "2025-09-30T15:40:54.455631184Z"
        },
        "app_session_id": "your-session-id",
        "owner_address": "0x1234567890abcdef1234567890abcdef12345678",
        "asset_symbol": "USDT",
        "total_balance": "10000.00000000",
        "allocated_balance": "2000.00000000",
        "locked_balance": "2000.00000000",
        "available_balance": "8000.00000000",
        "total_balance_usd": "10000.00000000",
        "allocated_balance_usd": "2000.00000000",
        "locked_balance_usd": "2000.00000000",
        "available_balance_usd": "8000.00000000",
        "last_updated": "2025-09-30T15:40:54.455631184Z"
      }
    }
  }
}
```

| Field                        | Type   | Description                                                           |
| ---------------------------- | ------ | --------------------------------------------------------------------- |
| `data.header`                | object | Event metadata                                                        |
| `data.app_session_id`        | string | App session ID of the perpetuals account                              |
| `data.owner_address`         | string | Wallet address of the account owner                                   |
| `data.asset_symbol`          | string | Collateral asset symbol (e.g. `USDT`, `USDC`)                         |
| `data.total_balance`         | string | Total balance for this asset                                          |
| `data.allocated_balance`     | string | Margin allocated to open positions (same value as `locked_balance`)   |
| `data.locked_balance`        | string | Locked collateral for this asset (orders + position margin)           |
| `data.available_balance`     | string | Balance available for new positions (includes unrealized PnL share)   |
| `data.total_balance_usd`     | string | Total balance in stablecoin equivalent; `"0"` when no price           |
| `data.allocated_balance_usd` | string | Allocated/locked margin in stablecoin equivalent; `"0"` when no price |
| `data.locked_balance_usd`    | string | Locked collateral in stablecoin equivalent; `"0"` when no price       |
| `data.available_balance_usd` | string | Available balance in stablecoin equivalent; `"0"` when no price       |
| `data.last_updated`          | string | Last balance update timestamp                                         |

### Perpetuals position update

Real-time updates on perpetuals position snapshots (size, margin, leverage-related fields). Pushed on position open and on each fill, and after a successful leverage change for that market (one update per open leg). Not pushed on mark price changes alone.

**Notification type**: `perpetuals_account.position_update`

```json
{
  "push": {
    "channel": "private.0x1234567890abcdef1234567890abcdef12345678",
    "pub": {
      "data": {
        "header": {
          "id": "6c9f53ea-7171-41f8-97c0-119241478703",
          "scope": "private",
          "type": "perpetuals_account.position_update",
          "user_address": "0x1234567890abcdef1234567890abcdef12345678",
          "app_session_id": "your-session-id",
          "created_at": "2025-09-30T15:40:54.473694018Z"
        },
        "app_session_id": "your-session-id",
        "market": "BTCUSD",
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
    }
  }
}
```

| Field                     | Type   | Description                                  |
| ------------------------- | ------ | -------------------------------------------- |
| `data.header`             | object | Event metadata                               |
| `data.app_session_id`     | string | App session ID of the position               |
| `data.market`             | string | Trading market (e.g. `BTCUSD`)               |
| `data.direction`          | string | Position direction (`long` or `short`)       |
| `data.amount`             | string | Position size (always positive)              |
| `data.entry_price`        | string | Average entry price                          |
| `data.mark_price`         | string | Current mark price used for PnL calculation  |
| `data.notional_value`     | string | Current notional value (amount × mark price) |
| `data.leverage`           | string | Effective leverage                           |
| `data.unrealized_pnl`     | string | Current unrealized profit/loss               |
| `data.realized_pnl`       | string | Realized profit/loss from partial closes     |
| `data.total_pnl`          | string | Total profit/loss (unrealized + realized)    |
| `data.allocated_margin`   | string | Margin allocated to this position            |
| `data.maintenance_margin` | string | Maintenance margin requirement               |
| `data.liquidation_price`  | string | Liquidation price (cross-margin)             |
| `data.margin_asset`       | string | Collateral asset (e.g. `USDT`)               |

### Perpetuals funding payment

Emitted when a funding settlement is applied to an open position.

**Notification type**: `perpetuals_account.funding_payment`

```json
{
  "push": {
    "channel": "private.0x1234567890abcdef1234567890abcdef12345678",
    "pub": {
      "data": {
        "header": {
          "id": "notif-uuid",
          "scope": "private",
          "type": "perpetuals_account.funding_payment",
          "user_address": "0x1234567890abcdef1234567890abcdef12345678",
          "app_session_id": "your-session-id",
          "created_at": "2026-04-29T12:00:00Z"
        },
        "app_session_id": "your-session-id",
        "owner_address": "0x1234567890abcdef1234567890abcdef12345678",
        "id": "funding-payment-event-uuid",
        "market": "BTC-USDT-PERP",
        "account_id": "550e8400-e29b-41d4-a716-446655440000",
        "position_id": "660e8400-e29b-41d4-a716-446655440001",
        "side": "long",
        "position_size": "0.50000000",
        "mark_price": "95000.00000000",
        "funding_rate": "0.00010000",
        "funding_amount": "-1.25000000"
      }
    }
  }
}
```

| Field                 | Type   | Description                                                                                           |
| --------------------- | ------ | ----------------------------------------------------------------------------------------------------- |
| `data.header`         | object | Event metadata                                                                                        |
| `data.app_session_id` | string | App session ID                                                                                        |
| `data.owner_address`  | string | Wallet address of the account owner                                                                   |
| `data.id`             | string | Unique ID for this funding payment event                                                              |
| `data.market`         | string | Perpetual market symbol                                                                               |
| `data.account_id`     | string | Perpetuals account UUID                                                                               |
| `data.position_id`    | string | Position UUID                                                                                         |
| `data.side`           | string | `long` or `short`                                                                                     |
| `data.position_size`  | string | Absolute position size at settlement                                                                  |
| `data.mark_price`     | string | Mark price used for the payment                                                                       |
| `data.funding_rate`   | string | Funding rate applied for the interval                                                                 |
| `data.funding_amount` | string | Signed payment in quote collateral. Positive = debited (you paid); negative = credited (you received) |

### Perpetuals liquidation warning

Real-time warning for cross-margin perpetual accounts. Sent when the danger ratio reaches configured thresholds.

**Notification type**: `perpetuals_account.liquidation_warning`

```json
{
  "push": {
    "channel": "private.0x1234567890abcdef1234567890abcdef12345678",
    "pub": {
      "data": {
        "header": {
          "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
          "scope": "private",
          "type": "perpetuals_account.liquidation_warning",
          "user_address": "0x1234567890abcdef1234567890abcdef12345678",
          "app_session_id": "your-session-id",
          "created_at": "2026-02-24T10:00:00Z"
        },
        "app_session_id": "your-session-id",
        "owner": "0x1234567890abcdef1234567890abcdef12345678",
        "warning_level": "final",
        "reminder_type": "first_cross",
        "margin_ratio": "95.0000",
        "threshold": "95",
        "triggered_at": "2026-04-06T10:00:00Z",
        "next_remind_at": "2026-04-06T11:00:00Z"
      }
    }
  }
}
```

| Field                 | Type   | Description                                                          |
| --------------------- | ------ | -------------------------------------------------------------------- |
| `data.warning_level`  | string | `early` \| `strong` \| `final`                                       |
| `data.reminder_type`  | string | `first_cross` \| `hourly`                                            |
| `data.margin_ratio`   | string | Danger ratio percent (`maintenance_margin / equity * 100`)           |
| `data.threshold`      | string | Triggered threshold (`80`, `90`, `95`)                               |
| `data.triggered_at`   | string | Current warning trigger time (UTC)                                   |
| `data.next_remind_at` | string | Optional. Next planned reminder time, present for `final` level only |

**Trigger rules**:

* `>= 80%`: triggers once when crossing into this band; resets only after the ratio drops below `75%`
* `>= 90%`: triggers once when crossing into this band; resets only after the ratio drops below `85%`
* `>= 95%`: crossing can trigger a popup with a minimum `10m` interval between popups; hourly reminders continue while still `>= 95%`
* `< 95%`: final-level hourly reminders stop (clear by state/absence; no separate cleared event)
* `>= 100%`: no warning popup (liquidation path takes priority)

### Order updates

Real-time notifications for order state changes, delivered via two types: `order.updated` and `order.cancelled`.

{% hint style="info" %}
Clients must listen for **both** `order.updated` and `order.cancelled` to receive all order notifications. Spot order notifications identify the order with `order_id`; perpetual order notifications use `uuid`.
{% endhint %}

#### `order.updated` — order state changes

Sent when an order is created, partially filled, or fully filled.

**Perpetual order example**:

```json
{
  "push": {
    "channel": "private.0x1234567890abcdef1234567890abcdef12345678",
    "pub": {
      "data": {
        "header": {
          "id": "9db2705e-ba95-4292-8639-f7cd91acc473",
          "scope": "private",
          "type": "order.updated",
          "user_address": "0x1234567890abcdef1234567890abcdef12345678",
          "app_session_id": "your-session-id",
          "created_at": "2025-09-30T15:40:54.458854996Z"
        },
        "id": 220507,
        "uuid": "125a9ddf-bc6b-4f07-82a7-f218d385b8e3",
        "app_session_id": "your-session-id",
        "user_address": "0x1234567890abcdef1234567890abcdef12345678",
        "market": "BTCUSD",
        "side": "buy",
        "type": "take_limit",
        "time_in_force": "gtc",
        "price": "114280",
        "trigger_price": "113000",
        "leverage": "1",
        "amount": "0",
        "origin_amount": "0.01",
        "state": "done",
        "position_mode": "ONE_WAY",
        "direction": "long",
        "reduce_only": false,
        "created_at": "2025-09-30T15:40:54.455065563Z",
        "triggered_at": "2025-09-30T15:40:54.456000000Z",
        "completed_at": "2025-09-30T15:40:54.460000000Z"
      }
    }
  }
}
```

**Spot order example**:

```json
{
  "push": {
    "channel": "private.0x1234567890abcdef1234567890abcdef12345678",
    "pub": {
      "data": {
        "header": {
          "type": "order.updated"
        },
        "order_id": "5d418afd-d0d1-430a-b1b5-4fc98828ce32",
        "market": "BTCUSD",
        "side": "buy",
        "type": "limit",
        "state": "wait"
      }
    }
  }
}
```

| Field                | Type    | Description                                                                                            |
| -------------------- | ------- | ------------------------------------------------------------------------------------------------------ |
| `data.header`        | object  | Event metadata                                                                                         |
| `data.id`            | number  | Order record ID (perpetual orders)                                                                     |
| `data.uuid`          | string  | Order UUID (perpetual orders only)                                                                     |
| `data.order_id`      | string  | Order UUID (spot orders only)                                                                          |
| `data.user_address`  | string  | User's wallet address                                                                                  |
| `data.market`        | string  | Trading market (e.g. `BTCUSD`)                                                                         |
| `data.side`          | string  | Order side (`buy` or `sell`)                                                                           |
| `data.type`          | string  | Order type (`limit`, `market`, `take_limit`, `post_only`, etc.)                                        |
| `data.time_in_force` | string  | Time in force (`gtc`, `ioc`, `fok`)                                                                    |
| `data.price`         | string  | Execution price                                                                                        |
| `data.trigger_price` | string  | Trigger price for trigger-originated orders; omitted otherwise                                         |
| `data.leverage`      | string  | Effective leverage (perpetual orders)                                                                  |
| `data.amount`        | string  | Current order amount                                                                                   |
| `data.origin_amount` | string  | Original order amount                                                                                  |
| `data.state`         | string  | Order state (`pending`, `wait`, `filled`, `done`, `canceled`)                                          |
| `data.position_mode` | string  | Position mode for perpetual orders (`ONE_WAY`); omitted for spot                                       |
| `data.direction`     | string  | Position direction for perpetual orders (`long`, `short`, `both`); omitted for spot                    |
| `data.reduce_only`   | boolean | `true` = order can only reduce a position (Close); `false` = regular order. Spot orders always `false` |
| `data.created_at`    | string  | Order creation timestamp                                                                               |
| `data.triggered_at`  | string  | Timestamp when the trigger condition was met (trigger orders only)                                     |
| `data.completed_at`  | string  | Timestamp when the order reached a final state; omitted while still open                               |

#### `order.cancelled` — order cancellation

Sent when an order is cancelled. This is a separate event type from `order.updated`.

```json
{
  "push": {
    "channel": "private.0x1234567890abcdef1234567890abcdef12345678",
    "pub": {
      "data": {
        "header": {
          "type": "order.cancelled"
        },
        "order_id": "5d418afd-d0d1-430a-b1b5-4fc98828ce32",
        "state": "canceled"
      }
    }
  }
}
```

| Field              | Type   | Description                   |
| ------------------ | ------ | ----------------------------- |
| `data.header.type` | string | `"order.cancelled"`           |
| `data.order_id`    | string | Order UUID (spot orders)      |
| `data.uuid`        | string | Order UUID (perpetual orders) |
| `data.state`       | string | `"canceled"`                  |

### Order expiration

Notification for an expired order.

**Notification type**: `order.expired`

```json
{
  "push": {
    "channel": "private.0x1234567890abcdef1234567890abcdef12345678",
    "pub": {
      "data": {
        "header": {
          "id": "8b8eb153-1acd-4438-b5d2-6d8715b53bb2",
          "scope": "private",
          "type": "order.expired",
          "user_address": "0x1234567890abcdef1234567890abcdef12345678",
          "app_session_id": "your-session-id",
          "created_at": "2025-09-30T15:42:43.71880063Z"
        },
        "id": 222488,
        "uuid": "ad04afd7-e581-4f40-a038-edd14c58c54b",
        "app_session_id": "your-session-id",
        "user_address": "0x1234567890abcdef1234567890abcdef12345678",
        "market": "BTCUSD",
        "side": "buy",
        "type": "limit",
        "time_in_force": "fok",
        "price": "1",
        "leverage": "1",
        "amount": "4037.1",
        "origin_amount": "4037.1",
        "state": "expired",
        "created_at": "2025-09-30T15:42:43.71319921Z"
      }
    }
  }
}
```

| Field                | Type   | Description                        |
| -------------------- | ------ | ---------------------------------- |
| `data.header`        | object | Event metadata                     |
| `data.id`            | number | Order record ID (perpetual orders) |
| `data.uuid`          | string | Order UUID (perpetual orders)      |
| `data.market`        | string | Trading market                     |
| `data.side`          | string | Order side (`buy` or `sell`)       |
| `data.type`          | string | Order type                         |
| `data.time_in_force` | string | Time in force                      |
| `data.price`         | string | Order price                        |
| `data.amount`        | string | Current order amount               |
| `data.origin_amount` | string | Original order amount              |
| `data.state`         | string | `"expired"`                        |
| `data.created_at`    | string | Order creation timestamp           |

### Spot account state update

Real-time notification for spot account state changes (opened, closed, etc.).

**Notification type**: `spot_account.state_update`

```json
{
  "push": {
    "channel": "private.0x1234567890abcdef1234567890abcdef12345678",
    "pub": {
      "data": {
        "header": {
          "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
          "scope": "private",
          "type": "spot_account.state_update",
          "user_address": "0x1234567890abcdef1234567890abcdef12345678",
          "created_at": "2025-10-24T12:00:00.000000000Z"
        },
        "account_id": "550e8400-e29b-41d4-a716-446655440000",
        "owner_address": "0x1234567890abcdef1234567890abcdef12345678",
        "app_session_id": "your-session-id",
        "state": "open"
      }
    }
  }
}
```

| Field                 | Type   | Description                         |
| --------------------- | ------ | ----------------------------------- |
| `data.header`         | object | Event metadata                      |
| `data.account_id`     | string | Spot account identifier             |
| `data.owner_address`  | string | Wallet address of the account owner |
| `data.app_session_id` | string | Associated app session ID           |
| `data.state`          | string | Account state (`open`, `closed`)    |

### Spot balance update

Real-time updates on spot account balance changes for a specific asset. Delivered on the `spot_account.balance_update` channel.

**Notification type**: `spot_account.balance_update`

```json
{
  "push": {
    "channel": "private.0x1234567890abcdef1234567890abcdef12345678",
    "pub": {
      "data": {
        "header": {
          "id": "b1c2d3e4-f5a6-7890-bcde-f12345678901",
          "scope": "private",
          "type": "spot_account.balance_update",
          "user_address": "0x1234567890abcdef1234567890abcdef12345678",
          "created_at": "2025-10-24T12:00:00.000000000Z"
        },
        "app_session_id": "your-session-id",
        "owner_address": "0x1234567890abcdef1234567890abcdef12345678",
        "asset_symbol": "USDT",
        "total_balance": "10000.5000",
        "available_balance": "9500.5000",
        "locked_balance": "500.0000",
        "total_balance_usd": "10000.5000",
        "available_balance_usd": "9500.5000",
        "locked_balance_usd": "500.0000",
        "last_updated": "2025-10-24T12:00:00.000000000Z"
      }
    }
  }
}
```

| Field                        | Type   | Description                                                     |
| ---------------------------- | ------ | --------------------------------------------------------------- |
| `data.header`                | object | Event metadata                                                  |
| `data.app_session_id`        | string | Associated app session ID                                       |
| `data.owner_address`         | string | Wallet address of the account owner                             |
| `data.asset_symbol`          | string | Asset symbol (e.g. `USDT`, `BTC`, `ETH`)                        |
| `data.total_balance`         | string | Total balance for this asset                                    |
| `data.available_balance`     | string | Available balance for trading/withdrawal                        |
| `data.locked_balance`        | string | Balance locked in orders or pending operations                  |
| `data.total_balance_usd`     | string | Total balance in stablecoin equivalent; `"0"` when no price     |
| `data.available_balance_usd` | string | Available balance in stablecoin equivalent; `"0"` when no price |
| `data.locked_balance_usd`    | string | Locked balance in stablecoin equivalent; `"0"` when no price    |
| `data.last_updated`          | string | Timestamp of last balance update                                |

### Spot funds deposited

Notification when funds are successfully deposited into a spot account.

**Notification type**: `spot_account.funds_deposited`

```json
{
  "push": {
    "channel": "private.0x1234567890abcdef1234567890abcdef12345678",
    "pub": {
      "data": {
        "header": {
          "id": "c1d2e3f4-a5b6-7890-cdef-123456789012",
          "scope": "private",
          "type": "spot_account.funds_deposited",
          "user_address": "0x1234567890abcdef1234567890abcdef12345678",
          "created_at": "2025-10-24T12:00:00.000000000Z"
        },
        "app_session_id": "your-session-id",
        "owner_address": "0x1234567890abcdef1234567890abcdef12345678",
        "asset_symbol": "USDT",
        "amount": "1000.0000",
        "transaction_hash": "0xabc123",
        "deposited_at": "2025-10-24T12:00:00.000000000Z"
      }
    }
  }
}
```

| Field                   | Type   | Description                             |
| ----------------------- | ------ | --------------------------------------- |
| `data.header`           | object | Event metadata                          |
| `data.app_session_id`   | string | Associated app session ID               |
| `data.owner_address`    | string | Wallet address of the account owner     |
| `data.asset_symbol`     | string | Deposited asset symbol                  |
| `data.amount`           | string | Deposit amount                          |
| `data.transaction_hash` | string | Optional. Transaction hash or reference |
| `data.deposited_at`     | string | Timestamp when the deposit completed    |

### Spot withdrawal accepted

Notification when a withdrawal request is accepted and funds are reserved.

**Notification type**: `spot_account.withdrawal_accepted`

```json
{
  "push": {
    "channel": "private.0x1234567890abcdef1234567890abcdef12345678",
    "pub": {
      "data": {
        "header": {
          "id": "d1e2f3a4-b5c6-7890-def1-234567890123",
          "scope": "private",
          "type": "spot_account.withdrawal_accepted",
          "user_address": "0x1234567890abcdef1234567890abcdef12345678",
          "created_at": "2025-10-24T12:00:00.000000000Z"
        },
        "withdrawal_id": "withdrawal-550e8400-e29b-41d4-a716",
        "app_session_id": "your-session-id",
        "owner_address": "0x1234567890abcdef1234567890abcdef12345678",
        "asset_symbol": "USDT",
        "amount": "500.0000",
        "requested_at": "2025-10-24T12:00:00.000000000Z"
      }
    }
  }
}
```

| Field                 | Type   | Description                                 |
| --------------------- | ------ | ------------------------------------------- |
| `data.header`         | object | Event metadata                              |
| `data.withdrawal_id`  | string | Withdrawal identifier                       |
| `data.app_session_id` | string | Associated app session ID                   |
| `data.owner_address`  | string | Wallet address of the account owner         |
| `data.asset_symbol`   | string | Asset being withdrawn                       |
| `data.amount`         | string | Withdrawal amount                           |
| `data.requested_at`   | string | Timestamp when the withdrawal was requested |

### Spot withdrawal completed

Notification when a withdrawal is successfully completed.

**Notification type**: `spot_account.withdrawal_completed`

```json
{
  "push": {
    "channel": "private.0x1234567890abcdef1234567890abcdef12345678",
    "pub": {
      "data": {
        "header": {
          "id": "e1f2a3b4-c5d6-7890-ef12-345678901234",
          "scope": "private",
          "type": "spot_account.withdrawal_completed",
          "user_address": "0x1234567890abcdef1234567890abcdef12345678",
          "created_at": "2025-10-24T12:05:00.000000000Z"
        },
        "withdrawal_id": "withdrawal-550e8400-e29b-41d4-a716",
        "app_session_id": "your-session-id",
        "owner_address": "0x1234567890abcdef1234567890abcdef12345678",
        "asset_symbol": "USDT",
        "amount": "500.0000",
        "transaction_hash": "0xdef456",
        "completed_at": "2025-10-24T12:05:00.000000000Z"
      }
    }
  }
}
```

| Field                   | Type   | Description                             |
| ----------------------- | ------ | --------------------------------------- |
| `data.header`           | object | Event metadata                          |
| `data.withdrawal_id`    | string | Withdrawal identifier                   |
| `data.app_session_id`   | string | Associated app session ID               |
| `data.owner_address`    | string | Wallet address of the account owner     |
| `data.asset_symbol`     | string | Asset that was withdrawn                |
| `data.amount`           | string | Withdrawal amount                       |
| `data.transaction_hash` | string | Transaction hash for the withdrawal     |
| `data.completed_at`     | string | Timestamp when the withdrawal completed |

### Spot withdrawal failed

Notification when a withdrawal attempt fails and funds are returned to the available balance.

**Notification type**: `spot_account.withdrawal_failed`

```json
{
  "push": {
    "channel": "private.0x1234567890abcdef1234567890abcdef12345678",
    "pub": {
      "data": {
        "header": {
          "id": "f1a2b3c4-d5e6-7890-f123-456789012345",
          "scope": "private",
          "type": "spot_account.withdrawal_failed",
          "user_address": "0x1234567890abcdef1234567890abcdef12345678",
          "created_at": "2025-10-24T12:05:00.000000000Z"
        },
        "withdrawal_id": "withdrawal-550e8400-e29b-41d4-a716",
        "app_session_id": "your-session-id",
        "owner_address": "0x1234567890abcdef1234567890abcdef12345678",
        "asset_symbol": "USDT",
        "amount": "500.0000",
        "failure_reason": "Insufficient liquidity in destination",
        "failed_at": "2025-10-24T12:05:00.000000000Z"
      }
    }
  }
}
```

| Field                 | Type   | Description                          |
| --------------------- | ------ | ------------------------------------ |
| `data.header`         | object | Event metadata                       |
| `data.withdrawal_id`  | string | Withdrawal identifier                |
| `data.app_session_id` | string | Associated app session ID            |
| `data.owner_address`  | string | Wallet address of the account owner  |
| `data.asset_symbol`   | string | Asset that was attempted             |
| `data.amount`         | string | Withdrawal amount that failed        |
| `data.failure_reason` | string | Reason for the failure               |
| `data.failed_at`      | string | Timestamp when the withdrawal failed |

### Funds transferred (spot ↔ perpetuals)

Real-time notification when funds are transferred between spot and perpetuals accounts. Delivered on the `transfer_updates` channel. The notification is pushed when the transfer reaches a terminal or failure-relevant state.

**Notification type**: `account.funds_transferred`

```json
{
  "push": {
    "channel": "transfer_updates",
    "pub": {
      "data": {
        "header": {
          "id": "f4a5b6c7-d8e9-0123-abcd-456789012345",
          "scope": "private",
          "type": "account.funds_transferred",
          "user_address": "0x1234567890abcdef1234567890abcdef12345678",
          "app_session_id": "your-session-id",
          "created_at": "2026-03-09T10:30:00.000000000Z"
        },
        "transfer_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
        "owner_address": "0x1234567890abcdef1234567890abcdef12345678",
        "app_session_id": "your-session-id",
        "source_type": "spot",
        "dest_type": "perps",
        "asset_symbol": "USDT",
        "amount": "500.00000000",
        "state": "dest_completed",
        "timestamp": "2026-03-09T10:30:00.000000000Z"
      }
    }
  }
}
```

**Failed transfer example**:

```json
{
  "push": {
    "channel": "transfer_updates",
    "pub": {
      "data": {
        "header": {
          "id": "c0d7b65a-2a25-4f7b-a59a-4ff8b0d2e9d1",
          "scope": "private",
          "type": "account.funds_transferred",
          "user_address": "0x1234567890abcdef1234567890abcdef12345678",
          "app_session_id": "your-session-id",
          "created_at": "2026-03-09T10:31:00.000000000Z"
        },
        "transfer_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
        "owner_address": "0x1234567890abcdef1234567890abcdef12345678",
        "app_session_id": "your-session-id",
        "source_type": "spot",
        "dest_type": "perps",
        "asset_symbol": "USDT",
        "amount": "500.00000000",
        "state": "failed",
        "failure_stage": "source",
        "failure_reason": "insufficient balance",
        "timestamp": "2026-03-09T10:31:00.000000000Z"
      }
    }
  }
}
```

| Field                 | Type   | Description                                                                                              |
| --------------------- | ------ | -------------------------------------------------------------------------------------------------------- |
| `data.header`         | object | Event metadata (`app_session_id` is shared between spot and perps accounts)                              |
| `data.transfer_id`    | string | Transfer identifier (UUID, idempotency key)                                                              |
| `data.owner_address`  | string | Wallet address of the owner initiating the transfer                                                      |
| `data.app_session_id` | string | App session ID (shared between the user's spot and perpetuals accounts)                                  |
| `data.source_type`    | string | Source account type (`spot` or `perps`)                                                                  |
| `data.dest_type`      | string | Destination account type (`spot` or `perps`)                                                             |
| `data.asset_symbol`   | string | Asset being transferred (e.g. `USDT`, `USDC`)                                                            |
| `data.amount`         | string | Transfer amount                                                                                          |
| `data.state`          | string | Transfer state (`dest_completed`, `failed`, `compensation_needed`, `compensated`, `compensation_failed`) |
| `data.failure_reason` | string | Optional. Failure reason (failure/compensation states)                                                   |
| `data.failure_stage`  | string | Optional. Which stage failed: `source`, `dest`, or `compensation`                                        |
| `data.timestamp`      | string | Timestamp for this transfer state change                                                                 |

{% hint style="info" %}
After a successful transfer you will also receive `spot_account.balance_update` and `perpetuals_account.balance_update` notifications reflecting the updated balances on both sides.
{% endhint %}

## Public channels

Public channels are available to all connections (authenticated or not) and must be subscribed to explicitly.

### Mark price

Real-time mark price updates for a market.

**Channel pattern**: `public.mark_price.{MARKET}`

**Subscribe**:

```json
{
  "id": 2,
  "subscribe": {
    "channel": "public.mark_price.BTCUSD"
  }
}
```

**Push notification**:

```json
{
  "push": {
    "channel": "public.mark_price.BTCUSD",
    "pub": {
      "data": {
        "header": {
          "id": "960fcdd0-6f0b-46a5-9647-6297093e168a",
          "scope": "public",
          "type": "mark_price",
          "created_at": "2025-09-30T14:42:52.153092769Z"
        },
        "market": "BTCUSD",
        "mark_price": "113143.17314493",
        "index_price": "113143.17314493",
        "funding_rate": -0.000001,
        "next_funding_time": "2026-03-16T08:00:00Z"
      }
    }
  }
}
```

| Field                    | Type   | Description                        |
| ------------------------ | ------ | ---------------------------------- |
| `data.market`            | string | Market symbol                      |
| `data.mark_price`        | string | Current mark price                 |
| `data.index_price`       | string | Current index price                |
| `data.funding_rate`      | number | Current funding rate               |
| `data.next_funding_time` | string | Next funding settlement time (UTC) |

### Order book (incremental)

Real-time order book changes. Subscribing returns an initial snapshot followed by incremental updates.

**Channel pattern**: `public.orderbook.increment.{MARKET}`

**Subscribe**:

```json
{
  "id": 3,
  "subscribe": {
    "channel": "public.orderbook.increment.BTCUSD"
  }
}
```

**Initial snapshot response**:

```json
{
  "id": 3,
  "subscribe": {
    "data": {
      "header": {
        "id": "b626d90c-dc5d-4be7-9021-de265ea6764c",
        "scope": "public",
        "type": "orderbook.snapshot",
        "created_at": "2025-09-30T14:42:51.949396721Z"
      },
      "market": "BTCUSD",
      "sequence_num": 1436308,
      "bids": [
        ["113150", "0.15741658"],
        ["113140", "0.08147461"]
      ],
      "asks": [
        ["113170", "0.23884178"],
        ["113180", "0.21656283"]
      ]
    }
  }
}
```

**Incremental update**:

```json
{
  "push": {
    "channel": "public.orderbook.increment.BTCUSD",
    "pub": {
      "data": {
        "header": {
          "id": "5ee5f313-e5f3-4392-b167-a485481acd6a",
          "scope": "public",
          "type": "orderbook.increment",
          "created_at": "2025-09-30T14:42:52.009378122Z"
        },
        "market": "BTCUSD",
        "sequence_num": 1436309,
        "bids": [
          ["113140", "0.23295975"]
        ],
        "asks": []
      }
    }
  }
}
```

| Field               | Type   | Description                                                   |
| ------------------- | ------ | ------------------------------------------------------------- |
| `data.market`       | string | Trading market symbol                                         |
| `data.sequence_num` | number | Monotonically increasing sequence number for ordering updates |
| `data.bids`         | array  | Bid levels `[price, amount]`; empty array means no changes    |
| `data.asks`         | array  | Ask levels `[price, amount]`; empty array means no changes    |

{% hint style="info" %}
Updates are aggregated within a short window (default 20ms): multiple changes to the same price level within a window are merged into the final state, and clients receive roughly 50 updates per second. A level set to `0` amount has been removed. Sequence numbers stay monotonically increasing — treat any gap as a reason to resubscribe.
{% endhint %}

### Trade stream (snapshot + increment)

Aggregated trade executions for a market. Each subscription receives a snapshot followed by incremental updates with guaranteed sequence continuity.

**Channel pattern**: `public.trades.increment.{MARKET}`

**Subscribe**:

```json
{
  "id": 7,
  "subscribe": {
    "channel": "public.trades.increment.BTC-USDT"
  }
}
```

**Snapshot response**:

```json
{
  "id": 7,
  "subscribe": {
    "data": {
      "header": {
        "id": "37521ebc-38fc-4498-83df-e7ffe40f2297",
        "scope": "public",
        "type": "trades.snapshot",
        "created_at": "2025-11-24T02:45:36.441934775Z"
      },
      "market": "BTC-USDT",
      "sequence_num": 10,
      "trades": [
        {
          "id": 123455,
          "price": "67999.80",
          "amount": "0.0100",
          "direction": "sell",
          "executed_at": "2025-11-24T02:45:36.341934775Z"
        },
        {
          "id": 123456,
          "price": "68000.25",
          "amount": "0.0025",
          "direction": "buy",
          "executed_at": "2025-11-24T02:45:36.441934775Z"
        }
      ]
    }
  }
}
```

**Incremental update**:

```json
{
  "push": {
    "channel": "public.trades.increment.BTC-USDT",
    "pub": {
      "data": {
        "header": {
          "id": "2a5084e6-43d0-4a9a-8d7d-95a83827d9c6",
          "scope": "public",
          "type": "trades.increment",
          "created_at": "2025-11-24T02:45:36.491934775Z"
        },
        "market": "BTC-USDT",
        "sequence_num": 11,
        "trades": [
          {
            "id": 123457,
            "price": "68001.10",
            "amount": "0.0150",
            "direction": "buy",
            "executed_at": "2025-11-24T02:45:36.481934775Z"
          }
        ]
      }
    }
  }
}
```

| Field                       | Type   | Description                                                    |
| --------------------------- | ------ | -------------------------------------------------------------- |
| `data.market`               | string | Market symbol (e.g. `BTC-USDT`)                                |
| `data.sequence_num`         | number | Monotonically increasing sequence number guaranteeing ordering |
| `data.trades`               | array  | Array of aggregated trade entries                              |
| `data.trades[].id`          | number | Monotonic identifier within the aggregation window             |
| `data.trades[].price`       | string | Executed price                                                 |
| `data.trades[].amount`      | string | Executed amount                                                |
| `data.trades[].direction`   | string | Side of the taker order (`buy` or `sell`)                      |
| `data.trades[].executed_at` | string | ISO 8601 timestamp with microsecond precision                  |

{% hint style="info" %}
Every subscription starts with a snapshot whose `sequence_num` matches the most recent increment. Sequence numbers are contiguous; if a gap or duplicate is detected, the server rebuilds and broadcasts a fresh snapshot. Treat any missing sequence number as invalid and resubscribe. Default snapshot depth is 50 aggregated entries per market.
{% endhint %}

### 24h ticker

24-hour ticker statistics for all markets.

**Channel**: `public.tickers.24h`

**Subscribe**:

```json
{
  "id": 4,
  "subscribe": {
    "channel": "public.tickers.24h"
  }
}
```

**Push notification**:

```json
{
  "push": {
    "channel": "public.tickers.24h",
    "pub": {
      "data": {
        "header": {
          "id": "75591ea1-0887-4b3d-8a5d-24400b3fb2ca",
          "scope": "public",
          "type": "tickers",
          "created_at": "2025-09-30T15:37:27.92730374Z"
        },
        "tickers": [
          {
            "market": "BTCUSD",
            "time": 1759246647916,
            "min": "113404.5",
            "max": "114377.2",
            "first": "113924.9",
            "last": "114288.3",
            "volume": "31003.27754213",
            "quoteVolume": "3534745005.83033429",
            "vwap": "114011.9782828447619754",
            "priceChange": "+0.32%"
          }
        ]
      }
    }
  }
}
```

| Field                        | Type   | Description                               |
| ---------------------------- | ------ | ----------------------------------------- |
| `data.tickers[].market`      | string | Market symbol                             |
| `data.tickers[].time`        | number | Timestamp of the data (Unix milliseconds) |
| `data.tickers[].min`         | string | 24h lowest price                          |
| `data.tickers[].max`         | string | 24h highest price                         |
| `data.tickers[].first`       | string | 24h opening price                         |
| `data.tickers[].last`        | string | Current/last price                        |
| `data.tickers[].volume`      | string | 24h base volume                           |
| `data.tickers[].quoteVolume` | string | 24h quote volume                          |
| `data.tickers[].vwap`        | string | Volume-weighted average price             |
| `data.tickers[].priceChange` | string | 24h price change percentage               |

### Kline / candlestick

Real-time kline (candlestick) data for a market and interval.

**Channel pattern**: `public.kline.{MARKET}.{INTERVAL}`

**Available intervals**: `1m`, `3m`, `5m`, `15m`, `30m`, `1h`, `2h`, `4h`, `6h`, `8h`, `12h`, `1d`, `3d`, `1w`, `1M`

**Subscribe**:

```json
{
  "id": 6,
  "subscribe": {
    "channel": "public.kline.BTCUSD.15m"
  }
}
```

**Push notification**:

```json
{
  "push": {
    "channel": "public.kline.BTCUSD.15m",
    "pub": {
      "data": {
        "header": {
          "id": "1b7a62eb-5f54-4492-a9c4-a7144335d183",
          "scope": "public",
          "type": "kline",
          "created_at": "2025-09-30T14:42:56.947631975Z"
        },
        "market": "BTCUSD",
        "interval": "15m",
        "kline": [
          1759242600000,
          "113044.8",
          "113225.5",
          "112978.9",
          "113137.7",
          "925.77107192",
          39456
        ]
      }
    }
  }
}
```

The `kline` array is positional:

| Index | Type   | Description              |
| ----- | ------ | ------------------------ |
| `[0]` | number | Open time (milliseconds) |
| `[1]` | string | Open price               |
| `[2]` | string | High price               |
| `[3]` | string | Low price                |
| `[4]` | string | Close price              |
| `[5]` | string | Volume                   |
| `[6]` | number | Number of trades         |

## Subscription management

Subscribe to and unsubscribe from channels with `subscribe` / `unsubscribe` commands.

{% hint style="info" %}
Each subscribe/unsubscribe message must use a unique, incrementing `id` (n+1). The `id` lets you match each request to its response.
{% endhint %}

### Subscribe

**Request**:

```json
{
  "id": 1,
  "subscribe": {
    "channel": "channel_name"
  }
}
```

**Success response**:

```json
{
  "id": 1,
  "subscribe": {}
}
```

**Error response**:

```json
{
  "id": 1,
  "subscribe": {
    "error": {
      "code": 400,
      "message": "Invalid channel name"
    }
  }
}
```

### Unsubscribe

**Request**:

```json
{
  "id": 2,
  "unsubscribe": {
    "channel": "channel_name"
  }
}
```

**Success response**:

```json
{
  "id": 2,
  "unsubscribe": {}
}
```

## Connection management

### Ping / pong

The server sends periodic ping frames to maintain connection health; clients respond with pong frames.

* **Ping interval**: 5 minutes
* **Pong timeout**: 5 minutes

### Reconnection

Implement automatic reconnection with exponential backoff:

* **Initial delay**: 1 second
* **Maximum delay**: 60 seconds
* **Backoff multiplier**: 2

### Connection recovery

The WebSocket API supports recovery on reconnect:

* **Message positioning**: resume from the last received message
* **Automatic resubscription**: previous subscriptions are restored
* **State synchronization**: account state is synchronized on reconnection

## Order placement

Placing and canceling orders over WebSocket is not available. Use the REST API instead:

* [Spot Trading API](spot-trading-api.md)
* [Perpetuals Trading API](perpetuals-trading-api.md)

Order **notifications** (`order.updated`, `order.cancelled`, `order.expired`) are still delivered over WebSocket on your private channel — see [Order updates](websocket-api.md#order-updates).

## Error handling

### Common error codes

**Authentication errors**:

* `401` — Authentication failed or token invalid
* `403` — Insufficient permissions for the requested operation

**Subscription errors**:

* `400` — Invalid channel name or subscription parameters
* `404` — Channel not found or not available
* `409` — Already subscribed to the channel

### Error message format

All error responses include structured error information:

```json
{
  "error": {
    "code": 400,
    "message": "Human-readable error description",
    "details": {
      "field": "additional_error_context"
    }
  }
}
```

## Rate limits

WebSocket connections are subject to rate limiting to ensure fair usage:

* **Connection limit**: 5 concurrent connections per user
* **Subscription limit**: 100 active subscriptions per connection
* **Message rate limit**: 100 messages per second per connection

Rate limiting violations result in connection termination.
