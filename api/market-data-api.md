# Market Data API

These endpoints provide real-time market data for prices, technical indicators, and market analysis. Base URL: see [Base URLs](./#base-urls).

## Health Endpoints

### GET /health

Check if the market data service is running and healthy.

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

## Market Data Endpoints

### GET /data/price

Retrieve the latest price observation for all markets.

{% hint style="info" %}
**Authentication:** Not required
{% endhint %}

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

| Field              | Type   | Description                    |
| ------------------ | ------ | ------------------------------ |
| `header`           | object | Event metadata                 |
| `header.event_id`  | string | Unique event identifier        |
| `header.timestamp` | string | Event timestamp                |
| `market`           | string | Market symbol (e.g., "BTCUSD") |
| `price`            | string | Latest observed price          |

**Status Codes**:

* `200` - Latest price retrieved successfully
* `204` - No price data available
* `500` - Internal server error

### GET /data/momentum

Retrieve the latest momentum indicator data for all markets.

{% hint style="info" %}
**Authentication:** Not required
{% endhint %}

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

| Field          | Type   | Description                                                 |
| -------------- | ------ | ----------------------------------------------------------- |
| `Market`       | object | Market information                                          |
| `Market.Base`  | string | Base currency (e.g., "btc")                                 |
| `Market.Quote` | string | Quote currency (e.g., "usd")                                |
| `Value`        | string | Momentum indicator value (decimal)                          |
| `Time`         | string | Timestamp when the indicator was computed (ISO 8601 format) |

**Status Codes**:

* `200` - Latest momentum retrieved successfully
* `204` - No momentum data available
* `500` - Internal server error

### GET /data/std

Retrieve the latest moving standard deviation data for all markets.

{% hint style="info" %}
**Authentication:** Not required
{% endhint %}

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

| Field          | Type   | Description                                                 |
| -------------- | ------ | ----------------------------------------------------------- |
| `Market`       | object | Market information                                          |
| `Market.Base`  | string | Base currency (e.g., "btc")                                 |
| `Market.Quote` | string | Quote currency (e.g., "usd")                                |
| `Value`        | string | Moving standard deviation value (decimal)                   |
| `Time`         | string | Timestamp when the indicator was computed (ISO 8601 format) |

**Status Codes**:

* `200` - Latest standard deviation retrieved successfully
* `204` - No standard deviation data available
* `500` - Internal server error

### GET /data/vwma

Retrieve the latest volume-weighted moving average (VWMA) data for all markets.

{% hint style="info" %}
**Authentication:** Not required
{% endhint %}

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

| Field          | Type   | Description                                                 |
| -------------- | ------ | ----------------------------------------------------------- |
| `Market`       | object | Market information                                          |
| `Market.Base`  | string | Base currency (e.g., "btc")                                 |
| `Market.Quote` | string | Quote currency (e.g., "usd")                                |
| `Value`        | string | Volume-weighted moving average value (decimal)              |
| `Time`         | string | Timestamp when the indicator was computed (ISO 8601 format) |

**Status Codes**:

* `200` - Latest VWMA retrieved successfully
* `204` - No VWMA data available
* `500` - Internal server error
