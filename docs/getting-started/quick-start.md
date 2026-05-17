---
icon: bolt-auto
---

# Quick start

Get your first secured LLM call running in about 5 minutes.

{% stepper %}
{% step %}
### Sign Up

Go to [app.collieai.com](https://app.collieai.com/) and create an account. You can sign in with **Google** (your account is created automatically on first login) or register with **email and password**.

All new accounts start on the **Free plan** - 15,000 API calls/month with full access to every security feature. See [Plans & Billing](/broken/pages/54aaf323a9267c336d93a3e24f21fa2eff822e46) for details.

See [Authentication](/broken/pages/1845ab390e5ce0610c3639336c3607da0944e5f1) for more details on sign-in methods.
{% endstep %}

{% step %}
### Add a Provider Token

CollieAI needs your LLM provider's API key to forward requests on your behalf.

1. Open **Settings > Provider Tokens** in the dashboard.
2. Click **Add Token**.
3. Select the provider (e.g., OpenAI) and paste your API key (`sk-...`).
4. Check **Set as default** so CollieAI uses this token when proxying requests.

Your key is encrypted at rest and never exposed in logs or responses.

See [Provider Tokens](/broken/pages/0e225d0c0161078d0be1ce78f322f6f14c27bb07) for more details.
{% endstep %}

{% step %}
### Create a Project

Projects group your policies, rules, and API keys together.

1. Go to **Projects** in the dashboard.
2. Click **New Project**.
3. Give it a name (e.g., `my-chatbot`) and an optional description.

See [Projects](/broken/pages/12493f7755c3bc9f868977b8eb076b8598bb0ddb) for more details.
{% endstep %}

{% step %}
### Create a Policy and Add Rules

Policies contain the security rules that filter your traffic.

1. Inside your project, go to **Policies** and click **Create Policy**.
2. Give it a name (e.g., `default-policy`) and set it as the active policy.
3. Click **Add Rule** and create a regex rule to mask email addresses:

| Field     | Value                                            |
| --------- | ------------------------------------------------ |
| Name      | `mask-emails`                                    |
| Type      | Regex                                            |
| Direction | All                                              |
| Decision  | Mask                                             |
| Pattern   | `[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}` |

See [Policies](/broken/pages/838f7df49ce642ec55293bc256df97f808287402) and [Security Rules](/broken/pages/a232c2070154c78b64b970b0ed0c632117af12b9) for more details.
{% endstep %}

{% step %}
### Generate an API Key

1. Inside your project, go to **API Keys** and click **Create Key**.
2. Give it a descriptive name (e.g., `dev-key`).
3. Copy the key immediately -- it starts with `clai_` and is shown only once.

```
clai_AbCdEfGhSwDWWArWWDcpQrStUvWxYz012345678901234
```

Store it securely in an environment variable or secrets manager.

See [API Keys](/broken/pages/7feac2acca8bcde057933e84cdddfb8ff4d6f647) for more details.
{% endstep %}

{% step %}
### Make Your First Request

Install the OpenAI Python SDK if you have not already:

```bash
pip install openai
```

Point it at CollieAI by changing the `base_url` and using your CollieAI API key:

```python
from openai import OpenAI

client = OpenAI(
    api_key="clai_AbAddEfGhIjwADnOpQrStUvWxYz012345678901234",
    base_url="https://api.collieai.com/v1",
)

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "user", "content": "Summarize our Q4 roadmap."}
    ],
)

print(response.choices[0].message.content)
```

That is it. Your request now flows through CollieAI's security pipeline before reaching OpenAI.
{% endstep %}

{% step %}
### See It in Action

With the email-masking rule active, try sending a message that contains a PII:

```python
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "user", "content": "My email is alice@example.com, can you help me?"}
    ],
)
```

CollieAI intercepts the request, masks the email address before it reaches OpenAI, and returns the model's response. Open the **Logs** page in the dashboard to see the request with the triggered `mask-emails` rule and the redacted content.
{% endstep %}
{% endstepper %}

## Next Steps

* [Proxy Integration](/broken/pages/157c3242e7f3564dfdc06320e342fd0f5853f712) -- Streaming, error handling, and advanced SDK usage.
* [Async Jobs](/broken/pages/0314bdda43619d99e4dcd4c0a5ebbd9996c1b9ec) -- Submit requests and receive results via webhooks.
* [Security Rules](/broken/pages/a232c2070154c78b64b970b0ed0c632117af12b9) -- Explore every rule type: PII detection, prompt injection blocking, URL filtering, and more.
* [Monitoring](/broken/pages/c43f7395c96e1c24318aa800d9a4b7229fa328db) -- View logs, analytics, and configure alerts.
