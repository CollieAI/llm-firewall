# Jobs

Asynchronous request processing. Create a job to evaluate content against policy rules, with optional LLM forwarding and webhook delivery.

***

## POST /v1/jobs

Create a new async job.

**URL:** `https://api.collieai.com/v1/jobs`

**Auth:** Bearer token (API key `clai_xxx`)

### Request Body

| Field                | Type    | Required | Description                                                                                   |
| -------------------- | ------- | -------- | --------------------------------------------------------------------------------------------- |
| `message`            | string  | No       | Shorthand for a single user message. Mutually exclusive with `message_input`/`message_output` |
| `message_input`      | string  | No       | Inbound message to evaluate against inbound rules                                             |
| `message_output`     | string  | No       | Outbound message to evaluate against outbound rules                                           |
| `webhook_url`        | string  | Yes      | URL to receive the job result via POST                                                        |
| `metadata`           | object  | No       | Arbitrary key-value pairs attached to the job                                                 |
| `expires_in_seconds` | integer | No       | Job TTL in seconds. Default: 86400 (24h)                                                      |
| `inbound_only`       | boolean | No       | Only run inbound rules, skip LLM forwarding. Default: `false`                                 |

### Field Behavior

| Scenario                           | Inbound Rules | LLM Call | Outbound Rules            |
| ---------------------------------- | ------------- | -------- | ------------------------- |
| `message` only                     | Evaluated     | Yes      | Evaluated on LLM response |
| `message_input` only               | Evaluated     | Yes      | Evaluated on LLM response |
| `message_output` only              | Skipped       | No       | Evaluated                 |
| `message_input` + `message_output` | Evaluated     | No       | Evaluated                 |
| `inbound_only: true` + `message`   | Evaluated     | No       | Skipped                   |

### Example Request

```bash
curl -X POST https://api.collieai.com/v1/jobs \
  -H "Authorization: Bearer clai_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "message_input": "Summarize the quarterly earnings report",
    "webhook_url": "https://your-app.com/webhooks/collie",
    "metadata": {
      "user_id": "user_123",
      "session_id": "sess_abc"
    }
  }'
```

### Response -- `202 Accepted`

```json
{
  "job_id": "job_abc123def456",
  "status": "pending",
  "webhook_secret": "whsec_abc123",
  "created_at": "2025-01-15T10:30:00Z",
  "expires_at": "2025-01-16T10:30:00Z"
}
```

### Error Responses

| Status | Description                                                        |
| ------ | ------------------------------------------------------------------ |
| `400`  | Invalid request (e.g. both `message` and `message_input` provided) |
| `401`  | Missing or invalid API key                                         |
| `422`  | Validation error                                                   |

***

## GET /v1/jobs/{job\_id}

Retrieve job status and results.

**URL:** `https://api.collieai.com/v1/jobs/{job_id}`

**Auth:** Bearer token (API key `clai_xxx`)

### Path Parameters

| Parameter | Type   | Description        |
| --------- | ------ | ------------------ |
| `job_id`  | string | The job identifier |

### Example Request

```bash
curl https://api.collieai.com/v1/jobs/job_abc123def456 \
  -H "Authorization: Bearer clai_your_api_key"
```

### Response -- `200 OK`

```json
{
  "job_id": "job_abc123def456",
  "status": "completed",
  "created_at": "2025-01-15T10:30:00Z",
  "completed_at": "2025-01-15T10:30:05Z",
  "expires_at": "2025-01-16T10:30:00Z",
  "metadata": {
    "user_id": "user_123",
    "session_id": "sess_abc"
  },
  "inbound_result": {
    "decision": "pass",
    "rules_evaluated": 3,
    "rules_triggered": [],
    "latency_ms": 120
  },
  "outbound_result": {
    "decision": "pass",
    "rules_evaluated": 4,
    "rules_triggered": [],
    "latency_ms": 85
  },
  "webhook_delivery": {
    "url": "https://your-app.com/webhooks/collie",
    "status": "delivered",
    "response_code": 200,
    "delivered_at": "2025-01-15T10:30:06Z"
  }
}
```

### Job Statuses

| Status              | Description                                               |
| ------------------- | --------------------------------------------------------- |
| `pending`           | Created, waiting to be processed                          |
| `processing`        | Currently being evaluated                                 |
| `awaiting_response` | Inbound rules passed, waiting for LLM response submission |
| `completed`         | All processing finished                                   |
| `failed`            | Processing encountered an error                           |
| `expired`           | Job exceeded its TTL                                      |

### Error Responses

| Status | Description                |
| ------ | -------------------------- |
| `401`  | Missing or invalid API key |
| `404`  | Job not found              |

***

## POST /v1/jobs/{job\_id}/response

Submit an LLM response for a job in `awaiting_response` status. CollieAI evaluates the response against outbound rules.

**URL:** `https://api.collieai.com/v1/jobs/{job_id}/response`

**Auth:** Bearer token (API key `clai_xxx`)

### Path Parameters

| Parameter | Type   | Description        |
| --------- | ------ | ------------------ |
| `job_id`  | string | The job identifier |

### Request Body

| Field      | Type   | Required | Description                          |
| ---------- | ------ | -------- | ------------------------------------ |
| `response` | string | Yes      | The LLM response content to evaluate |

### Example Request

```bash
curl -X POST https://api.collieai.com/v1/jobs/job_abc123def456/response \
  -H "Authorization: Bearer clai_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "response": "The quarterly earnings show revenue of $2.5B..."
  }'
```

### Response -- `202 Accepted`

```json
{
  "job_id": "job_abc123def456",
  "status": "processing"
}
```

### Error Responses

| Status | Description                              |
| ------ | ---------------------------------------- |
| `400`  | Job is not in `awaiting_response` status |
| `401`  | Missing or invalid API key               |
| `404`  | Job not found                            |

***

## GET /v1/jobs

List jobs for the authenticated project.

**URL:** `https://api.collieai.com/v1/jobs`

**Auth:** Bearer token (API key `clai_xxx`)

### Query Parameters

| Parameter       | Type    | Required | Description                                                                                       |
| --------------- | ------- | -------- | ------------------------------------------------------------------------------------------------- |
| `page`          | integer | No       | Page number, starting from 1. Default: 1                                                          |
| `page_size`     | integer | No       | Items per page, 1-100. Default: 20                                                                |
| `status_filter` | string  | No       | Filter by status (`pending`, `processing`, `awaiting_response`, `completed`, `failed`, `expired`) |

### Example Request

```bash
curl "https://api.collieai.com/v1/jobs?page=1&page_size=10&status_filter=completed" \
  -H "Authorization: Bearer clai_your_api_key"
```

### Response -- `200 OK`

```json
{
  "jobs": [
    {
      "job_id": "job_abc123def456",
      "status": "completed",
      "created_at": "2025-01-15T10:30:00Z",
      "completed_at": "2025-01-15T10:30:05Z",
      "metadata": {
        "user_id": "user_123"
      }
    }
  ],
  "total": 42,
  "page": 1,
  "page_size": 10
}
```

### Error Responses

| Status | Description                |
| ------ | -------------------------- |
| `401`  | Missing or invalid API key |
| `422`  | Invalid query parameters   |
