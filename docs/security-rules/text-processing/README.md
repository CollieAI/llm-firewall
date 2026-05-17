---
icon: input-text
---

# Text Processing

Text processing rules transform and analyze message content before other rules evaluate it. Place these rules early in your rule order so that downstream rules (PII detection, threat blocking) work with clean, standardized text.

### Rule Types <a href="#rule-types" id="rule-types"></a>

#### [Normalization](normalization.md)

Standardizes text to defeat obfuscation techniques. Handles Unicode normalization, homoglyph mapping, zero-width character removal, and more.

**Use for:** Preventing evasion via Cyrillic lookalikes, zero-width characters, Unicode tricks, and whitespace manipulation.

#### [Language Detection](language-detection.md)

Identifies the language of a message and optionally restricts which languages are allowed through the pipeline. Shares detected language context with downstream rules.

**Use for:** Enforcing language policies, routing messages to language-specific models, blocking unsupported languages.

### Recommended Placement <a href="#recommended-placement" id="recommended-placement"></a>

Text processing rules should run early in the pipeline. However, ML-based rules (prompt injection, LLM detection) should run **before** normalization because they are trained on natural text.

A typical ordering:

| Order | Rule Type                        |
| ----- | -------------------------------- |
| 1     | Language Detection               |
| 2-3   | Prompt Injection / LLM Detection |
| 4-5   | Normalization                    |
| 6+    | URL Filtering, Base64, PII rules |

This ensures language context is available for ML model routing, ML models see the original text, and all subsequent rules benefit from normalized input.
