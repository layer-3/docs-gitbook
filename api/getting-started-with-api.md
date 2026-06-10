# Getting Started with API

This guide walks you through authenticating with the Yellow API so you can start calling endpoints.

{% hint style="info" %}
All trading, account, and transfer endpoints require authentication. Market data endpoints are public and need no credentials.
{% endhint %}

For API base URLs, see [Base URLs](/broken/pages/d5e7080f2f3f4949887f3bed201f2ca3877b559a#base-urls).

## Authentication Flow

Yellow uses Ethereum wallet-based authentication with a challenge-response mechanism:

{% stepper %}
{% step %}
### Request a challenge

POST to `/auth/challenge` with your wallet address.
{% endstep %}

{% step %}
### Sign the challenge

Sign the returned challenge text with your Ethereum wallet.
{% endstep %}

{% step %}
### Verify the signature

POST to `/auth/verify` with your wallet address, the challenge, and the signature.
{% endstep %}

{% step %}
### Receive tokens

Get a JWT access token and a refresh token for API access.
{% endstep %}

{% step %}
### Use the access token

Include the JWT in the `Authorization` header on authenticated endpoints.

```http
Authorization: Bearer <your-jwt-token>
```
{% endstep %}
{% endstepper %}

See the [Authentication Service API](/broken/pages/578f9763d2d40b78e48c8f2facfb43abbfed2f2a) for full request and response details on each endpoint.

## Authentication Methods

All authenticated endpoints support two authentication methods.

{% tabs %}
{% tab title="JWT Token" %}
Obtained via the `/auth/verify` endpoint. Include the token in the `Authorization` header:

```http
Authorization: Bearer <token>
```

Supported by all authenticated endpoints.
{% endtab %}

{% tab title="API Key (HMAC)" %}
Best suited for programmatic / server-side access where you don't want to manage a wallet signing flow on every request. Each request includes three headers:

```http
X-API-KEY: <your-api-key>
X-TIMESTAMP: <Unix timestamp in seconds>
X-SIGNATURE: <HMAC-SHA256 signature of the request>
```

* Supported by all authenticated endpoints (spot, perpetuals, transfers)
* Automatically falls back to JWT if API key headers are not present

See [**Signing Requests with API Keys**](/broken/pages/bd0a002af558c9bbcc6292fe7c2b05b6876c7585) for the exact signature algorithm and ready-to-use Node.js and Python examples.

{% hint style="danger" %}
Never commit API keys or secrets to version control. Store them in environment variables or a secrets manager.
{% endhint %}
{% endtab %}
{% endtabs %}

## Security Notes

{% hint style="warning" %}
* Access tokens expire in 15 minutes
* Refresh tokens expire in 7 days
* Sessions can be invalidated via logout
* Rate limiting is enforced on authentication endpoints
{% endhint %}

## Next Steps

<a href="/broken/pages/ec147ad67071d832318aac1f934248e948963a53" class="button primary" data-icon="key">Create an API key</a>

| Section                            | Description                               | Target                                                                          |
| ---------------------------------- | ----------------------------------------- | ------------------------------------------------------------------------------- |
| **API Key Management**             | Create, list, and revoke API keys         | [api-keys.md](/broken/pages/ec147ad67071d832318aac1f934248e948963a53)           |
| **Signing Requests with API Keys** | HMAC-SHA256 signing with code examples    | [signing-requests.md](/broken/pages/bd0a002af558c9bbcc6292fe7c2b05b6876c7585)   |
| **Market Data API**                | Prices and indicators (no auth required)  | [market-data.md](/broken/pages/7bdd7f76fdbc648389cfe547c41d95e65d3b6d3a)        |
| **Spot Trading**                   | Place and manage spot orders              | [spot-trading.md](/broken/pages/d00fd8a1adf15936bec76105a8c1bbe57c4bb8f9)       |
| **Perpetuals Trading**             | Positions, leverage, and perpetual orders | [perpetuals-trading.md](/broken/pages/061f2de556a991ceddc49694e8fec148e2f4ed83) |
