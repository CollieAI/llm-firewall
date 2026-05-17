---
icon: signal-stream
---

# Streaming

CollieAI supports Server-Sent Events (SSE) streaming, matching the OpenAI streaming format. Your existing streaming code works without modification - just point it at CollieAI.

{% hint style="warning" %}
**Important:** CollieAI uses **buffered streaming** (also known as filtered streaming). The model's response is accumulated internally, security rules are applied to the complete text, and then the filtered result is delivered to your application in SSE format. This means the wire format is standard OpenAI streaming, but the first token is delayed until the full response has been received and filtered. See [How Buffered Streaming Works](streaming.md#how-buffered-streaming-works) for details.
{% endhint %}

## Enabling Streaming

Set `stream: true` in your request:

```bash
curl -X POST "https://api.collieai.com/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer clai_your_api_key_here" \
  -d '{
    "model": "gpt-4o-mini",
    "messages": [{"role": "user", "content": "Tell me a short story."}],
    "stream": true
  }'
```

## SSE Format

Each chunk is delivered as a `data:` line in standard SSE format. Here is an abbreviated example of what the stream looks like on the wire:

```
data: {"id":"chatcmpl-9a1b2c","object":"chat.completion.chunk","created":1709145600,"model":"gpt-4o-mini","choices":[{"index":0,"delta":{"role":"assistant","content":""},"finish_reason":null}]}

data: {"id":"chatcmpl-9a1b2c","object":"chat.completion.chunk","created":1709145600,"model":"gpt-4o-mini","choices":[{"index":0,"delta":{"content":"Once"},"finish_reason":null}]}

data: {"id":"chatcmpl-9a1b2c","object":"chat.completion.chunk","created":1709145600,"model":"gpt-4o-mini","choices":[{"index":0,"delta":{"content":" upon"},"finish_reason":null}]}

data: {"id":"chatcmpl-9a1b2c","object":"chat.completion.chunk","created":1709145600,"model":"gpt-4o-mini","choices":[{"index":0,"delta":{"content":" a"},"finish_reason":null}]}

data: {"id":"chatcmpl-9a1b2c","object":"chat.completion.chunk","created":1709145600,"model":"gpt-4o-mini","choices":[{"index":0,"delta":{"content":" time"},"finish_reason":null}]}

data: {"id":"chatcmpl-9a1b2c","object":"chat.completion.chunk","created":1709145600,"model":"gpt-4o-mini","choices":[{"index":0,"delta":{},"finish_reason":"stop"}]}

data: [DONE]
```

The first chunk typically contains the `role` field in `delta`. Subsequent chunks contain incremental `content`. The final chunk has an empty `delta` and a `finish_reason` of `"stop"`. The stream ends with `data: [DONE]`.

## Python Streaming Example

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.collieai.com/v1",
    api_key="clai_your_api_key_here"
)

stream = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Tell me a short story about a brave dog."}
    ],
    stream=True
)

for chunk in stream:
    content = chunk.choices[0].delta.content
    if content:
        print(content, end="", flush=True)

print()  # Newline after the stream completes
```

## How Buffered Streaming Works

CollieAI uses a **buffer-filter-replay** approach for all streaming requests. This is necessary to enforce outbound security rules, which require the complete response text before they can make a decision (e.g. detecting PII, blocking threats, or masking sensitive content across the full output).

```
Client request
      │
      ▼
Inbound filtering (applied before the stream starts)
      │
      ▼
OpenAI (streams tokens) ──▶ CollieAI buffers full response
                                    │
                                    ▼
                             Apply outbound rules
                                    │
                          ┌─────────┼─────────┐
                          ▼         ▼         ▼
                       Allowed    Masked    Blocked
                          │         │         │
                          ▼         ▼         ▼
                     Re-stream  Re-stream  Send error
                     original   with       chunk and
                     chunks     masked     close stream
                                content
