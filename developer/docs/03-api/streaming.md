# Streaming

Streaming returns the model's response incrementally over **Server-Sent Events (SSE)** instead of waiting for the full completion. For CCDV F, know when to enable `stream: true`, how streaming interacts with tool use, and how it differs from synchronous calls and Message Batches.

**Domain focus:** Applications and Integration (33%) — latency UX and implementation patterns.

## Learning Objectives

After this page you should be able to:

- Enable streaming on Messages API requests and consume SSE events in application code.
- Explain how streaming improves perceived latency without changing model behavior.
- Handle streaming events for text, tool use, and message lifecycle (`message_start`, `message_delta`, `message_stop`).
- Choose streaming vs synchronous vs batches for exam scenarios.
- Implement safe cancellation and error handling during an open stream.

## Key Ideas

### Enabling streaming

Set top-level parameter:

```json
{ "model": "claude-sonnet-4-20250514", "max_tokens": 1024, "stream": true, "messages": [...] }
```

The HTTP response is `text/event-stream`, not a single JSON body.

### Event types (conceptual map)

Streaming emits a sequence of events. Exact event names are documented in Anthropic's streaming reference; group them mentally as:

| Phase | Typical events | What you receive |
| --- | --- | --- |
| Message start | `message_start` | Message `id`, initial `usage` hints |
| Content blocks | `content_block_start`, `content_block_delta`, `content_block_stop` | Incremental `text` or structured start of `tool_use` |
| Tool use | deltas on `input` JSON | Partial tool argument JSON — accumulate until block complete |
| Message end | `message_delta` | Final `stop_reason`, output token counts |
| Done | `message_stop` | Stream complete |

**Exam insight:** Streaming delivers the **same final message** as a non-streaming call — only the transport differs. `stop_reason` is still authoritative at the end.

### Text streaming UX

- Append `text_delta` chunks to the UI as they arrive — users see tokens in real time.
- **Time-to-first-token (TTFT)** drops sharply vs waiting for full JSON — primary UX win.
- Buffer management: handle UTF-8 boundaries and markdown partial tokens gracefully in UI layers.

### Streaming + tool use

When the model chooses tools:

1. Stream may emit `tool_use` content blocks with **incremental `input` JSON**.
2. Your client must **accumulate** the full `input` object before executing the tool.
3. After `message_stop`, check `stop_reason`:
   - `tool_use` → execute tools, append `tool_result`, start a **new** request (stream or sync).
   - `end_turn` → display final text to user.

**Critical:** Do not execute tools on partial JSON — wait until the content block completes.

### Parallel tool calls in streams

One assistant turn can stream **multiple** `tool_use` blocks. Accumulate each block independently, then execute all tools before the next API call — same history rules as non-streaming.

### Streaming vs synchronous

| | Streaming | Synchronous |
| --- | --- | --- |
| Latency UX | Excellent (first token fast) | User waits for full response |
| Implementation | SSE parser, state machine | Single JSON parse |
| Tool use | Accumulate block JSON | Full blocks in one response |
| Best for | Chat UI, long answers | Batch processing, simple scripts |
| Token cost | Same | Same |

### Streaming vs Message Batches

- **Streaming:** real-time, interactive, full price tier.
- **Batches:** async bulk jobs, ~50% discount, results within ~24 hours — never for live chat.

## Exam Notes

- **"Improve chat responsiveness"** → `stream: true`, not a different model (unless also asked about quality).
- **"User sees nothing for 10 seconds"** → streaming; distractor "increase max_tokens" does not fix TTFT.
- **Tool call during stream** → still driven by final `stop_reason: tool_use`.
- **Same answer quality** — streaming is not a different model or prompt technique.
- Questions may ask which **event** carries `stop_reason` → `message_delta` at end of stream.

## Production Notes

- Always handle **connection drops**: retry with backoff if no `message_stop` received; consider idempotency for the user message side only (don't duplicate assistant turns).
- **Cancel** streams client-side (abort HTTP) when user navigates away — saves tokens if the API honors early close.
- Don't block the event loop — use async SSE readers in web servers.
- For tool use, log the **assembled** tool input, not every delta.
- Combine streaming with **prompt caching** on stable prefixes — cache hits still apply.
- Server-sent heartbeats / proxies: configure nginx/load balancers for long-lived SSE connections.

## Common Mistakes

- Executing tools on **partial streamed JSON** before `content_block_stop`.
- Treating streaming as **optional for agents** — agents can stream assistant text between tool rounds for better UX.
- Assuming streaming **reduces cost** — it reduces perceived latency, not tokens.
- Parsing the response as normal JSON when `stream: true` (will fail).
- Forgetting to run the **same post-stream logic** as sync (`stop_reason`, history append).
- Using Message Batches when the scenario says **"live dashboard"** or **"typing indicator"**.

## See Also

- [Messages API](./messages-api.md)
- [Agent Loop](../04-agent-engineering/agent-loop.md)
- [Tool Use](../04-agent-engineering/tool-use.md)
- [Error Handling](./error-handling.md)
- [Latency](../09-performance/latency.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
