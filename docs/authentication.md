# Authentication

The SensorHub Cloud API uses API keys to authenticate requests.

An API key tells SensorHub Cloud which account is making the request. Each request must include a valid API key in the `Authorization` header.

## Authentication method

SensorHub Cloud uses bearer token authentication.

Send your API key as a bearer token in the request header.

```http
Authorization: Bearer YOUR_API_KEY
```

Replace `YOUR_API_KEY` with your actual API key.

## Example request

The following example lists all sensors in your account.

```bash
curl -X GET "https://api.sensorhub.example.com/v1/sensors" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Accept: application/json"
```

## Example successful response

If the API key is valid, the API returns the requested data.

```json
{
  "data": [
    {
      "id": "sensor_12345",
      "name": "Walk-in Cooler Temperature",
      "type": "temperature",
      "status": "online"
    }
  ]
}
```

## Missing API key

If you do not include an API key, the API returns a `401 Unauthorized` error.

### Request

```bash
curl -X GET "https://api.sensorhub.example.com/v1/sensors" \
  -H "Accept: application/json"
```

### Response

```json
{
  "error": {
    "code": "unauthorized",
    "message": "Authentication is required. Include a valid API key in the Authorization header."
  }
}
```

## Invalid API key

If the API key is wrong, expired, or disabled, the API returns a `401 Unauthorized` error.

### Request

```bash
curl -X GET "https://api.sensorhub.example.com/v1/sensors" \
  -H "Authorization: Bearer invalid_key" \
  -H "Accept: application/json"
```

### Response

```json
{
  "error": {
    "code": "unauthorized",
    "message": "The API key is invalid or has expired."
  }
}
```

## API key format

For this sample API, test API keys use this format:

```text
shc_test_123456789
```

Production API keys use this format:

```text
shc_live_123456789
```

Use test keys while building and testing your integration. Use live keys only when your integration is ready for production.

## Keep API keys private

API keys can give access to account data. Treat them like passwords.

Do:

* Store API keys in environment variables.
* Rotate keys if they are exposed.
* Use test keys during development.
* Limit who can create, view, or delete keys.

Do not:

* Commit API keys to GitHub.
* Share API keys in support tickets.
* Put API keys in screenshots.
* Store API keys in client-side code, such as browser JavaScript or mobile apps.

## Use environment variables

An environment variable lets your app use an API key without hard-coding it.

Example:

```bash
export SENSORHUB_API_KEY="shc_test_123456789"
```

Then use the variable in a request:

```bash
curl -X GET "https://api.sensorhub.example.com/v1/sensors" \
  -H "Authorization: Bearer $SENSORHUB_API_KEY" \
  -H "Accept: application/json"
```

## Rotate an API key

Rotate an API key if it is lost, exposed, or no longer needed.

A safe rotation process usually looks like this:

1. Create a new API key.
2. Update your app to use the new key.
3. Test the app.
4. Disable the old key.
5. Delete the old key after you confirm it is no longer used.

## Authentication errors

| HTTP status | Error code            | Meaning                                                                         | What to do                                          |
| ----------- | --------------------- | ------------------------------------------------------------------------------- | --------------------------------------------------- |
| `401`       | `unauthorized`        | The API key is missing, invalid, expired, or disabled.                          | Check the `Authorization` header and API key value. |
| `403`       | `forbidden`           | The API key is valid, but it does not have permission for the requested action. | Use a key with the correct permissions.             |
| `429`       | `rate_limit_exceeded` | The API key has sent too many requests in a short time.                         | Wait before sending more requests.                  |

For more information, see [Error codes](errors.md).

## Troubleshooting

### The request returns `401 Unauthorized`

Check that:

* The request includes the `Authorization` header.
* The header starts with `Bearer`.
* There is one space between `Bearer` and the API key.
* The API key has not expired or been disabled.
* You are using the correct key for the correct environment.

Correct format:

```http
Authorization: Bearer shc_test_123456789
```

Incorrect formats:

```http
Authorization: shc_test_123456789
```

```http
Authorization: Bearer
```

```http
Authorization: Bearer invalid_key
```

### The request returns `403 Forbidden`

A `403 Forbidden` error means the API key is valid, but it does not have access to the requested resource or action.

For example, a read-only key can list sensors, but it cannot create alert rules.

Use an API key with the correct permissions, or ask an account admin to update the key permissions.

## Portfolio note

This authentication guide is part of a fictional REST API documentation sample. It is designed to show secure API key documentation, clear request examples, error handling, and developer-focused troubleshooting.
