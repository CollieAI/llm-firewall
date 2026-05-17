---
description: The security layer between your application and LLM providers.
icon: paw
---

# CollieAi

CollieAi is a drop-in proxy that intercepts API calls to large language models, applies configurable security rules to both input and output, and routes requests to your chosen LLM provider.&#x20;

It is fully CollieAi-compatible - change one line of code and all your existing SDK calls flow through CollieAi.

{% tabs %}
{% tab title="Code snippet" %}
```python
# Before — direct call to OpenAI
client = OpenAI(api_key="sk-...")

# After — change base_url, full protection
client = OpenAI(
    base_url="https://api.collieai.com/v1",
    api_key="clai_your_project_key",
)
```
{% endtab %}
{% endtabs %}

<button type="button" class="button primary" data-action="ask" data-icon="gitbook-assistant">Ask a question…</button>

<a href="/broken/pages/PbYb0GukRhiS4qCHdRal" class="button primary" data-icon="rocket-launch">Quick Start</a><a href="https://app.collieai.io" class="button primary" data-icon="right-to-bracket">Try for Free</a><a href="/broken/pages/FYSW8V8gr9ZF239mySuv" class="button secondary">API Reference</a>

## Key Features

* **Drop-in OpenAI Replacement** - Point your SDK at CollieAI by changing the `base_url`. No other code changes needed.
* **Security Rules Pipeline** - Define inbound and outbound rules that inspect, mask, or block content before it reaches the model or your users.
* **PII Detection** - Identify and mask personally identifiable information using regex patterns, structured ID validation, and dictionary matching.
* **Prompt Injection Blocking** - Stop malicious prompts with a lightweight ML classifier and LLM-based detection.
* **Async Jobs** - Submit long-running requests and receive results via webhooks when they complete.
* **Monitoring and Analytics** - View request logs, track usage analytics, and configure alerts from a single dashboard.
* **Multi-Tenancy** - Organize work into projects, each with its own policies, rules, API keys, and provider tokens.
* **Free & Growth Plans** - Start for free with 15,000 API calls/month and all features included. Upgrade to Growth for production-scale volume and priority support.



## Getting Started

<table data-view="cards"><thead><tr><th></th><th></th><th></th></tr></thead><tbody><tr><td><h4><i class="fa-rocket-launch" style="color:$primary;">:rocket-launch:</i></h4></td><td><h3><a href="/broken/pages/PbYb0GukRhiS4qCHdRal">Getting started</a></h3></td><td>Sign up, create a project, send your first secured request</td></tr><tr><td><h4><i class="fa-plug-circle-plus" style="color:$primary;">:plug-circle-plus:</i></h4></td><td><h3><a href="/broken/pages/EFXeLTHVDQLFgK0Iy51O">Proxy Integration</a></h3></td><td>Connect the OpenAI SDK, streaming, and error handling</td></tr><tr><td><h4><i class="fa-arrows-retweet" style="color:$primary;">:arrows-retweet:</i></h4></td><td><h3><a href="/broken/pages/UGbPFw59rdfVN1YQEWJN">Async Jobs</a></h3></td><td>Submit requests and receive results via webhooks</td></tr></tbody></table>

## Explore

<table data-view="cards"><thead><tr><th></th><th></th><th></th></tr></thead><tbody><tr><td><h4><i class="fa-square-a-lock" style="color:$primary;">:square-a-lock:</i></h4></td><td><h3><a href="/broken/pages/IFQLAOgCVBuArNftVIWh">Security Rules</a></h3></td><td>Configure PII masking, threat blocking, and text processing</td></tr><tr><td><h4><i class="fa-diagram-subtask" style="color:$primary;">:diagram-subtask:</i></h4></td><td><h3><a href="/broken/pages/MPdFTsHz3IYiAsoFdKcn">Projects &#x26; Policies</a></h3></td><td>Organize workloads with projects, policies, and provider tokens</td></tr><tr><td><h4><i class="fa-display-chart-up-circle-currency" style="color:$primary;">:display-chart-up-circle-currency:</i></h4></td><td><h3><a href="/broken/pages/Rx1ouIJ4TR4itfsu4i4G">Monitoring</a></h3></td><td>Submit requests and receive results via webhooks</td></tr></tbody></table>
