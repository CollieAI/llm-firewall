---
icon: messages-dollar
---

# Plans and billing

CollieAI offers two subscription plans. Both plans include **all security features** - the difference is in usage limits and scale.

### Free

|                           |                                                                              |
| ------------------------- | ---------------------------------------------------------------------------- |
| **Price**                 | **Free** forever                                                             |
| **API calls**             | 15,000 / month                                                               |
| **Max prompt size**       | 10,000 tokens                                                                |
| **Projects**              | 1                                                                            |
| **Security rules**        | All types                                                                    |
| **Dashboard & logs**      | Included                                                                     |
| **Async jobs & webhooks** | Included                                                                     |
| **Support**               | [Feedback & Support](/broken/pages/dd0dc1b9355d2edb63e73566059c5b917b7fbb73) |

The Free plan is designed for **evaluation and prototyping**. You get full access to every filtering feature - regex, dictionary matching, structured ID detection, prompt injection blocking, LLM detection, URL filtering, normalization, and language detection.

### Growth

|                           |                                   |
| ------------------------- | --------------------------------- |
| **Price**                 | **199 / month**                   |
| **API calls**             | 2,000,000 / month                 |
| **Max prompt size**       | Configurable (no platform cap)    |
| **Projects**              | Unlimited                         |
| **Security rules**        | All types                         |
| **Dashboard & logs**      | Included                          |
| **Async jobs & webhooks** | Included                          |
| **Support**               | Priority support, 4h response SLA |

Growth is for **teams shipping AI to production** who need higher volume, multiple projects, and guaranteed support.

### Enterprise

|                          |                                     |
| ------------------------ | ----------------------------------- |
| **Price**                | **Custom** - contact us             |
| **API calls**            | Unlimited                           |
| **Everything in Growth** | Included                            |
| **Deployment**           | On-premise / dedicated              |
| **Support**              | Dedicated support + SLA             |
| **Onboarding**           | Includes onboarding & dedicated CSM |

Enterprise is for **companies with compliance requirements, large volumes, or full data control**. [Contact us](/broken/pages/dd0dc1b9355d2edb63e73566059c5b917b7fbb73) to discuss your needs.

## Usage tracking

Your monthly API call count includes both **proxy requests** (`POST /v1/chat/completions`) and **async jobs** (`POST /v1/jobs`). Usage is tracked per user across all projects.

You can check your current usage at any time:

* **Dashboard:** Open **Pricing** in the sidebar to see the usage progress bar.
* **API:** `GET /api/v1/billing/subscription` returns usage details.
* **Response headers:** Every proxy and job response includes usage headers:

| Header                     | Description                                                                  |
| -------------------------- | ---------------------------------------------------------------------------- |
| `X-Collie-Usage-Limit`     | Monthly call limit for your plan                                             |
| `X-Collie-Usage-Remaining` | Remaining calls this period                                                  |
| `X-Collie-Usage-Reset`     | ISO 8601 timestamp when the counter resets                                   |
| `X-Collie-Usage-Warning`   | Present when usage exceeds 80% (`warning_80`, `warning_90`, or `hard_limit`) |

### Grace buffer

CollieAI does **not** hard-block at exactly 100% of your limit. Instead:

* **80%** - Warning header added to responses; email notification sent.
* **90%** - Second warning email.
* **100-110%** - Requests are still allowed, but a `hard_limit` warning is included.
* **>110%** - Requests return `429 Too Many Requests` with `"type": "billing_limit"`.

This ensures production workloads are not suddenly interrupted at the exact boundary.

{% hint style="info" %}
**Fairness note:** Only requests that are actually processed count toward your monthly usage. Requests rejected locally by rate limits, security policy blocks, or configuration errors (e.g. missing provider token) do **not** consume quota. Under high concurrency near the limit boundary, a small number of extra requests (within the grace buffer) may be allowed before enforcement kicks in. This is by design.
{% endhint %}

### Period reset

* **Growth plan:** Usage resets at the start of each Stripe billing cycle (monthly from your subscription date).
* **Free plan:** Usage resets at the start of each calendar month (UTC).

## Upgrading

{% stepper %}
{% step %}
### Open **Pricing**

Open **Pricing** in the CollieAI dashboard.
{% endstep %}

{% step %}
### Click **Upgrade to Growth**

Click **Upgrade to Growth**.
{% endstep %}

{% step %}
### Complete payment

Complete payment in the Stripe Checkout page.
{% endstep %}

{% step %}
### Return to CollieAI

You are redirected back to CollieAI with your new limits active immediately.
{% endstep %}
{% endstepper %}

Payment is handled entirely by [Stripe](https://stripe.com/). CollieAI never sees or stores your card details.

## Managing your subscription

Click **Manage Subscription** on the Pricing page to open the Stripe Customer Portal where you can:

* Update your payment method
* View invoices and receipts
* Cancel your subscription

When you cancel, your Growth plan remains active until the end of the current billing period, then downgrades to Free.

***

## Self-hosted deployments

When self-hosting CollieAI, billing is **disabled by default** (`BILLING_ENABLED=false`). All usage limits and Stripe integration are bypassed. You get unlimited projects, API calls, and prompt sizes with no billing UI.

See [Self-Hosting](/broken/pages/ff04569209c61292233939bb7e27818718607270) for details.