```

{% stepper %}
{% step %}
### Inbound filtering

Security rules are applied to the user's messages before the request is sent to OpenAI. This step does not affect streaming latency.
{% endstep %}

{% step %}
### Accumulate

CollieAI receives the full streamed response from OpenAI internally. The client does not receive any tokens during this phase.
{% endstep %}

{% step %}
### Apply outbound rules

All active outbound rules are applied to the complete response text.
{% endstep %}

{% step %}
### Replay

The filtered result is delivered to the client in SSE format:

* **If allowed** — The original chunks are replayed as-is.
* **If masked** — The response is reconstructed with masked content (e.g. `[EMAIL_REDACTED]`, `[PHONE_REDACTED]`) and re-streamed.
* **If blocked** — An error chunk is sent and the stream is closed immediately.
{% endstep %}

{% step %}
### Logging

The request, response, and triggered rules are logged for monitoring and analytics.
{% endstep %}
{% endstepper %}

### Latency Impact

Because the full response must be accumulated before filtering, there is a delay before the first token reaches your application. This delay equals the time it takes OpenAI to generate the complete response plus the time to apply your outbound rules.

For typical responses (a few hundred tokens), this is a few seconds. For longer generations (e.g. code generation, long-form content), the delay can be 10-30 seconds depending on the model and output length.

Once filtering is complete, the buffered chunks are delivered rapidly.

### Why Buffered Streaming?

Outbound security rules (PII detection, threat blocking, content masking) need to see the full response to make accurate decisions. For example:

* A credit card number might be split across two sentences
* A prompt injection response might only be identifiable from the full context
* Masking requires knowing all matches before producing the final output

Buffering the response is the only way to guarantee that security policies are enforced correctly. This is a deliberate trade-off: **security completeness over time-to-first-token**.

### Blocked Response During Streaming

If an outbound rule blocks the response, the stream will contain an error event instead of content:

```
data: {"error":{"message":"Response blocked by security policy","type":"policy_violation","code":"response_blocked"}}

data: [DONE]
```

Your application should handle this case. The OpenAI SDKs will raise an `APIError` when they encounter an error chunk in the stream.

## Best Practices

* **Set appropriate client timeouts.** The buffer-filter-replay approach means the first token is delayed by the full generation time plus filtering. Set your client timeout to at least the expected generation time plus 10-15 seconds. For long outputs, consider increasing to 60-120 seconds.
* **Handle stream errors gracefully.** A stream can be interrupted by a policy violation at any point. Always wrap your streaming loop in a try/except (Python) or try/catch (Node.js) block. See [Error Handling](/broken/pages/f519424725e6a060aa446fb73674ebd70d4b8a5c) for details.
* **Use non-streaming for short responses.** If your use case involves short, predictable outputs (e.g. classification, extraction), non-streaming mode (`stream: false`) may provide a simpler integration with similar perceived latency, since the buffering delay is minimal for short responses.
* **Use streaming for long responses.** For longer generations, streaming still provides a better experience than non-streaming — once the buffer is released, tokens arrive rapidly, and your application can begin rendering immediately rather than waiting for the full JSON response.
* **Test with your outbound rules.** Test the streaming behavior in your development environment first. The buffered approach means the perceived latency profile differs from a direct OpenAI stream.
* **Consider `stream_options` for token usage.** If you need token usage information in streaming mode, pass `stream_options: {"include_usage": true}` in your request. The final chunk will include a `usage` field.

```python
stream = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Hello!"}],
    stream=True,
    stream_options={"include_usage": True}
)
```

## Streaming vs Non-Streaming Comparison

| Aspect                   | Non-streaming                        | Streaming (buffered)                  |
| ------------------------ | ------------------------------------ | ------------------------------------- |
| Wire format              | Single JSON response                 | SSE chunks (`data:` lines)            |
| Time to first token      | After full generation + filtering    | After full generation + filtering     |
| Delivery after filtering | All at once                          | Chunk by chunk (rapid replay)         |
| Client compatibility     | Any HTTP client                      | SSE-capable client required           |
| Best for                 | Short responses, simple integrations | Long responses, progressive rendering |
