# Reference

## Rate Limiting

Rate limiting is enforced to ensure fair usage:

* Rate limits apply per endpoint and account
* Rate limit information is included in response headers
* Exceeding a limit returns a `429` response

***

## Data Types

### Decimal Values

All monetary and quantity values are represented as strings in decimal format to maintain precision:

```json
{
  "amount": "1.50000000",
  "price": "35000.00000000"
}
```

### Timestamps

Timestamps are provided in milliseconds since Unix epoch:

```json
{
  "server_time": 1640995200000
}
```

### UUIDs

Order UUIDs follow standard UUID format:

```json
{
  "order_uuid": "550e8400-e29b-41d4-a716-446655440000"
}
```

***

## Pagination

List endpoints use **cursor-based** pagination. You walk through results by following the `next_cursor` returned with each page:

{% stepper %}
{% step %}
**First page** — send `use_cursor=true` (and optionally `page_size`).
{% endstep %}

{% step %}
Read `next_cursor` and `has_more` from the response.
{% endstep %}

{% step %}
**Next page** — send `cursor=<next_cursor>` from the previous response.
{% endstep %}

{% step %}
Repeat while `has_more` is `true`.
{% endstep %}
{% endstepper %}

**Query parameters**

| Parameter    | Type    | Description                                                                                   |
| ------------ | ------- | --------------------------------------------------------------------------------------------- |
| `use_cursor` | boolean | Set `true` to use cursor pagination. Send it on the first request.                            |
| `cursor`     | string  | Opaque continuation token from the previous response's `next_cursor`. Omit on the first page. |
| `page_size`  | integer | Items per page. Default `50`, maximum `100`.                                                  |

**Response fields**

| Field         | Type    | Description                                                                      |
| ------------- | ------- | -------------------------------------------------------------------------------- |
| `next_cursor` | string  | Token to pass as `cursor` for the next page. Empty when there are no more pages. |
| `has_more`    | boolean | Whether another page exists.                                                     |
| `page_size`   | integer | Items per page.                                                                  |

{% hint style="info" %}
Always treat `cursor` values as opaque — do not parse or construct them. A malformed `cursor` returns `400` with `"error": "invalid_cursor"`.
{% endhint %}
