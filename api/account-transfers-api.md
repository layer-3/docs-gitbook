# Account Transfers API

The account transfer API provides an endpoint for transferring funds between spot and perpetuals accounts. A transfer request returns `202 Accepted` when the request is accepted; you are notified of completion on the `transfer_updates` WebSocket channel.

## POST /accounts/transfer

Initiate a fund transfer between spot and perpetuals accounts. The request returns `202 Accepted` when accepted, and you are notified of completion on the `transfer_updates` WebSocket channel.

{% hint style="info" %}
**Authentication:** Required (JWT or API Key)
{% endhint %}

**Request Body**:

```json
{
  "app_session_id": "your-session-id",
  "source_account_type": "spot",
  "dest_account_type": "perps",
  "asset_symbol": "USDT",
  "amount": "1000.50"
}
```

**Request Parameters**:

| Parameter             | Type   | Required | Options          | Description                                                                                                                   |
| --------------------- | ------ | -------- | ---------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `app_session_id`      | string | Yes      | —                | User's session ID (shared between spot and perps accounts)                                                                    |
| `source_account_type` | string | Yes      | `spot`, `perps`  | Source account type                                                                                                           |
| `dest_account_type`   | string | Yes      | `spot`, `perps`  | Destination account type                                                                                                      |
| `asset_symbol`        | string | Yes      | —                | Stablecoin asset to transfer (e.g., "USDT"). Only stablecoin assets that are active on both spot and perps sides are allowed. |
| `amount`              | string | Yes      | Positive decimal | Amount to transfer (decimal format)                                                                                           |

**Asset Restrictions**:

* Only **stablecoin** assets can be transferred between spot and perps accounts.
* The asset must be active as a stablecoin on both the spot and perps sides.
* Use `GET /perpetual/transfer-assets` to fetch the list of transferable stablecoin assets.

**Perpetual → Spot (perps as source)**:

When transferring from a perps account to spot, the maximum transferable amount is capped per request:

`transferable_max = max(0, min(total_balance - locked_balance, available_balance)) × 80%`

* `available_balance` follows balance-query semantics (it includes unrealized PnL where applicable).
* If `amount` exceeds `transferable_max`, the transfer is rejected with an insufficient-balance error.

**Balance Notifications**:

* After a successful transfer, both the source and destination accounts emit balance update events over WebSocket so clients can refresh displayed balances without polling.
* Spot balance updates are delivered on the `spot_account.balance_update` channel.
* Perps balance updates are delivered on the `perpetuals_account.balance_update` channel.

**Response** (202 Accepted):

```json
{
  "transfer_id": "550e8400-e29b-41d4-a716-446655440000",
  "message": "Transfer initiated successfully. You will be notified when the transfer completes."
}
```

**Response Fields**:

| Field         | Type   | Description                           |
| ------------- | ------ | ------------------------------------- |
| `transfer_id` | string | Unique UUID for tracking the transfer |
| `message`     | string | Human-readable status message         |

**Status Codes**:

* `202` - Transfer request accepted and queued for processing
* `400` - Invalid request parameters (non-stablecoin asset, invalid amount, etc.)
* `401` - Authentication failed (missing or invalid JWT/API key)
* `500` - Internal server error
* `503` - Transfer validation service temporarily unavailable

**Error Responses**:

**400 Bad Request** - Invalid parameters:

```json
{
  "error": "invalid_amount",
  "message": "Amount must be positive"
}
```

Possible error codes:

* `invalid_request_format` - Malformed JSON or missing required fields
* `invalid_amount` - Amount is zero, negative, or invalid decimal
* `invalid_source_type` - Source account type must be 'spot' or 'perps'
* `invalid_dest_type` - Destination account type must be 'spot' or 'perps'
* `same_account_type` - Source and destination types cannot be the same
* `invalid_asset` - Asset is not supported for spot-perps transfer (not a stablecoin, not active, or not found)
* `amount_below_minimum` - Amount is below minimum transfer amount for the asset

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

**Transfer Lifecycle**:

After a transfer is accepted:

1. The request is validated and accepted, returning `202 Accepted` with a `transfer_id`.
2. Funds are debited from the source account, then credited to the destination account.
3. When the transfer completes (or fails), the client is notified in real time on the `transfer_updates` WebSocket channel.

**Transfer States**:

| State              | Description                                              |
| ------------------ | -------------------------------------------------------- |
| `pending`          | Transfer initiated, waiting for source processing        |
| `source_completed` | Funds deducted from source account                       |
| `dest_completed`   | Funds added to destination account (final success state) |
| `failed`           | Transfer failed before any funds were moved              |

**WebSocket Notification**:

When the transfer completes (or fails), you receive a real-time notification:

```json
{
  "channel": "transfer_updates",
  "data": {
    "transfer_id": "550e8400-e29b-41d4-a716-446655440000",
    "owner_address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
    "app_session_id": "your-session-id",
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

* **Stablecoin Only**: Only stablecoin assets that are active on both the spot and perps sides can be transferred
* **Perp → Spot cap**: Outgoing perpetuals transfers are limited per request to 80% of `min(total - locked, available)` (see **Perpetual → Spot** above); plan `amount` accordingly or split into multiple transfers
* **Real-time Updates**: Subscribe to WebSocket `transfer_updates` channel for completion notifications. Balance update notifications are also pushed to `spot_account.balance_update` and `perpetuals_account.balance_update` channels.
* **Source and destination types must be different** (cannot transfer within the same account type)

**Example Request**:

```bash
curl -X POST https://trade.api.yellow.pro/accounts/transfer \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "app_session_id": "your-session-id",
    "source_account_type": "spot",
    "dest_account_type": "perps",
    "asset_symbol": "USDT",
    "amount": "1000.00"
  }'
```
