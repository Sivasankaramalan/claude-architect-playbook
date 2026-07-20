# Retries

Production Claude integrations must **retry transient failures** without duplicating side effects or amplifying rate-limit pressure. CCDV F tests which errors retry, backoff strategy, and idempotency awareness.

**Domain focus:** Applications and Integration (33%) — reliability implementation.

## Learning Objectives

After this page you should be able to:

- Classify HTTP/API errors as retryable vs non-retryable.
- Implement exponential backoff with jitter for 429 and 529 responses.
- Respect `retry-after` headers when present.
- Avoid retrying failed tool executions that mutate external state without idempotency.
- Choose SDK built-in retry behavior vs custom policies.

## Key Ideas

### Retryable vs non-retryable

| Status / type | Retry? | Notes |
| --- | --- | --- |
| **429** rate limit | Yes | Backoff; honor `retry-after` |
| **529** overloaded | Yes | Temporary capacity; backoff |
| **500**, **502**, **503** | Often yes | Transient server/network |
| **408** timeout | Careful yes | Check if request may have partially succeeded |
| **400** bad request | **No** | Fix payload (schema, content) |
| **401**, **403** | **No** | Auth/permissions |
| **404** | **No** | Wrong endpoint/resource |
| **413** too large | **No** | Shrink request |

**Exam rule:** If the fix is "change the JSON" or "fix API key," **don't retry** — fix the request.

### Exponential backoff with jitter

Standard pattern:

```
wait = min(cap, base * 2^attempt) + random_jitter
```

- **Jitter** prevents thundering herd when many clients retry simultaneously.
- Cap max wait (e.g., 60s) and max attempts (e.g., 5–8).
- For **429**, prefer **`retry-after`** from response when provided.

### Idempotency

- Retrying the **same Messages API POST** may produce **two different assistant messages** (LLM nondeterminism) — that's usually OK for read-only Q&A.
- Retrying **after your tool already charged a credit card** is dangerous — use idempotency keys on **your** side effects, not just API retries.
- Anthropic SDKs may support **idempotency keys** for safe network retries of identical requests — use for payment-critical flows.

### Retries in agent loops

Agent workflows compound retry decisions:

```
API call → tool_use → execute tool (maybe fails) → tool_result → API call
```

| Failure point | Retry strategy |
| --- | --- |
| API 529 before response | Retry API call with backoff |
| Tool transient error (HTTP 503) | Retry tool OR return `tool_result` with `is_error: true` |
| Tool permanent error | `is_error: true`, let model adapt — don't infinite retry |
| Partial stream disconnect | Resume UX or retry whole turn — log what was shown |

**Don't** blindly retry the entire agent loop from scratch if history already mutated external systems.

### SDK vs manual

Official SDKs (Python, TypeScript) often include **default retry policies** for 429/529 — know they exist on exam ("use SDK defaults" vs hand-rolled loops).

Configure:

- `maxRetries`
- timeout values
- custom retry predicates

### Parallel requests under retry

When many workers retry 429 simultaneously, limits worsen — combine backoff with **client-side rate limiting** — [rate-limits.md](./rate-limits.md).

## Exam Notes

- **"Intermittent 529 errors"** → exponential backoff retries.
- **"400 invalid tool schema"** → fix schema, no retry.
- **"429 immediately after limit hit"** → backoff + `retry-after`, not identical instant retry.
- **"Tool call failed due to network blip"** → retry tool or surface `is_error` — scenario may prefer `is_error` so model can explain to user.
- **"Duplicate orders on retry"** → idempotency on tool layer, not "disable retries entirely."

## Production Notes

- Log **attempt number**, error type, and `request-id` on every retry.
- Circuit breakers: stop calling API when error rate spikes — fail fast with user message.
- Distinguish **user-facing retry** ("try again" button) from **internal automatic retry**.
- Set **timeouts** slightly below client UX tolerance; retry only idempotent API reads/creates.
- For streaming, define policy: abort and retry whole generation vs resume — most apps retry whole turn.
- Test retry behavior in staging with fault injection.

## Common Mistakes

- Retrying **400** validation errors indefinitely.
- Zero jitter → synchronized retries amplify outages.
- Retrying **tool side effects** without idempotency keys.
- Counting **each agent iteration retry** against user patience — surface progress.
- Using retries instead of **requesting tier upgrade** for chronic 429.

## See Also

- [Error Handling](./error-handling.md)
- [Rate Limits](./rate-limits.md)
- [Messages API](./messages-api.md)
- [Tool Results](../04-agent-engineering/tool-results.md)
- [Agent Loop](../04-agent-engineering/agent-loop.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
