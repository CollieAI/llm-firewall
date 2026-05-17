# Overview

This section documents every endpoint in the CollieAI API.

## Base URL

```
https://api.collieai.com
```

If you are self-hosting, replace this with your own deployment URL.

## Authentication

CollieAI uses two authentication methods depending on the endpoint:

| Endpoint prefix | Auth method            | Description                                                                           |
| --------------- | ---------------------- | ------------------------------------------------------------------------------------- |
| `/v1/...`       | API key (Bearer token) | Used by your application to proxy LLM requests and manage resources programmatically. |
| `/api/v1/...`   | Session cookie         | Used by the CollieAI dashboard. Requires a logged-in session.                         |

### API Key Authentication

Pass your CollieAI API key in the `Authorization` header:

```
Authorization: Bearer clai_your_api_key_here
```

API keys are prefixed with `clai_` and scoped to a single project. See [API Keys](/broken/pages/af397d4f5724bb034d6176f20a291fee89f336eb) for details.

### Session Authentication

Session cookies are set automatically when you sign in to the dashboard. No manual header is needed for browser-based requests.

## Error Format

All errors follow an OpenAI-compatible shape:

```json
{
  "error": {
    "message": "A human-readable description of what went wrong.",
    "type": "error_type",
    "code": 400
  }
}
```

### Common Error Types

| Type                    | Code | Description                                                      |
| ----------------------- | ---- | ---------------------------------------------------------------- |
| `invalid_request_error` | 400  | The request body is malformed or missing required fields.        |
| `policy_violation`      | 400  | A security rule with a **block** decision matched the content.   |
| `authentication_error`  | 401  | Missing, invalid, or expired API key or session.                 |
| `forbidden_error`       | 403  | The authenticated user does not have permission for this action. |
| `not_found_error`       | 404  | The requested resource does not exist.                           |
| `validation_error`      | 422  | One or more fields failed validation.                            |
| `rate_limit_exceeded`   | 429  | Too many requests. Back off and retry.                           |
| `provider_error`        | 502  | The upstream LLM provider returned an error.                     |
| `internal_error`        | 500  | An unexpected server error occurred.                             |

## Rate Limiting

API requests are subject to two types of rate limits:

* **Per-project burst limit (RPM):** Configurable requests-per-minute limit for each project. Returns `429` with `type: "rate_limit_exceeded"`.
* **Monthly usage limit:** Based on your subscription plan (Free: 15,000 calls/month, Growth: 2,000,000). Returns `429` with `type: "billing_limit"` when exceeded.

The `Retry-After` header indicates how many seconds to wait before retrying. Monthly usage includes a [grace buffer](/broken/pages/690fe8189012ef47ec9f7c93b23e72dc8ebd5663) -- requests are blocked at 110% of the limit, not exactly 100%.

## Pagination

List endpoints support pagination with two query parameters:

| Parameter   | Default | Description                             |
| ----------- | ------- | --------------------------------------- |
| `page`      | 1       | The page number (1-indexed).            |
| `page_size` | 20      | Number of items per page (maximum 100). |

Paginated responses include metadata:

```json
{
  "items": [],
  "total": 42,
  "page": 1,
  "page_size": 20
}
```

## Endpoints

### Proxy

* [Chat Completions](/broken/pages/10e149b1d3a68a8c26d5ad3e4b67b28639085a3a) -- OpenAI-compatible chat completions endpoint.

### Async

* [Jobs](/broken/pages/ca4386410c794e0df1d18268aca2cefd2dbed279) -- Create and manage async processing jobs.

### Resources

* [Projects](/broken/pages/bbc44658241fe75e4d26afbb312f7eac09b1d7d6) -- Create and manage projects.
* [Policies](/broken/pages/c7331b5a68d27b83ada251799a9411486e542d7e) -- Create and manage policies within a project.
* [Rules](/broken/pages/24ebabefd203dcea61f32cb65bd5de1682755dc5) -- Add, update, and remove rules from policies.
* [API Keys](/broken/pages/e5054042e1f2237394193ea801aee161c35f94f8) -- Generate and manage API keys for a project.
* [Provider Tokens](/broken/pages/f7bb3e820e7c3a4054ce2e60f704b32d932806cd) -- Store and manage LLM provider credentials.
* [Dictionaries](/broken/pages/121edafeb970f481758965b7e422723e5eea44ba) -- Upload and manage term dictionaries for matching rules.
* [Alerts](/broken/pages/637c07a43891d76e49a4616a60f72c50c618d124) -- Configure and manage monitoring alerts.

### Account

* [Auth](/broken/pages/aad9168cb0bfeb5c47849c9443ff77582fa67a24) -- Sign in, sign out, and manage sessions.
* [Health](/broken/pages/2dd9ec2c36fc50c6f523a13235f7f023029bb24a) -- Server health check.
* [Admin](/broken/pages/3240b33c4969c940758b489a4000870b131660d4) -- Administrative operations (user management, system resources).
* [Billing](/broken/pages/ccd5c37041370ab2b9c22dcedb8fe1f6461ed0dd) -- Subscription management, usage info, Stripe checkout and portal.
