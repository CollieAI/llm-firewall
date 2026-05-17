<p align="center">
  <a href="https://collieai.io">
    <img src="https://collieai.io/logo-1.svg" alt="CollieAi" width="80" />
  </a>
</p>

<h1 align="center">LLM Firewall — AI Security Proxy</h1>

<h4 align="center">
  Drop-in OpenAI proxy that masks PII, blocks prompt injections &amp; logs every LLM call.<br/>
  One line change. Nine rule types. Works with any provider.
</h4>

<p align="center">
  <a href="https://collieai.io"><img src="https://img.shields.io/badge/website-collieai.io-blue" alt="CollieAi Website" /></a>
  <a href="https://docs.collieai.io"><img src="https://img.shields.io/badge/docs-docs.collieai.io-green" alt="Documentation" /></a>
  <a href="https://app.collieai.com"><img src="https://img.shields.io/badge/free_tier-15K_calls/mo-brightgreen" alt="Free Tier" /></a>
</p>

<p align="center">
  <a href="https://docs.collieai.io/getting-started/quick-start">Quick Start</a> · <a href="https://docs.collieai.io">Docs</a> · <a href="https://docs.collieai.io/api-reference">API Reference</a> · <a href="https://app.collieai.com">Try Free</a> · <a href="https://docs.collieai.io/self-hosting">Self-Host</a>
</p>

## Why LLM Firewall?

You're building with GPT-4, Claude, Gemini — but every API call is a potential data leak. Users paste credit cards into chat. Attackers inject malicious prompts. Sensitive data flows to third-party models without any filtering.

