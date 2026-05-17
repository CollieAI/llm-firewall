---
icon: toggle-large-on
---

# Overview

Security rules are the core of CollieAI. They define what to look for in messages and what to do when a match is found. Rules are organized inside [policies](/broken/pages/3363b7ce051b58141f1afd0d0e3f380b2a90637e), and each project has one active policy at a time.

## How the Pipeline Works

Every request passes through a multi-stage pipeline:

```
1. Authenticate    -->  Validate the API key and load the project's policy
2. Inbound filter  -->  Apply inbound rules to the user's message
3. Proxy / Process -->  Forward the (filtered) request to the LLM provider
4. Outbound filter -->  Apply outbound rules to the model's response
5. Return          -->  Send the filtered response back to your application
```

If any rule with a **block** decision triggers during the inbound stage, the request is rejected immediately and never reaches the provider.

## Rule Properties

Every rule has the following properties:

| Property      | Description                                                                           |
| ------------- | ------------------------------------------------------------------------------------- |
| **Name**      | A human-readable label for the rule.                                                  |
| **Type**      | Determines what the rule does (e.g., `regex`, `structured_id`, `llm_detection`).      |
| **Order**     | Numeric value that controls when this rule runs relative to others. Lower runs first. |
| **Direction** | Which stage of the pipeline the rule applies to: `inbound`, `outbound`, or `all`.     |
| **Decision**  | What happens when the rule matches: `allow`, `mask`, or `block`.                      |

## Processing Order

Rules execute in ascending order by the **order** field. If two rules share the same order value, the one created first runs first.

Choose order values intentionally -- the sequence matters because earlier rules can change the content that later rules see.

## Decisions

When a rule matches content, its **decision** determines what happens next:

| Decision  | Behavior                                                                                                              |
| --------- | --------------------------------------------------------------------------------------------------------------------- |
| **Allow** | The content is explicitly permitted. **Processing stops** -- no further rules are evaluated.                          |
| **Mask**  | The matched content is replaced with a placeholder (e.g., `****`). Processing **continues** with the remaining rules. |
| **Block** | The entire request is rejected with a `policy_violation` error. **Processing stops** immediately.                     |

Because `allow` and `block` both stop the pipeline, put your most important rules first.

## Directions

| Direction    | When it runs                                                                                        |
| ------------ | --------------------------------------------------------------------------------------------------- |
| **Inbound**  | Before the request is sent to the LLM provider. Use this to filter what the model sees.             |
| **Outbound** | After the response comes back from the provider. Use this to filter what your application receives. |
| **All**      | Runs on both inbound and outbound stages.                                                           |

## The 9 Rule Types

CollieAI provides 9 rule types organized into three categories:

### PII Detection

Identify and mask personally identifiable information.

| Type                                                                           | Description                                                                                                                                                 |
| ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [**Regex Patterns**](/broken/pages/2758ce52079eab3cf924b9157be77a0d2ea48e02)   | Match custom patterns (emails, phone numbers, account IDs) using regular expressions.                                                                       |
| [**Structured IDs**](/broken/pages/dbfff7aa5a38c6d1859fdd10c918e78034d890c9)   | Detect credit card numbers, IBANs, BIC/SWIFT codes, and national IDs with checksum validation.                                                              |
| [**Dictionary Match**](/broken/pages/08a603b2d4a71a6d75f4f6c5c19fbd915b00652a) | Fast multi-pattern matching against word lists. Link a dictionary of terms (names, company names, medical terms) and find all occurrences in a single pass. |

### Threat Blocking

Prevent malicious or unwanted content from reaching the model or your users.

| Type                                                                           | Description                                                                                                                |
| ------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| [**Prompt Injection**](/broken/pages/511a7a12e6e081c01f866708abbb9a29b7baa155) | A lightweight ML classifier that detects prompt injection and jailbreak attempts with a configurable confidence threshold. |
| [**LLM Detection**](/broken/pages/331a04445a5e9af126e9dc37b7843d10ee9a9080)    | Uses a secondary LLM call to analyze messages for sophisticated injection attacks that evade simpler classifiers.          |
| [**URL Filtering**](/broken/pages/309f46749e98ff15e6d52132476582d0e46982e7)    | Inspect URLs in messages. Allow or deny specific schemes, domains, and ports. Block IP literals and encoded patterns.      |
| [**Base64 Payloads**](/broken/pages/43ed53bca68e4b3d1aeb7fe0ca888c03b1b1423a)  | Detect base64-encoded content that might hide malicious payloads. Configure minimum length and file signature detection.   |

### Text Processing

Normalize or classify content before other rules run.

| Type                                                                             | Description                                                                                                                   |
| -------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| [**Normalization**](/broken/pages/3d868483dff62181bca1fa2636fef459d62d3157)      | Standardize Unicode, whitespace, and encoding to prevent rule-evasion tricks (homoglyphs, zero-width characters, diacritics). |
| [**Language Detection**](/broken/pages/35baca5a0d913a9f93a2e32146628b2b6a3e1381) | Identify the language of the content and optionally block unsupported languages.                                              |

## Best Practices for Rule Ordering

A well-ordered pipeline processes text in layers. Here is a recommended pattern:

| Order Range   | Purpose            | Example Rules                                                  |
| ------------- | ------------------ | -------------------------------------------------------------- |
| **1 -- 5**    | Text normalization | Normalization (clean up Unicode tricks before other rules run) |
| **5 -- 10**   | Whitelisting       | Allow rules for trusted patterns or known-safe content         |
| **10 -- 50**  | PII masking        | Regex, Structured ID, and Dictionary rules                     |
| **50 -- 100** | Threat blocking    | Prompt injection, LLM detection, URL filtering, Base64         |

This ordering ensures that normalization happens first (so attackers cannot bypass rules with encoding tricks), then trusted content is allowed through, then sensitive data is masked, and finally threats are blocked.

## Enforcement Mode

Policies can run in **enforce** or **monitor** mode. In monitor mode, rules still evaluate and their results are recorded, but content is never actually masked or blocked. This is useful for testing new rules before enabling them in production.

See [Enforcement Mode](/broken/pages/1fdf541d9fa5a4a6abccb61802163b5b1165aa0e) for details.

## Next Steps

* [Enforcement Mode](/broken/pages/1fdf541d9fa5a4a6abccb61802163b5b1165aa0e) -- Understand enforce vs. monitor mode.
* [Detecting PII](/broken/pages/d48a8216437a2ca9df0c414b29aab3627ea5b1c4) -- Regex, structured IDs, and dictionary matching.
* [Blocking Threats](/broken/pages/dd999da5021942ff72e2e9d16fc2166acaee8bd0) -- Prompt injection, LLM detection, URL filtering, base64.
* [Text Processing](/broken/pages/e0b4966a4a13ae998f54dc096e30cd346760d03f) -- Normalization and language detection.
* [Policies](/broken/pages/3363b7ce051b58141f1afd0d0e3f380b2a90637e) -- How rules are organized into policies.
