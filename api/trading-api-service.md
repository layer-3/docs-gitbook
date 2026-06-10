# Trading API Service

## Health & System Endpoints

### GET /health

Check if the trading API service is running and healthy.

{% hint style="info" %}
**Authentication:** Not required
{% endhint %}

**Response**:

```json
{
  "status": "UP"
}
```

**Status Codes**:

* `200` - Service is healthy
* `500` - Service is unhealthy

## Market Data

{% hint style="info" %}
Exchange information (markets, precisions, fees) is available per product at [`GET /spot/exchangeInfo`](spot-trading-api.md) and [`GET /perpetual/exchangeInfo`](perpetuals-trading-api.md).
{% endhint %}

### GET /orderbook

Retrieve order book data for a specific trading symbol.

{% hint style="info" %}
**Authentication:** Not required
{% endhint %}

**Query Parameters**:

| Parameter | Type   | Required | Options | Description                          |
| --------- | ------ | -------- | ------- | ------------------------------------ |
| `symbol`  | string | Yes      | —       | Trading symbol to get order book for |

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

| Field  | Type  | Description                                           |
| ------ | ----- | ----------------------------------------------------- |
| `bids` | array | Array of bid levels, each containing \[price, amount] |
| `asks` | array | Array of ask levels, each containing \[price, amount] |

**Status Codes**:

* `200` - Order book retrieved successfully
* `400` - Missing symbol parameter
* `404` - Symbol not found
* `500` - Internal server error

### GET /klines

Retrieve OHLC/candlestick data for a specific trading symbol. Compatible with Binance API format.

{% hint style="info" %}
**Authentication:** Not required
{% endhint %}

**Query Parameters**:

| Parameter   | Type    | Required | Options                                                                                     | Description                                 |
| ----------- | ------- | -------- | ------------------------------------------------------------------------------------------- | ------------------------------------------- |
| `symbol`    | string  | Yes      | —                                                                                           | Trading symbol to get klines for            |
| `interval`  | string  | No       | `1m`, `3m`, `5m`, `15m`, `30m`, `1h`, `2h`, `4h`, `6h`, `8h`, `12h`, `1d`, `3d`, `1w`, `1M` | Kline interval                              |
| `startTime` | integer | No       | —                                                                                           | Start time in milliseconds since Unix epoch |
| `endTime`   | integer | No       | —                                                                                           | End time in milliseconds since Unix epoch   |
| `limit`     | integer | No       | default `500`, max `1000`                                                                   | Number of klines to return                  |
| `timeZone`  | string  | No       | —                                                                                           | Timezone offset (e.g., "UTC", "+08:00")     |

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

* `[0]`: Open time (milliseconds)
* `[1]`: Open price
* `[2]`: High price
* `[3]`: Low price
* `[4]`: Close price
* `[5]`: Volume
* `[6]`: Number of trades

**Status Codes**:

* `200` - Klines retrieved successfully
* `400` - Invalid parameters or missing symbol
* `404` - Symbol not found
* `500` - Internal server error

### GET /ticker/24hr

Retrieve 24-hour ticker statistics for a specific trading symbol.

{% hint style="info" %}
**Authentication:** Not required
{% endhint %}

**Query Parameters**:

| Parameter | Type   | Required | Options | Description                      |
| --------- | ------ | -------- | ------- | -------------------------------- |
| `symbol`  | string | Yes      | —       | Trading symbol to get ticker for |

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

| Field         | Type           | Description                                |
| ------------- | -------------- | ------------------------------------------ |
| `marketId`    | string         | Market identifier (e.g., BTCUSD)           |
| `time`        | integer        | Timestamp of the data in Unix milliseconds |
| `min`         | decimal string | Lowest price in the period                 |
| `max`         | decimal string | Highest price in the period                |
| `first`       | decimal string | First price in the period (open price)     |
| `last`        | decimal string | Last trade price                           |
| `volume`      | decimal string | Total traded volume in the base currency   |
| `quoteVolume` | decimal string | Total traded volume in the quote currency  |
| `vwap`        | decimal string | Volume-weighted average price              |
| `priceChange` | string         | Price change percentage over the period    |