**LLM Firewall** by [CollieAi](https://collieai.io) is a **drop-in security proxy** that sits between your application and the LLM provider. It inspects, masks, and blocks — in real time, with zero code changes beyond swapping the base URL.

<!-- GIF: dashboard, policy engine, real-time logs -->

<img width="1000" height="561" alt="github-4" src="https://github.com/user-attachments/assets/de356d6e-6483-4d30-aa80-d8c36482816e" />



## Key Features

- **Drop-in OpenAI-compatible proxy** — change `base_url`, keep your SDK. Python, Node.js, Go, curl — anything that speaks OpenAI works with CollieAi
- **PII detection & masking** — regex patterns, structured ID validation (Luhn, MOD-97), and dictionary matching via Aho-Corasick
- **Prompt injection protection** — ML classifier (10-50 ms) plus optional LLM-based analysis for sophisticated attacks
- **Flexible policy engine** — 6 configurable rule categories, each with monitor (log only) or enforce (mask/block) mode. Custom regex, dictionaries, thresholds — tune everything per project
- **Inbound + outbound filtering** — scan both user prompts and model responses with independent policies
- **User satisfaction tracking** — detect negative sentiment, monitor dissatisfaction metrics, get alerts when something goes wrong
- **Full audit trail** — every request logged with triggered rules, latency, and token counts. Filter by project, time range, or rule type
- **Dashboard** — create projects, configure rules, add provider tokens, review logs. No config files, no deployments

> **Works with:** OpenAI · Claude · deepseek · Gemini · Azure OpenAI · AWS Bedrock · any OpenAI-compatible endpoint

## Quick Start — 3 Steps

<table>
<tr>
<td width="40%">

**1. Point your client at CollieAi**

Sign up for a free API key and change your `base_url`. No SDK required, no architecture changes.

</td>
<td width="60%">

```python
# Change two config values
base_url = "https://api.collieai.com/v1"
api_key  = "clai_your_project_key"

# Free: 15,000 calls/month
```

</td>
</tr>
<tr>
<td width="40%">

**2. Your code stays the same**

CollieAi is a transparent proxy. Use the standard OpenAI SDK — every request is filtered by your policy rules automatically.

</td>
<td width="60%">

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.collieai.com/v1",
    api_key="clai_...",
)
client.chat.completions.create(...)
```

</td>
</tr>
<tr>
<td width="40%">

**3. Threats blocked, full audit trail**

Only safe, compliant responses reach your users. Dashboard shows every blocked request with triggered rules and context.

</td>
<td width="60%">

```json
{
  "error": {
    "message": "Content blocked by policy",
    "type":    "content_blocked",
    "code":    400
  }
}
```

</td>
</tr>
</table>

Or set environment variables — **zero code changes**:

```bash
export OPENAI_BASE_URL=https://api.collieai.com/v1
export OPENAI_API_KEY=clai_your_project_key
# Your existing code works unchanged
```

**[Get Started Free — 15,000 calls/month, no credit card](https://app.collieai.com)**

## How It Works

<img width="1341" height="548" alt="github-2" src="https://github.com/user-attachments/assets/cf40c56e-a7d9-4255-9a21-a3c30e7a06cf" />


## Security Rules — Fully Configurable

Select the threats you want to protect against. Each rule category is independently configurable — choose enforcement mode, set thresholds, add custom patterns, and apply rules to inbound prompts, outbound responses, or both. Start in **monitor** mode to observe, switch to **enforce** when ready.

### Prompt Injection <kbd>#1 LLM threat</kbd>

Detect and block prompt injection & jailbreak attempts using dictionaries, ML models, and language detection. Lightweight ML classifier runs in 10-50 ms — catches jailbreaks, role hijacking, and instruction override before the request ever reaches the LLM. Optional secondary LLM-based analysis for sophisticated attacks that evade pattern matching.

### PII & Financial Data <kbd>Compliance</kbd>

Detect credit cards, IBANs, SSNs, emails, and other personal data. Three complementary methods: configurable regex patterns for any format, checksum validation (Luhn for credit cards, MOD-97 for IBANs) for structural verification, and Aho-Corasick dictionary matching for names and org-specific terms.

### Profanity & Sensitive Words <kbd>Brand safety</kbd>

Filter profanity and sensitive content using multi-language dictionary groups. Upload custom word lists per project — block or mask offensive terms, competitor names, internal codenames, or any content that violates your brand policy.

### Secrets & API Keys <kbd>Data leak risk</kbd>

Detect API keys, tokens, private keys, and credentials via regex patterns. Catches `sk-...`, `ghp_...`, `AKIA...`, PEM keys, JWT tokens, and other secret formats before they leak to third-party models.

### Malicious URLs <kbd>Common threat</kbd>

Filter suspicious URLs by scheme, domain, IP literals, and encoded patterns. Block phishing links, data exfiltration endpoints, and obfuscated URLs that try to bypass simple string matching.

### Hidden Payloads <kbd>Advanced threat</kbd>

Detect hidden base64-encoded payloads and file data in messages. Automatically decode and inspect embedded content — catches hidden instructions, encoded exploits, and smuggled data that looks like innocent text.

> **Flexible enforcement:** Every rule supports `monitor` mode (log detections, pass through) or `enforce` mode (mask PII / block request). Rules run bidirectionally — configure inbound and outbound independently. Add your own regex patterns, upload custom dictionaries, set ML confidence thresholds, and define different policies per project.

---

## Works With Every Provider

CollieAi routes to any OpenAI-compatible LLM provider. Configure your provider token in the dashboard — CollieAi handles the rest.

| Provider | Models |
|----------|--------|
| **OpenAI** | GPT-4o, GPT-4, GPT-3.5 |
| **Anthropic** | Claude Sonnet, Opus, Haiku |
| **Google** | Gemini Pro, Flash |
| **deepseek** | deepseek-chat, deepseek-coder |
| **Azure OpenAI** | All Azure-hosted models |
| **AWS Bedrock** | Via OpenAI-compat wrapper |
| **Self-hosted** | vLLM, Ollama, LocalAI |

Full SSE streaming support for all providers. Async webhook mode available for batch processing.

---

## Documentation

Full documentation at **[docs.collieai.io](https://docs.collieai.io)**

| Section | What you'll learn |
|---------|------------------|
| [**Quick Start**](https://docs.collieai.io/getting-started/quick-start) | Sign up → first secured request in 5 minutes |
| [**Proxy Integration**](https://docs.collieai.io/proxy-integration) | Python, Node.js, cURL, streaming, error handling |
| [**Security Rules**](https://docs.collieai.io/security-rules) | All rule types, pipeline ordering, enforcement modes |
| [**Async Jobs**](https://docs.collieai.io/async-jobs) | Webhooks, job lifecycle, two-hook pattern |
| [**Projects & Policies**](https://docs.collieai.io/projects-and-policies) | Multi-tenancy, data retention, provider tokens |
| [**Monitoring**](https://docs.collieai.io/monitoring) | Logs, analytics dashboards, alerts |
| [**API Reference**](https://docs.collieai.io/api-reference) | Full endpoint docs for 14 API groups |
| [**Self-Hosting**](https://docs.collieai.io/self-hosting) | Docker Compose, ML model config, local dev |

---

<p align="center">
  <b>Secure your LLM calls in 5 minutes. Free forever, no credit card.</b><br/><br/>
  <a href="https://app.collieai.com">
    <img src="https://img.shields.io/badge/Get_Started_Free-→-brightgreen?style=for-the-badge" alt="Get Started Free" />
  </a>
</p>

<p align="center">
  <a href="https://collieai.io">Website</a> · 
  <a href="https://app.collieai.com">Dashboard</a> · 
  <a href="https://docs.collieai.io">Docs</a> · 
  <a href="https://github.com/CollieAi/llm-firewall/issues">Issues</a>
</p>
