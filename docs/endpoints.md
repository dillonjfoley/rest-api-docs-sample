# Endpoint Reference

Use this reference to learn how to call the main SensorHub Cloud API endpoints.

The SensorHub Cloud API lets you work with sensors, readings, gateways, and alert rules.

## Base URL

Send all API requests to this base URL:

```text
https://api.sensorhub.example.com/v1
```

> Note: This is a fictional API URL for portfolio use.

## Authentication

All endpoints require an API key.

Include your API key in the `Authorization` header:

```http
Authorization: Bearer YOUR_API_KEY
```

For more information, see [Authentication](authentication.md).

## Content type

Requests that include a JSON body must use this header:

```http
Content-Type: application/json
```

Requests that return JSON should use this header:

```http
Accept: application/json
```

## Endpoint summary

| Method   | Endpoint                       | Description                      |
| -------- | ------------------------------ | -------------------------------- |
| `GET`    | `/sensors`                     | List sensors in your account.    |
| `GET`    | `/sensors/{sensorId}`          | Get details for one sensor.      |
| `GET`    | `/sensors/{sensorId}/readings` | Get readings for one sensor.     |
| `GET`    | `/gateways/{gatewayId}/status` | Check gateway connection status. |
| `GET`    | `/alerts`                      | List alert rules.                |
| `POST`   | `/alerts`                      | Create an alert rule.            |
| `PATCH`  | `/alerts/{alertId}`            | Update an alert rule.            |
| `DELETE` | `/alerts/{alertId}`            | Delete an alert rule.            |

---

# Sensors

## List sensors

Returns a list of sensors in your account.

```http
GET /sensors
```

### Query parameters

| Parameter | Type    | Required | Description                                                              |
| --------- | ------- | -------- | ------------------------------------------------------------------------ |
| `status`  | string  | No       | Filters sensors by status. Use `online`, `offline`, or `inactive`.       |
| `type`    | string  | No       | Filters sensors by type. For example, `temperature` or `humidity`.       |
| `limit`   | integer | No       | Sets the number of results to return. Default is `25`. Maximum is `100`. |
| `cursor`  | string  | No       | Returns the next page of results.                                        |

### Example request

```bash
curl -X GET "https://api.sensorhub.example.com/v1/sensors?status=online&type=temperature" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Accept: application/json"
```

### Example response

```json
{
  "data": [
    {
      "id": "sensor_12345",
      "name": "Walk-in Cooler Temperature",
      "type": "temperature",
      "status": "online",
      "gatewayId": "gateway_67890",
      "lastReadingAt": "2026-06-16T14:25:00Z"
    },
    {
      "id": "sensor_23456",
      "name": "Freezer Temperature",
      "type": "temperature",
      "status": "online",
      "gatewayId": "gateway_67890",
      "lastReadingAt": "2026-06-16T14:24:00Z"
    }
  ],
  "pagination": {
    "limit": 25,
    "nextCursor": null
  }
}
```

## Get a sensor

Returns details for one sensor.

```http
GET /sensors/{sensorId}
```

### Path parameters

| Parameter  | Type   | Required | Description                  |
| ---------- | ------ | -------- | ---------------------------- |
| `sensorId` | string | Yes      | The unique ID of the sensor. |

### Example request

```bash
curl -X GET "https://api.sensorhub.example.com/v1/sensors/sensor_12345" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Accept: application/json"
```

### Example response

```json
{
  "id": "sensor_12345",
  "name": "Walk-in Cooler Temperature",
  "type": "temperature",
  "status": "online",
  "gatewayId": "gateway_67890",
  "location": "Main Kitchen",
  "lastReading": {
    "timestamp": "2026-06-16T14:25:00Z",
    "temperatureF": 72.4,
    "batteryPercent": 88,
    "signalStrength": "good"
  }
}
```

---

# Sensor readings

## Get sensor readings

Returns recent readings for one sensor.

```http
GET /sensors/{sensorId}/readings
```

### Path parameters

| Parameter  | Type   | Required | Description                  |
| ---------- | ------ | -------- | ---------------------------- |
| `sensorId` | string | Yes      | The unique ID of the sensor. |

### Query parameters

| Parameter   | Type    | Required | Description                                                                |
| ----------- | ------- | -------- | -------------------------------------------------------------------------- |
| `startTime` | string  | No       | Start time for the reading range. Use ISO 8601 format.                     |
| `endTime`   | string  | No       | End time for the reading range. Use ISO 8601 format.                       |
| `limit`     | integer | No       | Sets the number of readings to return. Default is `100`. Maximum is `500`. |
| `cursor`    | string  | No       | Returns the next page of results.                                          |

### Example request

```bash
curl -X GET "https://api.sensorhub.example.com/v1/sensors/sensor_12345/readings?startTime=2026-06-16T14:00:00Z&endTime=2026-06-16T15:00:00Z" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Accept: application/json"
```

