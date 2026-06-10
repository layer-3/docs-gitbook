# Error Handling API

All endpoints return consistent error responses with the following structure:

## Authentication Service Errors

```json
{
  "error": "error_code",
  "message": "Human-readable error message",
  "code": 400
}
```

## Trading API Service Errors

```json
{
  "error": "error_code",
  "message": "Detailed error description"
}
```

## Common Error Codes

### Authentication Service

* `invalid_request` - Invalid request body format
* `invalid_wallet_address` - Invalid Ethereum wallet address
* `internal_error` - Internal server error
* `challenge_expired` - Challenge not found or expired
* `invalid_signature` - Signature verification failed
* `invalid_refresh_token` - Refresh token invalid or expired
* `missing_token` - Authorization header missing
* `invalid_token_format` - Invalid Authorization header format
* `unauthorized` - Authentication required

### Trading API Service

Order and leverage endpoints (`POST /spot/order`, `POST /perpetual/order`, `POST /perpetual/leverage`) may return a snake\_case `error` code describing why the request was rejected (for example, insufficient balance). When no specific code applies to a rejected fund reservation, `lock_funds_rejected` is returned.

#### General Trading API codes

* `missing_parameter` - Required parameter is missing
* `invalid_request_format` - Request body format is invalid
* `validation_failed` - Request validation failed (common on perpetual order and other endpoints)
* `order_creation_failed` - Spot order creation failed (message carries detail)
* `lock_funds_rejected` - The funds could not be reserved for this order (typically insufficient available balance)
* `missing_channel_id` - Channel ID parameter is required
* `missing_symbol` - Symbol parameter is required
* `symbol_not_found` - Requested symbol not found
* `missing_order_uuid` - Order UUID is required
* `invalid_order_uuid` - Order UUID format is invalid
* `internal_error` - Internal server error occurred

#### Spot order placement errors

* `service_unavailable` - Spot service is unavailable
* `invalid_amount` - Amount is invalid or non-positive
* `invalid_price` - Price is invalid
* `invalid_side` - Side is not buy/sell
* `no_permitted` - Caller is not the account owner
* `account_not_active` - Spot account is not active
* `account_not_exists` - Account not found
* `insufficient_balance` - Not enough available balance to reserve
* `market_price_unavailable` - No usable market price (e.g. market order) or reference price issue
* `market_not_found` - Unknown market symbol
* `unsupported_order_type` - Order type not supported for this request
* `missing_limit_price` - Limit order requires a non-zero price
* `limit_price_deviation_exceeded` - Limit price outside max deviation vs reference
* `fee_rate_unavailable` - Could not resolve maker/taker fee rate
* `lock_failed` - Could not reserve funds (see `message`)

#### Spot order cancellation errors

* `service_unavailable` - Spot service is unavailable
* `invalid_amount` - Amount is invalid, non-positive, or an invalid delta
* `insufficient_locked_balance` - Cannot release more than was reserved
* `unlock_failed` - Could not release funds (see `message`)

#### Perpetual order placement errors

* `service_unavailable` - Perpetuals service is unavailable
* `invalid_amount` - Amount is invalid, non-positive, or truncated to zero
* `invalid_leverage_value` - Request leverage not a number ≥ 1
* `leverage_exceeds_max` - Leverage above market maximum
* `invalid_side` - Side is not buy/sell
* `invalid_direction` - Direction not long or short
* `account_not_exists` - Account not found
* `no_permitted` - Caller is not the account owner
* `account_locked_for_liquidation` - Cross account locked for liquidation
* `fee_rate_unavailable` - Could not load maker/taker fee rate
* `no_position_to_close` - Closing order but no position
* `position_locked_for_liquidation` - Position locked for liquidation
* `insufficient_position` - Close size exceeds available position
* `mark_price_unavailable` - Mark price needed but unavailable
* `invalid_limit_price` - Limit price missing or non-positive
* `limit_price_deviation_exceeded` - The limit price is too far from the reference (mark) price
* `insufficient_margin_for_fees` - Not enough margin for closing fees
* `lock_position_failed` - Failed to lock position quantity
* `initial_margin_calculation_failed` - Initial margin computation failed
* `insufficient_margin` - Not enough margin for open (margin + fees)
* `lock_margin_failed` - Failed to reserve margin

#### Set leverage errors

* `missing_app_session_id` - `app_session_id` is empty
* `missing_market` - `market` is empty
* `missing_user_address` - `user_address` is empty
* `service_unavailable` - Perpetuals service is unavailable
* `invalid_leverage_value` - Leverage is not a number ≥ 1
* `leverage_exceeds_max` - Above market max leverage
* `account_not_exists` - Account not found
* `no_permitted` - Caller is not the account owner
* `account_not_active` - Account not active
* `open_orders_check_failed` - Could not verify open orders
* `order_exists` - Open orders exist for this market
* `insufficient_balance` - Not enough available collateral to apply higher leverage when positions require extra margin
* `leverage_exceeds_safe_margin` - New leverage would put allocated margin below the maintenance buffer for a position
* `set_leverage_failed` - Could not update leverage

## HTTP Status Codes

* `200` - Success
* `400` - Bad Request (client error)
* `401` - Unauthorized (authentication required/failed)
* `404` - Not Found (resource doesn't exist)
* `500` - Internal Server Error
