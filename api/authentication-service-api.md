# Authentication Service API

## Health Endpoints

### GET /health

Check if the authentication service is running and healthy.

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

## Authentication Endpoints

### POST /auth/challenge

Generate a unique authentication challenge for wallet signature verification.

{% hint style="info" %}
**Authentication:** Not required
{% endhint %}

**Request Body**:

```json
{
  "wallet_address": "0x1234567890abcdef1234567890abcdef12345678"
}
```

**Request Parameters**:

| Parameter        | Type   | Required | Options                    | Description             |
| ---------------- | ------ | -------- | -------------------------- | ----------------------- |
| `wallet_address` | string | Yes      | 0x-prefixed, 42 characters | Ethereum wallet address |

**Response**:

```json
{
  "challenge": "Yellow Challenge: 1234567890abcdef, Timestamp: 2023-12-07T10:30:00Z, Nonce: 1234567890abcdef1234567890abcdef",
  "expires_at": "2023-12-07T10:35:00Z"
}
```

**Response Fields**:

| Field        | Type   | Description                                             |
| ------------ | ------ | ------------------------------------------------------- |
| `challenge`  | string | The challenge text that must be signed with your wallet |
| `expires_at` | string | When the challenge expires (ISO 8601 format)            |

**Status Codes**:

* `200` - Challenge generated successfully
* `400` - Invalid wallet address or request format
* `429` - Rate limit exceeded
* `500` - Internal server error

### POST /auth/verify

Verify the signature of a challenge and issue JWT tokens.

{% hint style="info" %}
**Authentication:** Not required
{% endhint %}

**Request Body**:

```json
{
  "wallet_address": "0x1234567890abcdef1234567890abcdef12345678",
  "challenge": "Yellow Challenge: 1234567890abcdef, Timestamp: 2023-12-07T10:30:00Z, Nonce: 1234567890abcdef1234567890abcdef",
  "signature": "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef12"
}
```

**Request Parameters**:

| Parameter        | Type   | Required | Options                     | Description                            |
| ---------------- | ------ | -------- | --------------------------- | -------------------------------------- |
| `wallet_address` | string | Yes      | —                           | Same wallet address used for challenge |
| `challenge`      | string | Yes      | —                           | The challenge text that was generated  |
| `signature`      | string | Yes      | 0x-prefixed, 132 characters | Ethereum signature of the challenge    |

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

| Field           | Type    | Description                           |
| --------------- | ------- | ------------------------------------- |
| `access_token`  | string  | JWT token for API authentication      |
| `refresh_token` | string  | Token for refreshing the access token |
| `expires_in`    | integer | Access token lifetime in seconds      |
| `token_type`    | string  | Always "Bearer"                       |

**Status Codes**:

* `200` - Signature verified successfully, tokens issued
* `400` - Invalid request format
* `401` - Invalid signature or expired challenge
* `429` - Rate limit exceeded
* `500` - Internal server error

### POST /auth/refresh

Generate a new access token using a valid refresh token.

{% hint style="info" %}
**Authentication:** Not required
{% endhint %}

**Request Body**:

```json
{
  "refresh_token": "rt_1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef"
}
```

**Request Parameters**:

| Parameter       | Type   | Required | Options                      | Description         |
| --------------- | ------ | -------- | ---------------------------- | ------------------- |
| `refresh_token` | string | Yes      | rt\_ prefixed, 67 characters | Valid refresh token |

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

* `200` - Token refreshed successfully
* `400` - Invalid request format
* `401` - Invalid or expired refresh token
* `429` - Rate limit exceeded (20 requests per minute)
* `500` - Internal server error

### POST /auth/logout

Invalidate the current session by blacklisting it and revoking all associated refresh tokens.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

**Response**:

```json
{
  "message": "Successfully logged out"
}
```

**Status Codes**:

* `200` - Successfully logged out
* `401` - Missing or invalid access token
* `429` - Rate limit exceeded
* `500` - Internal server error

### GET /auth/me

Retrieve information about the authenticated user from their JWT token.

{% hint style="info" %}
**Authentication:** Required
{% endhint %}

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

| Field            | Type    | Description                             |
| ---------------- | ------- | --------------------------------------- |
| `wallet_address` | string  | User's Ethereum wallet address          |
| `session_id`     | string  | Current session identifier              |
| `token_id`       | string  | Current token identifier (JTI)          |
| `issued_at`      | integer | Token issued timestamp (Unix epoch)     |
| `expires_at`     | integer | Token expiration timestamp (Unix epoch) |

**Status Codes**:

* `200` - User information retrieved successfully
* `401` - Missing or invalid access token
* `429` - Rate limit exceeded
* `500` - Internal server error
