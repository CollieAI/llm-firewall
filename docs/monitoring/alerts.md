---
icon: light-emergency-on
---

# Alerts

Alerts let you define threshold-based rules that are automatically evaluated against your project's metrics. When a condition is met, CollieAI creates an alert event so you can respond quickly to anomalies.

## How alerts work

1. You create an **alert rule** that defines a condition (e.g. "error rate > 5% over the last 15 minutes").
2. CollieAI evaluates all active rules **every 60 seconds**.
3. When a condition is met, an **alert event** is created with the actual metric value that triggered it.
4. You can view and **acknowledge** alert events in the dashboard or via the API.

## Available metrics

| Metric           | Unit  | Description                     |
| ---------------- | ----- | ------------------------------- |
| `blocked_rate`   | %     | Percentage of blocked requests. |
| `error_rate`     | %     | Percentage of error responses.  |
| `latency_p95`    | ms    | 95th percentile latency.        |
| `rule_triggers`  | count | Total rule trigger count.       |
| `total_requests` | count | Total request count.            |

## Operators

You can use any of the following comparison operators in your alert conditions:

* `>` greater than
* `>=` greater than or equal to
* `<` less than
* `<=` less than or equal to

## Time windows

Each rule evaluates its metric over a rolling time window. Available windows:

* **5m** -- 5 minutes
* **15m** -- 15 minutes
* **1h** -- 1 hour

## Cooldown

Every alert rule has a **cooldown period** (between 1 and 1440 minutes, default 60 minutes). After an alert event fires, the rule will not fire again until the cooldown period has elapsed. This prevents alert storms when a metric stays above the threshold for an extended period.

## Using the Alerts page

The Alerts page in the dashboard has two tabs:

{% tabs %}
{% tab title="Rules tab" %}
Manage your alert rules here. Each rule shows its metric, condition, time window, and current status. Use the **enable/disable toggle** to activate or deactivate individual rules without deleting them.
{% endtab %}

{% tab title="History tab" %}
View a timeline of all alert events. Unread events are marked with an indicator so you can quickly spot new alerts. The history **auto-refreshes every 30 seconds** to keep you up to date.

To **acknowledge** an alert event, click on it and mark it as read. You can also acknowledge events in bulk.
{% endtab %}
{% endtabs %}

## API reference

### Create a rule

```http
POST /api/v1/alerts/projects/{project_id}/rules
```

```json
{
  "metric": "error_rate",
  "operator": ">",
  "threshold": 5,
  "window": "15m",
  "cooldown_minutes": 60
}
```

### List alert events

```http
GET /api/v1/alerts/projects/{project_id}/events
```

Returns all alert events for the project, newest first.

### Acknowledge an event

```http
POST /api/v1/alerts/projects/{project_id}/events/{event_id}/acknowledge
```

Marks a single alert event as read.

### Get unread count

```http
GET /api/v1/alerts/projects/{project_id}/events/unread-count
```

Returns the number of unacknowledged alert events. Useful for displaying a badge in your own UI or for polling.

## Example rules

**High error rate** -- Alert when errors exceed 5% over 15 minutes:

* Metric: `error_rate`, Operator: `>`, Threshold: `5`, Window: `15m`

**Slow P95 latency** -- Alert when 95th percentile latency exceeds 3 seconds:

* Metric: `latency_p95`, Operator: `>`, Threshold: `3000`, Window: `5m`

**Low traffic detection** -- Alert when request volume drops below expected levels, which may indicate an outage or misconfiguration:

* Metric: `total_requests`, Operator: `<`, Threshold: `10`, Window: `1h`
