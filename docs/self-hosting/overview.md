---
icon: server
---

# Overview

CollieAI can be self-hosted for full control over your data and infrastructure. This section covers everything you need to deploy and run the platform on your own servers.

## System Requirements

| Requirement             | Minimum version |
| ----------------------- | --------------- |
| Python                  | 3.11+           |
| Docker & Docker Compose | Latest stable   |
| Git                     | 2.x             |

## Architecture

CollieAI consists of five core components:

| Component             | Technology       | Purpose                                             |
| --------------------- | ---------------- | --------------------------------------------------- |
| **API server**        | FastAPI (ASGI)   | REST API, proxy endpoints, dashboard backend        |
| **Primary database**  | PostgreSQL 16    | Projects, policies, rules, users, API keys          |
| **Analytics store**   | ClickHouse 24.11 | Request logs, analytics, audit trail                |
| **Cache & queues**    | Redis 7          | Caching, background job queues, rate limiting       |
| **Background worker** | Python process   | Async filtering, webhook delivery, alert evaluation |

### Components Diagram

```
                    ┌─────────────────────┐
                    │       Client        │
                    └─────────┬───────────┘
                              │
                              ▼
                    ┌─────────────────────┐
                    │   FastAPI API        │
                    │   (uvicorn)          │
                    └──┬───────┬────────┬──┘
                       │       │        │
              ┌────────┘       │        └────────┐
              ▼                ▼                  ▼
     ┌──────────────┐  ┌────────────┐   ┌──────────────┐
     │ PostgreSQL   │  │   Redis    │   │  ClickHouse  │
     │ (primary DB) │  │ (cache +   │   │ (analytics/  │
     │              │  │  queues)   │   │  logs)       │
     └──────────────┘  └─────┬──────┘   └──────────────┘
                             │
                             ▼
                    ┌─────────────────────┐
                    │   Background Worker │
                    │   • Filtering tasks │
                    │   • Webhook delivery│
                    │   • Alert evaluation│
                    └─────────────────────┘
```

The API server handles all HTTP traffic: proxy requests, REST endpoints, and the frontend dashboard. When an async job is created, the API enqueues work into Redis. The background worker picks up jobs from Redis queues, runs security filtering, delivers webhooks, and evaluates alert conditions.

## ML Models

CollieAI uses optional machine learning models for advanced security rules:

| Model                       | Purpose                                            | Notes                                                                                                                                                                                            |
| --------------------------- | -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Lightweight classifiers** | Prompt injection detection                         | Small, fast classifiers                                                                                                                                                                          |
| **LLM detection**           | Generative AI-based threat detection               | Default: Qwen2.5-0.5B-Instruct. Qwen3-8B available as opt-in higher-quality classifier (8-bit, ≥10 GB VRAM). See [LLM Detection rule docs](../security-rules/blocking-threats/llm-detection.md). |
| **Language detection**      | Identify message language for language-based rules | FastText lid.176.ftz                                                                                                                                                                             |

Models are **auto-downloaded from HuggingFace** on first startup when `PRELOAD_MODELS=true`. A valid `HF_TOKEN` is required for gated models. Set `PRELOAD_MODELS=false` to skip model loading for faster startup during development (rules that depend on ML models will silently fail open).

## Billing

When self-hosting, subscription billing is **disabled by default**. The environment variable `BILLING_ENABLED` defaults to `false`, which means:

* No usage limits (API calls, prompt size, project count are all unlimited)
* Billing API endpoints return `404`
* No Stripe integration or billing UI in the dashboard
* The Pricing page shows plan details for reference only

If you want to enable Stripe billing on a self-hosted instance (e.g., for an internal billing system), add the following to your `.env` file:

```bash
BILLING_ENABLED=true
STRIPE_SECRET_KEY=sk_live_...
STRIPE_PUBLISHABLE_KEY=pk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_GROWTH_PRICE_ID=price_...
```

{% hint style="warning" %}
`STRIPE_SECRET_KEY` and `STRIPE_WEBHOOK_SECRET` are sensitive credentials. Keep them in `.env` (which is gitignored) and never commit them to version control.
{% endhint %}

See [Plans & Billing](../getting-started/plans-and-billing.md) for details on plan limits.

## Next Steps

* [**Local Development**](local-development.md) -- step-by-step guide to running CollieAI on your machine.
