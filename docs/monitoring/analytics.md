---
icon: subtitles
---

# Analytics

The Analytics page provides aggregated metrics and charts that summarize your project's traffic patterns, rule effectiveness, and performance over time.

## Key metrics

The following KPIs are displayed at the top of the Analytics page:

| Metric                 | Description                                                                                  |
| ---------------------- | -------------------------------------------------------------------------------------------- |
| **Total requests**     | The total number of API calls processed in the selected time range.                          |
| **Blocked rate**       | The percentage of requests that were blocked by inbound or outbound rules.                   |
| **Error rate**         | The percentage of requests that resulted in an error.                                        |
| **P95 latency**        | The 95th percentile response time, meaning 95% of requests completed faster than this value. |
| **Rule trigger count** | The total number of times any rule was triggered across all requests.                        |

## Using the Analytics page

1. **Select a project** using the project selector at the top of the page (same pattern as the Logs page).
2. **Choose a time range** to control the period of data shown in all metrics and charts.
3. **Review the KPI cards** for a high-level summary.
4. **Examine the charts** to see how each metric trends over time within the selected range.

## What to look for

Analytics helps you answer common operational questions:

* **Monitor traffic patterns** — See how request volume changes throughout the day or week. Identify peak usage periods.
* **Track rule effectiveness** — Check the blocked rate and rule trigger count to understand how often your rules are firing and whether they are catching the traffic you expect.
* **Identify spikes in blocked requests** — A sudden increase in the blocked rate may indicate a new pattern of misuse, or it may mean a rule is too aggressive and needs tuning.
* **Spot performance issues** — Watch the P95 latency metric for degradation. If latency is climbing, it may point to upstream model slowdowns or increased payload sizes.
