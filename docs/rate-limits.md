# Rate Limits

Use this guide to understand how SensorHub Cloud API rate limits work.

Rate limits help keep the API fast and reliable for all users. If your app sends too many requests in a short time, the API may slow or block extra requests for a short period.

## Default rate limit

The default rate limit is:

```text
600 requests per minute per API key
```

This means one API key can send up to 600 requests in one minute.

If your app sends more than 600 requests in one minute, the API returns a `429 Too Many Requests` error.

## How rate limits are counted

Rate limits are counted by API key.

For example, if three services use the same API key, their requests are counted together.

```text
Service A: 200 requests
Service B: 250 requests
Service C: 200 requests
Total:     650 requests
```

In this example, the API key goes over the 600-request limit.

To avoid this, use separate API keys for separate services when possible.

## Rate limit headers

Successful API responses may include rate limit headers.

```http
X-RateLimit-Limit: 600
X-RateLimit-Remaining: 452
X-RateLimit-Reset: 1718548800
```

| Header                  | Description                                                        |
| ----------------------- | ------------------------------------------------------------------ |
| `X-RateLimit-Limit`     | The maximum number of requests allowed in the current time window. |
| `X-RateLimit-Remaining` | The number of requests left in the current time window.            |
| `X-RateLimit-Reset`     | The time when the rate limit resets. This value uses Unix time.    |

## Example response headers

```http
HTTP/1.1 200 OK
Content-Type: application/json
X-RateLimit-Limit: 600
X-RateLimit-Remaining: 452
X-RateLimit-Reset: 1718548800
```

In this example:

* The API key can send 600 requests per minute.
* The API key has 452 requests left.
* The limit resets at the time shown in `X-RateLimit-Reset`.

## When you exceed the rate limit

If your app sends too many requests, the API returns:

```http
HTTP/1.1 429 Too Many Requests
```

The response body includes an error message.

```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Too many requests. Try again later."
  }
}
```

## Retry-After header

A `429` response may include a `Retry-After` header.

```http
Retry-After: 60
```

This means your app should wait 60 seconds before sending another request.

## Example 429 response

```http
HTTP/1.1 429 Too Many Requests
Content-Type: application/json
Retry-After: 60
X-RateLimit-Limit: 600
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1718548800
```

```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Too many requests. Try again in 60 seconds."
  }
}
```

## Recommended retry behavior

When your app receives a `429` response:

1. Stop sending requests for that API key.
2. Check the `Retry-After` header.
3. Wait for the amount of time shown in the header.
4. Retry the request.
5. If the request fails again, wait longer before trying again.

Do not retry the request right away in a loop.

## Use backoff for retries

Backoff means your app waits longer after each failed retry.

Example backoff pattern:

```text
First retry: wait 1 second
Second retry: wait 2 seconds
Third retry: wait 4 seconds
Fourth retry: wait 8 seconds
```

Stop retrying after a small number of attempts.

For most apps, 3 to 5 retries is enough.

## Best practices

Use these practices to stay within rate limits.

### Cache data when possible

Do not request the same data again and again if it has not changed.

For example, sensor names and gateway names usually do not change often. Store them in your app and refresh them only when needed.

### Use pagination

List endpoints may return many results. Use `limit` and `cursor` to request data in smaller groups.

Example:

```bash
curl -X GET "https://api.sensorhub.example.com/v1/sensors?limit=50" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Accept: application/json"
```

If the response includes a `nextCursor`, send another request with that cursor.

```bash
curl -X GET "https://api.sensorhub.example.com/v1/sensors?limit=50&cursor=eyJwYWdlIjoyfQ" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Accept: application/json"
```

### Avoid tight polling loops

Do not send the same request every few seconds unless your app truly needs near real-time data.

Instead, use a longer polling interval.

Recommended polling intervals:

| Use case             | Recommended interval               |
| -------------------- | ---------------------------------- |
| Dashboard display    | Every 1 to 5 minutes               |
| Background sync      | Every 5 to 15 minutes              |
| Historical reporting | Every 30 to 60 minutes             |
| Alert monitoring     | Use alert rules instead of polling |

### Use alert rules instead of polling

If you need to know when a sensor goes above or below a value, create an alert rule instead of checking the sensor again and again.

For example, instead of polling every 10 seconds to check if a cooler is too warm, create an alert rule:

```json
{
  "sensorId": "sensor_12345",
  "name": "High temperature alert",
  "condition": {
    "field": "temperatureF",
    "operator": "greater_than",
    "value": 80
  },
  "notificationEmail": "facilities@example.com"
}
```

### Separate development and production keys

Use different API keys for development and production.

This helps prevent test scripts from using the same rate limit as your live app.

Example:

```text
Development key: shc_test_123456789
Production key:  shc_live_123456789
```

### Monitor request volume

Track how many requests your app sends.

Useful things to log include:

* Endpoint
* HTTP method
* Response status
* Request time
* Rate limit headers
* Retry attempts

Do not log API keys or private customer data.

## Rate limit examples

### Example: Safe request volume

```text
API key limit: 600 requests per minute
App usage:     300 requests per minute
Result:        Requests continue normally
```

### Example: Too many requests

```text
API key limit: 600 requests per minute
App usage:     750 requests per minute
Result:        Extra requests return 429 errors
```

### Example: Shared API key

```text
API key limit: 600 requests per minute
Service A:     250 requests per minute
Service B:     250 requests per minute
Service C:     150 requests per minute
Total:         650 requests per minute
Result:        Some requests return 429 errors
```

## Troubleshooting

### My app gets frequent 429 errors

Check whether:

* Your app is polling too often.
* Multiple services use the same API key.
* Retry logic is sending requests too quickly.
* A script is stuck in a loop.
* The app is requesting more data than it needs.

### My app only gets 429 errors during busy times

Check whether request volume increases during:

* Store opening or closing
* Report generation
* Scheduled sync jobs
* Shift changes
* Batch imports

Spread large jobs across a longer time period when possible.

### My app retries but still fails

Check that your app:

* Reads the `Retry-After` header.
* Waits before retrying.
* Uses backoff between retries.
* Stops retrying after a set number of attempts.

## Related errors

| HTTP status | Error code            | Meaning                                      |
| ----------- | --------------------- | -------------------------------------------- |
| `429`       | `rate_limit_exceeded` | Too many requests were sent in a short time. |
| `500`       | `server_error`        | The server could not complete the request.   |
| `503`       | `service_unavailable` | The service is temporarily unavailable.      |

For more information, see [Error codes](errors.md).

## Portfolio note

This rate limits guide is part of a fictional REST API documentation sample. It is designed to show API behavior documentation, retry guidance, troubleshooting steps, and developer-focused best practices.
