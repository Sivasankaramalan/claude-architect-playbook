# Error Handling

Robust Claude applications classify API errors, handle tool failures gracefully, and keep agent loops stable. CCDV F emphasizes **what to return to the model** (`is_error`), when to retry, and when to stop.

**Domain focus:** Applications and Integration (33%) + Agents and Workflows (15%).

## Learning Objectives

After this page you should be able to:

- Parse Anthropic error response shape (`type`, `message`, HTTP status).
- Map errors to user-facing messages vs silent retries vs model-recoverable tool errors.
- Return proper `tool_result` blocks with `is_error: true` when tools fail.
- Prevent corrupted conversation history after partial failures.
- Pick the best error-handling pattern in scenario questions.

## Key Ideas

### API error response shape

Errors typically return JSON:

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "messages: roles must alternate..."
  }
}
```

Know common `error.type` values at a high level:

| type | Typical cause |
| --- | --- |
| `invalid_request_error` | Malformed request, bad parameters |
| `authentication_error` | Invalid/missing API key |
| `permission_error` | Key lacks access |
| `not_found_error` | Wrong resource ID |
| `rate_limit_error` | 429 — [rate-limits.md](./rate-limits.md) |
| `overloaded_error` | 529 — retry with backoff |
| `api_error` | Server-side issue — retry cautiously |

### HTTP status quick map

- **400** — your bug; fix request
- **401/403** — credentials/permissions
- **404** — wrong URL or file ID
- **413** — payload too large
- **429** — rate limit → [retries.md](./retries.md)
- **529** — overloaded → retry
- **500+** — server/transient

### Tool execution errors → `tool_result`

When **your tool** fails, still continue the agent loop — return a structured result to the model:

```json
{
  "type": "tool_result",
  "tool_use_id": "toolu_01ABC...",
  "is_error": true,
  "content": "Database timeout after 30s. Query was: SELECT ..."
}
```

| Field | Rule |
| --- | --- |
| `tool_use_id` | **Must match** the `id` from the assistant's `tool_use` block |
| `is_error` | `true` tells the model the tool failed — it can retry, apologize, or ask user |
| `content` | Human-readable or JSON string explaining failure — not empty |

**Without `is_error`**, the model may treat garbage output as successful data — **schema ≠ semantic truth** applies doubly here.

### Successful tool errors vs API errors

| Layer | Handling |
| --- | --- |
| **Anthropic API** down | Retry/backoff; don't append fake assistant message |
| **Your tool** failed | `tool_result` + `is_error: true`; append to history; call API again |
| **Model refusal** (`stop_reason: refusal`) | Don't retry same prompt; adjust or escalate |
| **Validation** of tool output | Your code rejects bad data → `is_error: true` with reason |

### Agent loop error discipline

Correct sequence on tool failure:

1. Assistant message contains `tool_use`(s) — already appended.
2. User message contains matching `tool_result`(s) — include failures with `is_error`.
3. Next API call with **full history**.

**Never** skip the tool_result turn — model expects paired results for every `tool_use`.

### Parallel tool calls with mixed success

If two tools run and one fails:

- Return **both** `tool_result` blocks in one user message
- Failed one: `is_error: true`
- Successful one: normal content
- Model decides next step — may retry failed tool only

### User-facing vs internal errors

- **Internal:** log stack traces, request IDs, tool latency.
- **User:** sanitized message; don't leak API keys or SQL.
- Let model generate user explanation **after** receiving `is_error` tool_result — often better UX than generic "something went wrong."

### Streaming errors

Mid-stream failures may lack final `stop_reason`:

- Don't append partial assistant message as complete — either discard or mark truncated
- Retry policy: often restart turn unless you implement partial persistence

## Exam Notes

- **"Database lookup timed out — what do you send Claude?"** → `tool_result` with `is_error: true`, not a new user chat message.
- **"Model hallucinates success after tool crash"** → missing `is_error` or missing tool_result entirely.
- **"invalid_request_error on messages"** → fix alternating roles / tool_result pairing — not retry 529-style.
- **"Guarantee user sees failure reason"** → structured tool error content + let model summarize; or app-level UI overlay.
- **Parallel tools, one fails** → still return all `tool_use_id` results in one user turn.

## Production Notes

- Central **error taxonomy** in your service: `RETRYABLE`, `USER_FIXABLE`, `FATAL`.
- Never log full prompts containing PII in error trackers — scrub first.
- Attach `anthropic-request-id` to support tickets.
- Set **max tool iterations**; on exhaustion, break loop with graceful user message.
- Validate tool outputs before returning success `tool_result` — return `is_error` for business rule violations.
- Feature-flag risky tools; require human approval on `is_error` recovery paths that mutate data.

## Common Mistakes

- Swallowing tool exceptions without **`tool_result`** — breaks next API call.
- Mismatched **`tool_use_id`** — causes 400 or confused model.
- Retrying **400 invalid_request** errors.
- Putting tool errors in **`assistant` role** instead of `tool_result` in `user` message.
- Returning `is_error: true` but **empty content** — model lacks recovery context.
- Assuming **`stop_reason: end_turn`** means tools succeeded — always verify tool results.

## See Also

- [Retries](./retries.md)
- [Rate Limits](./rate-limits.md)
- [Messages API](./messages-api.md)
- [Tool Results](../04-agent-engineering/tool-results.md)
- [Agent Loop](../04-agent-engineering/agent-loop.md)
- [Debugging](../08-production/debugging.md)
