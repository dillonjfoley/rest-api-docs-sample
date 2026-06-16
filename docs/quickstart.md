# Quickstart

Use this guide to make your first request to the SensorHub Cloud API.

The SensorHub Cloud API lets you connect apps to SensorHub Cloud. You can use it to view sensors, check sensor readings, create alert rules, and review gateway status.

## Before you start

You need:

* A SensorHub Cloud account
* An API key
* A tool for sending API requests, such as:

  * Postman
  * curl
  * An API client in your code editor

## Base URL

Send all API requests to this base URL:

```text
https://api.sensorhub.example.com/v1
```

> Note: This is a fictional API URL for portfolio use.

## Step 1: Get an API key

To use the API, you need an API key.

In a real product, you would usually create an API key from your account settings.

For this sample, use this placeholder value:

```text
YOUR_API_KEY
```

Do not share real API keys in public repositories, support tickets, screenshots, or client-side code.

## Step 2: Add the authentication header

Each request must include an `Authorization` header.

Use this format:

```http
Authorization: Bearer YOUR_API_KEY
```

Example:

```http
Authorization: Bearer shc_test_123456789
```

## Step 3: Send your first request

Use the `GET /sensors` endpoint to list sensors in your account.

### Request

```bash
curl -X GET "https://api.sensorhub.example.com/v1/sensors" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Accept: application/json"
```

### Response

A successful response returns a list of sensors.

```json
{
  "data": [
    {
      "id": "sensor_12345",
      "name": "Walk-in Cooler Temperature",
      "type": "temperature",
      "status": "online",
      "gatewayId": "gateway_67890"
    },
    {
      "id": "sensor_23456",
      "name": "Storage Room Humidity",
      "type": "humidity",
      "status": "online",
      "gatewayId": "gateway_67890"
    }
  ]
}
```

## Step 4: Get readings for one sensor

After you have a sensor ID, you can request readings for that sensor.

This example gets readings for `sensor_12345`.

### Request

```bash
curl -X GET "https://api.sensorhub.example.com/v1/sensors/sensor_12345/readings" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Accept: application/json"
```

### Response

```json
{
  "sensorId": "sensor_12345",
  "data": [
    {
      "timestamp": "2026-06-16T14:25:00Z",
      "temperatureF": 72.4,
      "batteryPercent": 88,
      "signalStrength": "good"
    },
    {
      "timestamp": "2026-06-16T14:20:00Z",
      "temperatureF": 72.1,
      "batteryPercent": 88,
      "signalStrength": "good"
    }
  ]
}
```

## Step 5: Create an alert rule

Use the `POST /alerts` endpoint to create an alert rule.

This example creates an alert when a temperature sensor reads above 80°F.

### Request

```bash
curl -X POST "https://api.sensorhub.example.com/v1/alerts" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "sensorId": "sensor_12345",
    "name": "High temperature alert",
    "condition": {
      "field": "temperatureF",
      "operator": "greater_than",
      "value": 80
    },
    "notificationEmail": "facilities@example.com"
  }'
```

### Response

```json
{
  "id": "alert_98765",
  "sensorId": "sensor_12345",
  "name": "High temperature alert",
  "status": "enabled",
  "condition": {
    "field": "temperatureF",
    "operator": "greater_than",
    "value": 80
  },
  "notificationEmail": "facilities@example.com",
  "createdAt": "2026-06-16T14:30:00Z"
}
```

## Common errors

| Status code | Error code            | Meaning                                                |
| ----------- | --------------------- | ------------------------------------------------------ |
| `400`       | `invalid_request`     | The request is missing required information.           |
| `401`       | `unauthorized`        | The API key is missing or invalid.                     |
| `404`       | `not_found`           | The requested sensor, gateway, or alert was not found. |
| `429`       | `rate_limit_exceeded` | Too many requests were sent in a short time.           |
| `500`       | `server_error`        | The server could not complete the request.             |

For more information, see [Error codes](errors.md).

## Portfolio note

This quickstart is part of a fictional REST API documentation sample. It is designed to show API documentation structure, Markdown writing, request examples, response examples, and developer-focused task flow.
