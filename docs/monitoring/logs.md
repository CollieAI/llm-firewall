---
icon: laptop-binary
---

# Logs

Every request that passes through CollieAI is recorded as a log entry with detailed metadata. Logs give you a complete, searchable history of all API activity for a project.

***

## Event types

Each log entry is classified by event type, which tells you exactly what happened:

### Chat events

| Event type              | Description                                                                             |
| ----------------------- | --------------------------------------------------------------------------------------- |
| `chat.completion`       | A chat completion request was processed successfully.                                   |
| `chat.inbound_blocked`  | The incoming request was blocked by an inbound rule before reaching the upstream model. |
| `chat.outbound_blocked` | The model response was blocked by an outbound rule before being returned to the caller. |
| `chat.error`            | The request resulted in an error (upstream failure, timeout, etc.).                     |

### Job events

| Event type               | Description                                        |
| ------------------------ | -------------------------------------------------- |
| `job.created`            | A new asynchronous job was created.                |
| `job.response_submitted` | The job completed and its response was submitted.  |
| `job.inbound_filtered`   | The job's input was filtered by an inbound rule.   |
| `job.outbound_filtered`  | The job's output was filtered by an outbound rule. |
| `job.error`              | The job encountered an error during processing.    |

## What each log entry contains

Every log entry includes the following fields:

* **Event type** - One of the event types listed above.
* **Direction** - `inbound` or `outbound`, indicating which stage of the request the event occurred at.
* **Timestamp** - When the event was recorded.
* **Duration** - How long the request took (in milliseconds).
* **Model** - The model that was called (e.g. `gpt-4o`, `claude-3-opus`).
* **Tokens used** - Input and output token counts.
* **Triggered rules** - Which rules fired during processing.
* **Blocked status** - Whether the request or response was blocked.

## Using the Logs page

The Logs page in the CollieAI dashboard provides an interactive view of your project's event history.

{% stepper %}
{% step %}
### Select a project

Select a project using the project selector at the top of the page.
{% endstep %}

{% step %}
### Filter by event type

Filter by event type using the dropdown filter to narrow results to specific event types (e.g. show only `chat.inbound_blocked` events).
{% endstep %}

{% step %}
### Browse entries

Browse entries in the log table. Each row shows the event type as a color-coded badge, direction, timestamp, model, and other key fields.
{% endstep %}

{% step %}
### Open a detail view

Click any entry to open a detail view containing:

* The full request and response payloads.
* All rules that were triggered.
* The `request_id` (for chat events) or `job_id` (for job events), useful for correlating with your own systems.
{% endstep %}
{% endstepper %}

## Logs API

You can also query logs programmatically:

```http
GET /logs/api?event_type=chat.completion
```

Use the `event_type` query parameter to filter results. This returns the same data shown in the dashboard, in JSON format.

## Retention and storage

Logs are stored in ClickHouse, which is optimized for fast querying over large volumes of time-series data. This means searches and filters remain responsive even as your log volume grows.
