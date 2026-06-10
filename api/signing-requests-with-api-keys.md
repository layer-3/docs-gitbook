# Signing Requests With API Keys

API key authentication lets you call authenticated endpoints from servers and scripts without the interactive wallet signing flow. Every authenticated request carries three headers, and the signature is recomputed per request from the request contents.

{% hint style="danger" %}
Your API secret is used to compute signatures and is **never sent** in a request. Never commit it to version control, log it, or expose it in client-side code. Store it in an environment variable or a secrets manager.
{% endhint %}

## Required headers

| Header        | Value                                           |
| ------------- | ----------------------------------------------- |
| `X-API-KEY`   | Your API key                                    |
| `X-TIMESTAMP` | Request time as a **Unix timestamp in seconds** |
| `X-SIGNATURE` | Hex-encoded HMAC-SHA256 signature (see below)   |

## How the signature is computed

The signature is `HMAC-SHA256(secret, prehash)`, hex-encoded, where:

```
prehash = METHOD + PATH + TIMESTAMP + CANONICAL
```

| Part        | Description                                                              |
| ----------- | ------------------------------------------------------------------------ |
| `METHOD`    | Uppercase HTTP method, e.g. `GET`, `POST`, `DELETE`                      |
| `PATH`      | Request path only, **without** the query string, e.g. `/perpetual/order` |
| `TIMESTAMP` | The same value sent in `X-TIMESTAMP` (Unix seconds)                      |
| `CANONICAL` | The canonical string built from the request parameters (below)           |

### Building the canonical string

The canonical string is built from sorted parameters joined into `key=value` pairs with a `|` separator. Where the parameters come from depends on the method:

{% tabs %}
{% tab title="POST / PUT / PATCH" %}
Use the **JSON request body**. Sort its top-level keys alphabetically and join them:

```
key1=value1|key2=value2|...
```

For a body of `{"market":"BTCUSDT","side":"buy","type":"limit","amount":"0.5","price":"35000"}`:

```
amount=0.5|market=BTCUSDT|price=35000|side=buy|type=limit
```

{% hint style="info" %}
Send decimal values (`amount`, `price`, …) as **strings**, exactly as they appear in the body you transmit. This keeps the signed value identical to the sent value and avoids floating-point formatting differences.
{% endhint %}
{% endtab %}

{% tab title="GET / DELETE" %}
Use the **query parameters**. Sort the parameter names alphabetically and join them. If a parameter appears multiple times, sort its values and join them with commas:

```
key1=value1|key2=value2|...
```

For `GET /perpetual/orders?market=BTCUSDT&app_session_id=your-session-id`:

```
app_session_id=your-session-id|market=BTCUSDT
```

A request with no query parameters has an **empty** canonical string.
{% endtab %}
{% endtabs %}

### Worked example

For `POST /perpetual/order` at timestamp `1701938200` with the body above:

```
prehash   = POST/perpetual/order1701938200amount=0.5|market=BTCUSDT|price=35000|side=buy|type=limit
signature = hex( HMAC_SHA256(your_api_secret, prehash) )
```

The request then carries:

```http
X-API-KEY: <your-api-key>
X-TIMESTAMP: 1701938200
X-SIGNATURE: <signature>
Content-Type: application/json
```

## Code examples

{% tabs %}
{% tab title="Node.js" %}
```javascript
const crypto = require('crypto');

function buildAuthHeaders({ method, path, query = {}, body = null, apiKey, apiSecret }) {
  const m = method.toUpperCase();
  const timestamp = Math.floor(Date.now() / 1000).toString(); // Unix seconds

  const canonical = (m === 'GET' || m === 'DELETE')
    ? canonicalFromQuery(query)
    : canonicalFromBody(body || {});

  const prehash = m + path + timestamp + canonical;
  const signature = crypto.createHmac('sha256', apiSecret).update(prehash).digest('hex');

  return {
    'X-API-KEY': apiKey,
    'X-TIMESTAMP': timestamp,
    'X-SIGNATURE': signature,
    'Content-Type': 'application/json',
  };
}

function canonicalFromBody(body) {
  return Object.keys(body).sort().map((k) => `${k}=${body[k]}`).join('|');
}

function canonicalFromQuery(query) {
  return Object.keys(query).sort().map((k) => {
    const v = query[k];
    const value = Array.isArray(v) ? [...v].sort().join(',') : v;
    return `${k}=${value}`;
  }).join('|');
}

// Example: place a perpetual order
const body = { market: 'BTCUSDT', side: 'buy', type: 'limit', amount: '0.5', price: '35000' };
const headers = buildAuthHeaders({
  method: 'POST',
  path: '/perpetual/order',
  body,
  apiKey: process.env.API_KEY,
  apiSecret: process.env.API_SECRET,
});

await fetch('https://trade.api.yellow.pro/perpetual/order', {
  method: 'POST',
  headers,
  body: JSON.stringify(body), // must match the body used for the signature
});
```
{% endtab %}

{% tab title="Python" %}
```python
import hashlib
import hmac
import json
import time
import requests

def build_auth_headers(method, path, *, api_key, api_secret, query=None, body=None):
    m = method.upper()
    timestamp = str(int(time.time()))  # Unix seconds

    if m in ("GET", "DELETE"):
        canonical = canonical_from_query(query or {})
    else:
        canonical = canonical_from_body(body or {})

    prehash = f"{m}{path}{timestamp}{canonical}"
    signature = hmac.new(
        api_secret.encode(), prehash.encode(), hashlib.sha256
    ).hexdigest()

    return {
        "X-API-KEY": api_key,
        "X-TIMESTAMP": timestamp,
        "X-SIGNATURE": signature,
        "Content-Type": "application/json",
    }

def canonical_from_body(body):
    return "|".join(f"{k}={body[k]}" for k in sorted(body))

def canonical_from_query(query):
    parts = []
    for k in sorted(query):
        v = query[k]
        if isinstance(v, (list, tuple)):
            v = ",".join(sorted(map(str, v)))
        parts.append(f"{k}={v}")
    return "|".join(parts)

# Example: place a perpetual order
body = {"market": "BTCUSDT", "side": "buy", "type": "limit", "amount": "0.5", "price": "35000"}
headers = build_auth_headers("POST", "/perpetual/order", api_key=API_KEY, api_secret=API_SECRET, body=body)

requests.post(
    "https://trade.api.yellow.pro/perpetual/order",
    headers=headers,
    data=json.dumps(body),  # must match the body used for the signature
)
```
{% endtab %}
{% endtabs %}

## Common pitfalls

{% hint style="warning" %}
* **Sign the body you send.** The JSON you transmit must contain exactly the values used to build the canonical string. Re-serializing or reordering after signing will invalidate the signature.
* **Seconds, not milliseconds.** `X-TIMESTAMP` is Unix **seconds**. Sending milliseconds will fail verification.
* **Path only.** Exclude the query string from `PATH`; query parameters belong in the canonical string for `GET`/`DELETE`.
* **Sort keys.** Both body keys and query keys must be sorted alphabetically before joining.
{% endhint %}
