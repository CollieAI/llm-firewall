---
icon: id-card
---

# Detecting PII

CollieAI provides three rule types for detecting and redacting personally identifiable information (PII) in LLM messages. Each approach is optimized for different kinds of sensitive data.

## Rule Types

### [Regex Patterns](regex-patterns.md)

Pattern-based detection using regular expressions. Best for data that follows predictable text formats.

**Use when detecting:**

* Email addresses
* Phone numbers
* Social Security Numbers
* API keys and secrets
* Any data matching a known text pattern

### [Structured IDs](structured-ids.md)

Checksum-validated detection for identifiers with built-in validation algorithms. Produces fewer false positives than regex alone.

**Use when detecting:**

* Credit card numbers (Luhn algorithm)
* IBAN numbers (MOD-97-10)
* SWIFT/BIC codes (ISO 9362)
* National identity numbers with checksum validation

### [Dictionary Matching](dictionary-matching.md)

Fast multi-pattern matching against word lists. Ideal for organization-specific terms and custom blocklists.

**Use when detecting:**

* Company-specific terms and internal project names
* Custom PII keyword lists
* Known entity names (people, organizations)
* Profanity and prohibited terms

## Choosing the Right Rule Type

| Scenario                          | Recommended Rule Type             |
| --------------------------------- | --------------------------------- |
| Emails, phone numbers, SSNs       | Regex Patterns                    |
| Credit card numbers, IBANs        | Structured IDs                    |
| Custom word lists, internal terms | Dictionary Matching               |
| API keys, tokens, secrets         | Regex Patterns (built-in presets) |
| National IDs with checksums       | Structured IDs                    |
| Profanity filtering               | Dictionary Matching               |

You can combine multiple rule types in a single policy. For example, use Regex Patterns for emails and phone numbers, Structured IDs for credit cards, and Dictionary Matching for company-specific terms -- all evaluated in the order you define.