**Status Codes**:

* `200` - Ticker retrieved successfully
* `400` - Missing symbol parameter
* `404` - Symbol not found
* `500` - Internal server error

## Fees

### GET /account/fee-schedule

Retrieve the public fee tier schedule (maker/taker rates per tier, in basis points, and the volume thresholds that qualify for each tier).

{% hint style="info" %}
**Authentication:** Not required
{% endhint %}

**Response**:

```json
[
  {
    "tier": 0,
    "spot_maker_bps": "10",
    "spot_taker_bps": "15",
    "perp_maker_bps": "2",
    "perp_taker_bps": "5",
    "spot_volume_usd_min": "0",
    "perp_volume_usd_min": "0",
    "yellow_min": "0"
  }
]
```

**Response Fields** (array of tiers):

| Field                 | Type           | Description                                                     |
| --------------------- | -------------- | --------------------------------------------------------------- |
| `tier`                | integer        | Tier number (0 is the base tier)                                |
| `spot_maker_bps`      | decimal string | Spot maker fee, in basis points                                 |
| `spot_taker_bps`      | decimal string | Spot taker fee, in basis points                                 |
| `perp_maker_bps`      | decimal string | Perpetuals maker fee, in basis points                           |
| `perp_taker_bps`      | decimal string | Perpetuals taker fee, in basis points                           |
| `spot_volume_usd_min` | decimal string | Minimum 30-day spot volume (USD) to qualify for this tier       |
| `perp_volume_usd_min` | decimal string | Minimum 30-day perpetuals volume (USD) to qualify for this tier |
| `yellow_min`          | decimal string | Minimum $YELLOW balance to qualify for this tier                |

**Status Codes**:

* `200` - Fee schedule retrieved successfully
* `502` - Fee service error
* `503` - Fee service temporarily unavailable

### GET /account/fee-tier

Retrieve the authenticated user's current fee tier and the rates that apply to them.

{% hint style="info" %}
**Authentication:** Required (`read:spot` or `read:futures` scope)
{% endhint %}

**Response**:

```json
{
  "fee_tier": 2,
  "qualifying_track": "volume",
  "fee_tier_version": 3,
  "evaluated_at": "2026-06-10T00:00:00Z",
  "spot_maker_bps": "8",
  "spot_taker_bps": "12",
  "perp_maker_bps": "2",
  "perp_taker_bps": "4",
  "spot_30d_usd": "1250000",
  "perp_30d_usd": "5400000",
  "yellow_balance": "10000"
}
```

**Response Fields**:

| Field              | Type           | Description                                          |
| ------------------ | -------------- | ---------------------------------------------------- |
| `fee_tier`         | integer        | The user's current tier number                       |
| `qualifying_track` | string         | How the tier was reached (e.g. `volume` or `yellow`) |
| `fee_tier_version` | integer        | Version of the fee schedule used for evaluation      |
| `evaluated_at`     | string         | When the tier was last evaluated (RFC3339)           |
| `spot_maker_bps`   | decimal string | Effective spot maker fee, in basis points            |
| `spot_taker_bps`   | decimal string | Effective spot taker fee, in basis points            |
| `perp_maker_bps`   | decimal string | Effective perpetuals maker fee, in basis points      |
| `perp_taker_bps`   | decimal string | Effective perpetuals taker fee, in basis points      |
| `spot_30d_usd`     | decimal string | Trailing 30-day spot volume (USD)                    |
| `perp_30d_usd`     | decimal string | Trailing 30-day perpetuals volume (USD)              |
| `yellow_balance`   | decimal string | The user's $YELLOW balance used for tier evaluation  |

**Status Codes**:

* `200` - Fee tier retrieved successfully
* `401` - Authentication failed
* `403` - API key does not have the required scope
* `404` - No fee tier found for the user
* `503` - Fee service temporarily unavailable