### Example response

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
  ],
  "pagination": {
    "limit": 100,
    "nextCursor": null
  }
}
```

---

# Gateways

## Get gateway status

Returns the current status for one gateway.

```http
GET /gateways/{gatewayId}/status
```

### Path parameters

| Parameter   | Type   | Required | Description                   |
| ----------- | ------ | -------- | ----------------------------- |
| `gatewayId` | string | Yes      | The unique ID of the gateway. |

### Example request

```bash
curl -X GET "https://api.sensorhub.example.com/v1/gateways/gateway_67890/status" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Accept: application/json"
```

### Example response

```json
{
  "gatewayId": "gateway_67890",
  "name": "Main Kitchen Gateway",
  "status": "online",
  "lastConnectedAt": "2026-06-16T14:27:00Z",
  "signalStrength": "excellent",
  "connectedSensors": 12
}
```

---

# Alerts

## List alert rules

Returns alert rules in your account.

```http
GET /alerts
```

### Query parameters

| Parameter  | Type    | Required | Description                                                              |
| ---------- | ------- | -------- | ------------------------------------------------------------------------ |
| `status`   | string  | No       | Filters alert rules by status. Use `enabled` or `disabled`.              |
| `sensorId` | string  | No       | Filters alert rules by sensor ID.                                        |
| `limit`    | integer | No       | Sets the number of results to return. Default is `25`. Maximum is `100`. |
| `cursor`   | string  | No       | Returns the next page of results.                                        |

### Example request

```bash
curl -X GET "https://api.sensorhub.example.com/v1/alerts?status=enabled" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Accept: application/json"
```

### Example response

```json
{
  "data": [
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
  ],
  "pagination": {
    "limit": 25,
    "nextCursor": null
  }
}
```

## Create an alert rule

Creates a new alert rule for a sensor.

```http
POST /alerts
```

### Request body

| Field                | Type   | Required | Description                                             |
| -------------------- | ------ | -------- | ------------------------------------------------------- |
| `sensorId`           | string | Yes      | The sensor ID the alert rule applies to.                |
| `name`               | string | Yes      | A clear name for the alert rule.                        |
| `condition`          | object | Yes      | The rule that triggers the alert.                       |
| `condition.field`    | string | Yes      | The sensor field to check. For example, `temperatureF`. |
| `condition.operator` | string | Yes      | The comparison to use.                                  |
| `condition.value`    | number | Yes      | The value that triggers the alert.                      |
| `notificationEmail`  | string | Yes      | The email address that receives the alert.              |

### Supported operators

| Operator       | Description                                              |
| -------------- | -------------------------------------------------------- |
| `greater_than` | Triggers when the reading is greater than the set value. |
| `less_than`    | Triggers when the reading is less than the set value.    |
| `equals`       | Triggers when the reading equals the set value.          |
| `not_equals`   | Triggers when the reading does not equal the set value.  |

### Example request

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

### Example response

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

## Update an alert rule

Updates an existing alert rule.

```http
PATCH /alerts/{alertId}
```

### Path parameters

| Parameter | Type   | Required | Description                      |
| --------- | ------ | -------- | -------------------------------- |
| `alertId` | string | Yes      | The unique ID of the alert rule. |

### Request body

You can update one or more of these fields:

| Field               | Type   | Required | Description                                    |
| ------------------- | ------ | -------- | ---------------------------------------------- |
| `name`              | string | No       | The alert rule name.                           |
| `status`            | string | No       | The alert status. Use `enabled` or `disabled`. |
| `condition`         | object | No       | The rule that triggers the alert.              |
| `notificationEmail` | string | No       | The email address that receives the alert.     |

### Example request

```bash
curl -X PATCH "https://api.sensorhub.example.com/v1/alerts/alert_98765" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "status": "disabled"
  }'
```

### Example response

```json
{
  "id": "alert_98765",
  "sensorId": "sensor_12345",
  "name": "High temperature alert",
  "status": "disabled",
  "condition": {
    "field": "temperatureF",
    "operator": "greater_than",
    "value": 80
  },
  "notificationEmail": "facilities@example.com",
  "updatedAt": "2026-06-16T15:10:00Z"
}
```

## Delete an alert rule

Deletes an alert rule.

```http
DELETE /alerts/{alertId}
```

### Path parameters

| Parameter | Type   | Required | Description                      |
| --------- | ------ | -------- | -------------------------------- |
| `alertId` | string | Yes      | The unique ID of the alert rule. |

### Example request

```bash
curl -X DELETE "https://api.sensorhub.example.com/v1/alerts/alert_98765" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Accept: application/json"
```

### Example response

A successful delete request returns `204 No Content`.

```http
HTTP/1.1 204 No Content
```

---

# Pagination

List endpoints may return many results. Use pagination to request results in smaller groups.

The response includes a `pagination` object:

```json
{
  "pagination": {
    "limit": 25,
    "nextCursor": "eyJwYWdlIjoyfQ"
  }
}
```

To get the next page, send another request with the `cursor` value:

```bash
curl -X GET "https://api.sensorhub.example.com/v1/sensors?cursor=eyJwYWdlIjoyfQ" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Accept: application/json"
```

If `nextCursor` is `null`, there are no more results.

---

# Error response format

When a request fails, the API returns an error object.

```json
{
  "error": {
    "code": "not_found",
    "message": "The requested sensor was not found."
  }
}
```

For more information, see [Error codes](errors.md).

---

## Portfolio note

This endpoint reference is part of a fictional REST API documentation sample. It is designed to show endpoint structure, request and response examples, parameter tables, pagination, and developer-focused API reference writing.
