---
icon: arrows-repeat
---

# Overview

CollieAI Async Jobs provide fully asynchronous message filtering with webhook delivery. Unlike the [drop-in proxy](/broken/pages/b60c2f10242f8586ac226c3dc4936c58d5004432), which sits between your application and OpenAI, the Jobs API lets you **use any LLM** and just filter messages through CollieAI.

CollieAI does **not** call LLMs on your behalf. You send content to be filtered, receive the filtered result via webhook, call your own LLM, and optionally send the LLM response back for outbound filtering.

***

## Filtering Modes

Async Jobs support three modes depending on which fields you include in the request body.

| Mode         | Fields                                  | Use Case                                     |
| ------------ | --------------------------------------- | -------------------------------------------- |
| **Legacy**   | `message`                               | Inbound filtering only (backward compatible) |
| **Separate** | `message_input` and/or `message_output` | Filter inbound or outbound independently     |
| **Combined** | `message_input` + `message_output`      | Filter both directions in a single request   |

**Legacy mode** exists for backward compatibility. For new integrations, use `message_input` and `message_output` for better ML accuracy -- CollieAI applies different rule sets to inbound (user) and outbound (AI) content.

***

## How It Works

### Two-Hook Pattern (Separate Inbound + Outbound)

This is the most common pattern when you need to filter both user input and LLM output.

```
Your App                        CollieAI                        Your LLM
   |                               |                               |
   |  1. POST /v1/jobs             |                               |
   |     message_input + webhook   |                               |
   |  ---------------------------> |                               |
   |                               |                               |
   |  <-- 202 Accepted             |                               |
   |     job_id + webhook_secret   |                               |
   |                               |                               |
   |  2. Webhook: inbound_complete |                               |
   |  <--------------------------- |                               |
   |     (filtered user content)   |                               |
   |                               |                               |
   |  3. Call LLM with filtered content                            |
   |  ----------------------------------------------------------> |
   |                                                               |
   |  <-- LLM response                                            |
   |  <---------------------------------------------------------- |
   |                               |                               |
   |  4. POST /v1/jobs/{id}/response                               |
   |     llm_response              |                               |
   |  ---------------------------> |                               |
   |                               |                               |
   |  5. Webhook: outbound_complete|                               |
   |  <--------------------------- |                               |
   |     (filtered LLM response)   |                               |
   |                               |                               |
```

### Combined Mode (Single Request)

When you already have both the user message and the AI response (for example, logging or post-processing), you can filter both in a single request.

```
Your App                        CollieAI
   |                               |
   |  POST /v1/jobs                |
   |  message_input + message_output + webhook
   |  ---------------------------> |
   |                               |
   |  <-- 202 Accepted             |
   |                               |
   |  Webhook: inbound_complete    |
   |  <--------------------------- |
   |                               |
   |  Webhook: outbound_complete   |
   |  <--------------------------- |
   |                               |
```

Webhooks are always delivered in order: inbound first, then outbound.

***

## When to Use Async Jobs vs Drop-in Proxy

|                        | Async Jobs                                                 | Drop-in Proxy            |
| ---------------------- | ---------------------------------------------------------- | ------------------------ |
| **LLM provider**       | Any (you call it)                                          | OpenAI-compatible APIs   |
| **Integration effort** | Webhook handler required                                   | Change one base URL      |
| **Flexibility**        | Full control over LLM calls                                | Transparent proxying     |
| **Latency**            | Async (webhook-based)                                      | Synchronous              |
| **Best for**           | Custom LLM pipelines, multi-model setups, batch processing | Quick OpenAI integration |

Choose **Async Jobs** when you need full control over your LLM calls, use non-OpenAI models, or want to decouple filtering from your request flow.

Choose the **Drop-in Proxy** when you use OpenAI and want the simplest possible integration.

***

## Next Steps

* [Creating Jobs](/broken/pages/c548a53e0fd1fb57f84d6df42235d3a0da3881f2) -- submit content for filtering
* [Webhooks](/broken/pages/aa060b7742db4c9ab1c629efbddc3a3db0916e4a) -- receive and verify filtered results
* [Job Lifecycle](/broken/pages/d578acc1c0d46e4c010641fc6e28fd9abc3c6019) -- understand job statuses and best practices
