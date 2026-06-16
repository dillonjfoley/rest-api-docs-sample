# Error Codes

Use this guide to understand SensorHub Cloud API errors and how to fix them.

When a request fails, the API returns an HTTP status code and a JSON error response.

## Error response format

Errors use this format:

```json
{
  "error": {
    "code": "invalid_request",
    "message": "The request is missing a required field."
  }
}
```

| Field           | Type   | Description                                                |
| --------------- | ------ | ---------------------------------------------------------- |
| `error`         | object | Contains error details.                                    |
| `error.code`    | string | A short error code you can use in logs or support tickets. |
| `error.message` | string | A plain-language message that explains what went wrong.    |

## Error code summary

| HTTP status | Error code            | Meaning                                                                        | What to do                                                     |
| ----------- | --------------------- | ------------------------------------------------------------------------------ | -------------------------------------------------------------- |
| `400`       | `invalid_request`     | The request is missing required data or has invalid values.                    | Check the request body, query parameters, and path parameters. |
| `401`       | `unauthorized`        | The API key is missing, invalid, expired, or disabled.                         | Add a valid API key to the `Authorization` header.             |
| `403`       | `forbidden`           | The API key is valid, but it does not have permission for this action.         | Use an API key with the correct permissions.                   |
| `404`       | `not_found`           | The requested sensor, gateway, alert, or reading was not found.                | Check that the ID in the request is correct.                   |
| `409`       | `conflict`            | The request conflicts with an existing resource.                               | Check whether the resource already exists or has changed.      |
| `422`       | `validation_failed`   | The request body is formatted correctly, but one or more fields are not valid. | Review the field errors and update the request.                |
| `429`       | `rate_limit_exceeded` | Too many requests were sent in a short time.                                   | Wait before sending more requests.                             |
| `500`       | `server_error`        | The server could not complete the request.                                     | Try again later.                                               |
| `503`       | `service_unavailable` | The service is temporarily unavailable.                                        | Wait and retry the request.                                    |

---

# 400 Bad Request

A `400 Bad Request` error means the API could not process the request because the request is missing required information or includes invalid data.

## Example causes

* A required query parameter is missing.
* A path parameter is formatted incorrectly.
* The request body is not valid JSON.
* A field uses the wrong data type.

## Example response

```json
{
  "error": {
    "code": "invalid_request",
    "message": "The request is missing the required sensorId field."
  }
}
```

## How to fix it

Check that:

* The request uses the correct endpoint.
* Required fields are included.
* JSON is formatted correctly.
* Query parameters use the expected names and values.

---

# 401 Unauthorized

A `401 Unauthorized` error means the request did not include a valid API key.

## Example causes

* The `Authorization` header is missing.
* The API key is incorrect.
* The API key has expired.
* The API key has been disabled.
* The header does not use the `Bearer` format.

## Correct header format

```http
Authorization: Bearer YOUR_API_KEY
```

## Example response

```json
{
  "error": {
    "code": "unauthorized",
    "message": "Authentication is required. Include a valid API key in the Authorization header."
  }
}
```

## How to fix it

Check that:

* The request includes the `Authorization` header.
* The header starts with `Bearer`.
* There is one space between `Bearer` and the API key.
* The API key is active.
* You are using the correct key for the correct environment.

For more information, see [Authentication](authentication.md).

---

# 403 Forbidden

A `403 Forbidden` error means the API key is valid, but it does not have permission to complete the request.

## Example causes

* A read-only API key is used to create, update, or delete a resource.
* The API key does not have access to the requested sensor or gateway.
* The account does not have access to the requested feature.

## Example response

```json
{
  "error": {
    "code": "forbidden",
    "message": "This API key does not have permission to create alert rules."
  }
}
```

## How to fix it

Check that:

* The API key has the right permissions.
* The requested resource belongs to your account.
* Your account includes access to the requested feature.

---

# 404 Not Found

A `404 Not Found` error means the requested resource does not exist or cannot be found.

## Example causes

* The sensor ID is incorrect.
* The gateway ID is incorrect.
* The alert rule was deleted.
* The resource belongs to another account.

## Example response

```json
{
  "error": {
    "code": "not_found",
    "message": "The requested sensor was not found."
  }
}
```

## How to fix it

Check that:

* The resource ID is spelled correctly.
* The resource has not been deleted.
* The resource belongs to the same account as the API key.
* You are using the correct environment.

---

# 409 Conflict

A `409 Conflict` error means the request conflicts with the current state of a resource.

## Example causes

* An alert rule with the same name already exists.
* A resource was updated by another user or system.
* The request tries to enable a rule that is already enabled.
* The request tries to delete a resource that is currently in use.

## Example response

```json
{
  "error": {
    "code": "conflict",
    "message": "An alert rule with this name already exists for the selected sensor."
  }
}
```

