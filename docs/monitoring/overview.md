---
icon: computer
---

# Overview

CollieAI provides built-in observability for every API call that passes through your proxy. Every request is automatically logged with detailed metadata, giving you full visibility into traffic, rule enforcement, and performance.

Monitoring is organized around three pillars:

* [**Logs**](logs.md) - Raw event history. Every request and its outcome is recorded as an individual log entry, including the event type, model used, tokens consumed, which rules fired, and whether the request was blocked.
* [**Analytics**](analytics.md) - Aggregated KPIs and charts. See total request volume, blocked rate, error rate, P95 latency, and rule trigger counts over configurable time ranges.
* [**Alerts**](alerts.md) - Threshold-based notifications. Define rules like "alert me when the error rate exceeds 5% over 15 minutes" and get notified automatically.

## Per-project isolation

All monitoring data is scoped to a project. Use the project selector at the top of any monitoring page to switch between projects. Each project's logs, analytics, and alerts are completely independent.

## How data is collected

CollieAI automatically logs every API call that flows through the proxy. There is no additional configuration required -- once you start routing traffic through CollieAI, monitoring data begins appearing immediately.
