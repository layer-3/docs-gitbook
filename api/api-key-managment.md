# API Key Managment

API keys let you call authenticated endpoints from servers and scripts. You create and manage keys through these endpoints, then sign each request as described in [Signing Requests with API Keys](/broken/pages/bd0a002af558c9bbcc6292fe7c2b05b6876c7585).

{% hint style="warning" %}
API key management endpoints require **JWT authentication** — obtain a JWT first via the [Authentication Service](/broken/pages/578f9763d2d40b78e48c8f2facfb43abbfed2f2a). You cannot manage API keys using API key headers; doing so returns `403 forbidden_auth_method`.
{% endhint %}

## Scopes

Each key is granted one or more scopes. Requests signed with a key are limited to the actions its scopes allow.

| Scope           | Grants                                                    |
| --------------- | --------------------------------------------------------- |
| `read:spot`     | Read spot accounts, orders, trades, deposits, withdrawals |
| `trade:spot`    | Place and cancel spot orders                              |
| `withdraw:spot` | Request spot withdrawals                                  |
| `read:futures`  | Read perpetuals accounts, positions, orders, trades       |
| `trade:futures` | Place and cancel perpetuals orders, manage positions      |

{% hint style="info" %}
Grant the narrowest set of scopes a key needs. A read-only data key should not carry `trade:*` or `withdraw:*` scopes.
{% endhint %}

## GET /accounts/api-keys

List the API keys belonging to the authenticated account.

{% hint style="info" %}
**Authentication:** Required (JWT only)
{% endhint %}

**Query Parameters**:

| Parameter   | Type    | Required | Options                  | Description    |
| ----------- | ------- | -------- | ------------------------ | -------------- |
| `page`      | integer | No       | default `1`              | Page number    |
| `page_size` | integer | No       | capped at server maximum | Items per page |

**Response**:

```json
{
  "keys": [
    {
      "id": "key_550e8400e29b41d4",
      "api_key": "ak_live_1234567890abcdef",
      "scopes": ["read:spot", "trade:spot"],
      "ip_whitelist": ["203.0.113.10"],
      "state": "active",
      "expires_at": "2027-01-01T00:00:00Z",
      "last_used_at": "2026-06-10T09:30:00Z",
      "created_at": "2026-06-01T12:00:00Z",
      "description": "trading bot"
    }
  ],
  "total": 1,
  "page": 1,
  "page_size": 50
}
```

**Response Fields**:

| Field                 | Type    | Description                                               |
| --------------------- | ------- | --------------------------------------------------------- |
| `keys`                | array   | Array of API key records                                  |
| `keys[].id`           | string  | Internal key identifier (used to revoke the key)          |
| `keys[].api_key`      | string  | The public API key value (sent in the `X-API-KEY` header) |
| `keys[].scopes`       | array   | Granted scopes                                            |
| `keys[].ip_whitelist` | array   | Allowed source IPs (empty means no IP restriction)        |
| `keys[].state`        | string  | Key state (e.g. `active`, `revoked`)                      |
| `keys[].expires_at`   | string  | Expiry timestamp, or empty if the key does not expire     |
| `keys[].last_used_at` | string  | Last time the key was used, or empty if never used        |
| `keys[].created_at`   | string  | Creation timestamp                                        |
| `keys[].description`  | string  | User-supplied label                                       |
| `total`               | integer | Total number of keys                                      |
| `page`                | integer | Pagination echo                                           |
| `page_size`           | integer | Pagination echo                                           |

**Status Codes**:

* `200` - Keys retrieved successfully
* `401` - Authentication failed
* `403` - Request was not authenticated with a JWT (`forbidden_auth_method`)

## POST /accounts/api-keys

Create a new API key. The **secret is returned only once**, in this response — store it immediately; it cannot be retrieved later.

{% hint style="info" %}
**Authentication:** Required (JWT only)
{% endhint %}

**Request Body**:

```json
{
  "scopes": ["read:spot", "trade:spot"],
  "ip_whitelist": ["203.0.113.10"],
  "expires_at": "2027-01-01T00:00:00Z",
  "description": "trading bot"
}
```

**Request Parameters**:

| Parameter      | Type   | Required | Options | Description                                                                   |
| -------------- | ------ | -------- | ------- | ----------------------------------------------------------------------------- |
| `scopes`       | array  | Yes      | —       | Scopes to grant (see [Scopes](api-key-managment.md#scopes))                   |
| `ip_whitelist` | array  | No       | —       | Source IPs allowed to use the key. Omit or leave empty for no IP restriction. |
| `expires_at`   | string | No       | RFC3339 | Expiry timestamp. Omit for a non-expiring key.                                |
| `description`  | string | No       | —       | A label to help you identify the key                                          |

**Response** (`201 Created`):

```json
{
  "key": {
    "id": "key_550e8400e29b41d4",
    "api_key": "ak_live_1234567890abcdef",
    "scopes": ["read:spot", "trade:spot"],
    "ip_whitelist": ["203.0.113.10"],
    "state": "active",
    "expires_at": "2027-01-01T00:00:00Z",
    "last_used_at": "",
    "created_at": "2026-06-10T12:00:00Z",
    "description": "trading bot"
  },
  "secret": "sk_live_abcdef1234567890abcdef1234567890"
}
```

**Response Fields**:

| Field    | Type   | Description                                                              |
| -------- | ------ | ------------------------------------------------------------------------ |
| `key`    | object | The created key record (same shape as in the list response)              |
| `secret` | string | The API secret, used to compute request signatures. **Shown only once.** |

{% hint style="danger" %}
Store the `secret` securely the moment you receive it. It is never returned again, and it must never be committed to version control or exposed in client-side code.
{% endhint %}

**Status Codes**:

* `201` - Key created successfully
* `400` - Invalid request body or invalid scope/parameter
* `401` - Authentication failed
* `403` - Request was not authenticated with a JWT (`forbidden_auth_method`)
* `500` - Internal server error

## POST /accounts/api-keys/:id/revoke

Revoke an API key by its `id`. Revocation is immediate and permanent; the key can no longer be used to sign requests.

{% hint style="info" %}
**Authentication:** Required (JWT only)
{% endhint %}

**Path Parameters**:

| Parameter | Type   | Required | Options | Description                                            |
| --------- | ------ | -------- | ------- | ------------------------------------------------------ |
| `id`      | string | Yes      | —       | The `id` of the key to revoke (from the list response) |

**Response**:

```json
{
  "message": "API key revoked"
}
```

**Status Codes**:

* `200` - Key revoked successfully
* `401` - Authentication failed
* `403` - Request was not authenticated with a JWT (`forbidden_auth_method`)
* `404` - Key not found or not owned by the authenticated account (`api_key_not_found`)

## Next step

Once you have an API key and secret, see [Signing Requests with API Keys](/broken/pages/bd0a002af558c9bbcc6292fe7c2b05b6876c7585) to authenticate your requests.