## How to fix it

Check that:

* The resource does not already exist.
* You are using the latest version of the resource.
* The requested action is allowed for the current resource state.

---

# 422 Validation Failed

A `422 Validation Failed` error means the API understood the request, but one or more fields failed validation.

## Example causes

* An email address is not valid.
* A number is outside the allowed range.
* A field uses an unsupported value.
* A required nested field is missing.

## Example response

```json
{
  "error": {
    "code": "validation_failed",
    "message": "One or more fields failed validation.",
    "fields": [
      {
        "field": "notificationEmail",
        "message": "Enter a valid email address."
      },
      {
        "field": "condition.operator",
        "message": "Use one of: greater_than, less_than, equals, not_equals."
      }
    ]
  }
}
```

## How to fix it

Check the `fields` array in the response. Each item shows which field failed and how to fix it.

---

# 429 Too Many Requests

A `429 Too Many Requests` error means the API key sent too many requests in a short time.

## Example causes

* The app is polling too often.
* A script is sending requests in a loop.
* Several users or services are using the same API key.
* Retry logic is sending requests too quickly.

## Example response

```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Too many requests. Try again later."
  }
}
```

## Retry-After header

When possible, the API includes a `Retry-After` header.

```http
Retry-After: 60
```

This means you should wait 60 seconds before sending another request.

## How to fix it

Check that your app:

* Waits before retrying requests.
* Uses the `Retry-After` header when it is available.
* Avoids sending the same request in a tight loop.
* Stores data when possible instead of requesting the same data again and again.

For more information, see [Rate limits](rate-limits.md).

---

# 500 Server Error

A `500 Server Error` means the server could not complete the request.

## Example causes

* The API had an unexpected problem.
* A backend service failed.
* The request could not be completed because of a temporary system issue.

## Example response

```json
{
  "error": {
    "code": "server_error",
    "message": "The server could not complete the request. Try again later."
  }
}
```

## How to fix it

Try the request again later.

If the error continues, collect the following information before contacting support:

* Endpoint
* HTTP method
* Request time
* Request ID, if available
* Error response
* Steps to reproduce the issue

---

# 503 Service Unavailable

A `503 Service Unavailable` error means the API is temporarily unavailable.

## Example causes

* Scheduled maintenance is in progress.
* A service is restarting.
* The API is receiving more traffic than expected.
* A dependent service is unavailable.

## Example response

```json
{
  "error": {
    "code": "service_unavailable",
    "message": "The service is temporarily unavailable. Try again later."
  }
}
```

## How to fix it

Wait and retry the request.

If the response includes a `Retry-After` header, wait for that amount of time before retrying.

---

# Best practices for handling errors

Use these practices when building an app with the SensorHub Cloud API.

## Show clear messages to users

Do not show raw error responses to end users.

Instead, show a short message that explains what happened.

Example:

```text
We could not load sensor readings right now. Try again in a few minutes.
```

## Log useful details

Log details that help you troubleshoot the issue.

Useful details include:

* HTTP status code
* Error code
* Endpoint
* Request time
* Request ID, if available

Do not log API keys, passwords, or private customer data.

## Retry only when it is safe

Some errors can be retried. Others need to be fixed before you try again.

| Status code | Retry?    | Notes                                                      |
| ----------- | --------- | ---------------------------------------------------------- |
| `400`       | No        | Fix the request first.                                     |
| `401`       | No        | Fix the API key first.                                     |
| `403`       | No        | Fix permissions first.                                     |
| `404`       | No        | Check the resource ID first.                               |
| `409`       | Sometimes | Check the resource state before retrying.                  |
| `422`       | No        | Fix the field values first.                                |
| `429`       | Yes       | Wait before retrying.                                      |
| `500`       | Yes       | Retry after a short delay.                                 |
| `503`       | Yes       | Retry after a short delay or use the `Retry-After` header. |

## Use backoff for retries

When retrying requests, do not retry right away in a tight loop.

Use a delay between retries. Increase the delay after each failed attempt.

Example retry pattern:

```text
First retry: wait 1 second
Second retry: wait 2 seconds
Third retry: wait 4 seconds
Fourth retry: wait 8 seconds
```

Stop retrying after a small number of attempts.

---

# Troubleshooting checklist

If you receive an error, check these items first:

* Is the endpoint correct?
* Is the HTTP method correct?
* Is the API key valid?
* Does the API key have the right permissions?
* Are all required fields included?
* Is the request body valid JSON?
* Are IDs spelled correctly?
* Are you using the correct base URL?
* Did you exceed the rate limit?

## Portfolio note

This error code guide is part of a fictional REST API documentation sample. It is designed to show API error documentation, troubleshooting guidance, retry behavior, and developer-focused support content.
