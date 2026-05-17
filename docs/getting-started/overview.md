---
description: >-
  CollieAi is a security layer that sits between your application and your LLM
  provider.
icon: rocket-launch
---

# Getting started

CollieAi is a security layer that sits between your application and your LLM provider. Every request and response passes through a configurable filtering pipeline that can detect PII, block prompt injections, filter URLs, and more - without changing how your application talks to the model.

<figure><img src="../.gitbook/assets/1 (2).png" alt=""><figcaption></figcaption></figure>

## What you'll need

Before you begin, make sure you have:

* [x] **A CollieAi account** - Sign up at [app.collieai.com](https://app.collieai.com/) using Google OAuth or email/password.
* [x] **A provider API key** - You need an API key from your LLM provider (e.g., an OpenAI API key). CollieAi proxies your requests using this key.

## Two Ways to Integrate

<table data-card-size="large" data-view="cards"><thead><tr><th></th><th></th><th></th></tr></thead><tbody><tr><td><h4><i class="fa-right-to-bracket" style="color:$primary;">:right-to-bracket:</i></h4></td><td><strong>Drop-in Proxy</strong></td><td>The fastest path. Point your OpenAI SDK (or any OpenAI-compatible client) at CollieAI by changing the <code>base_url</code>. <br><br>Your existing code works as-is - CollieAI transparently applies security rules and forwards requests to the provider.<br><br>Best for: real-time chat, streaming responses, low-latency use cases.</td></tr><tr><td><h4><i class="fa-rotate" style="color:$primary;">:rotate:</i></h4></td><td><strong>Async Jobs</strong></td><td>Submit a request and receive a callback when the result is ready. <br><br>Useful for batch processing, long-running prompts, or workflows where you do not need an immediate response.<br><br>Best for: background processing, batch pipelines, webhook-driven architectures.</td></tr></tbody></table>
