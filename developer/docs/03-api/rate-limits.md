# Rate Limits

Anthropic enforces **rate limits** per API key and usage tier to protect platform stability. CCDV F expects you to know limit types, HTTP **429** behavior, and how to design apps that degrade gracefully under throttling.

**Domain focus:** Applications and Integration (33%) — production reliability and API behavior.

## Learning Objectives

After this page you should be able to:

- Name the main limit dimensions (requests per minute, tokens per minute, and related tier caps).
- Recognize rate-limit responses and distinguish them from other errors.
- Apply backoff, queuing, and workload shaping strategies.
- Choose Message Batches or caching when scenarios emphasize throughput vs realtime.
- Answer exam questions on "what to do when you hit 429."

## Key Ideas

### Limit dimensions (conceptual)

Limits vary by **account tier** and **model**. Public docs organize caps roughly as:

| Dimension | Meaning |
| --- | --- |
| **RPM** (requests per minute) | How many API calls you can make |
| **TPM** (tokens per minute) | Input + output tokens processed per minute |
| **ITPM / OTPM** | Input vs output token splits on some tiers |
| **Concurrent / batch limits** | Caps on parallel batch jobs or special endpoints |

Check your organization's **Limits** page in Console — numbers change as tiers upgrade.

### HTTP 429 Too Many Requests

When exceeded:

- Status **429** with error type indicating rate limit
- Response may include **`retry-after`** header (seconds to wait)
- Body describes which limit was hit (requests vs tokens)

**Not the same as:** 401 (auth), 400 (bad request), 529 (overloaded) — see [error-handling.md](./error-handling.md) and [retries.md](./retries.md).

### Workload patterns vs limits

| Pattern | Limit pressure |
| --- | --- |
| Chat app spike | RPM + TPM |
| Large context uploads | TPM (input-heavy) |
| Agent with many tool rounds | RPM multiplies (one user message → many API calls) |
| Bulk offline jobs | Message Batches separate queue — higher throughput, not realtime |

**Agent exam trap:** One "user turn" may trigger **5–20 API calls** — agent loops consume RPM faster than simple chat.

### Mitigation strategies

1. **Exponential backoff + jitter** on 429 — see [retries.md](./retries.md)
2. **Queue** requests server-side; smooth spikes
3. **Reduce tokens** — prompt caching, shorter history pruning, smaller models for subtasks
4. **Request tier upgrade** for sustained production load
5. **Message Batches** for non-interactive volume (separate from RPM for sync API)
6. **Parallel tool calls** — one API call with multiple tools beats serial calls when the model supports it in one turn

### Client-side vs server-side

- Never expose API keys in browsers — rate limits and abuse risk compound.
- Central gateway tracks usage per tenant; enforce fair queuing in multi-tenant SaaS.

### Streaming and limits

Streaming requests count toward **RPM and TPM** like synchronous calls — streaming does not bypass limits.

## Exam Notes

- **"Getting 429 during agent workflow"** → backoff + queue; also reduce redundant calls (cache tool definitions, don't retry successful turns).
- **"Process 100k prompts overnight"** → Message Batches, not hammering sync API.
- **"Same error after instant retry"** → need exponential backoff respecting `retry-after`.
- **"Hit token limit but few requests"** → TPM issue — shrink context or upgrade tier.
- Distractor: switching models **without** checking that model's separate TPM/RPM caps.

## Production Notes

- Monitor **429 rate** and p95 latency per endpoint; alert before users notice.
- Implement **token bucket** or leaky-bucket client-side limiters aligned to your tier.
- Separate API keys per environment (dev/staging/prod) so testing doesn't exhaust prod quota.
- For agents, cap **max tool iterations** per user request to prevent runaway RPM burn.
- Log `request-id` / `anthropic-request-id` headers for support tickets on limit disputes.
- Pre-count tokens for large PDF/image uploads before sending — avoid predictable TPM failures.

## Common Mistakes

- **Tight retry loops** on 429 (amplifies problem — gets you blocked longer).
- Treating every error as retryable — **400 validation errors** should not backoff-retry.
- Ignoring that **each agent loop iteration** is a billable request toward RPM.
- Using sync API for **massive parallel fan-out** instead of batches or controlled concurrency.
- Hard-coding limit numbers from a blog post instead of **Console Limits** page.

## See Also

- [Retries](./retries.md)
- [Error Handling](./error-handling.md)
- [Pricing](./pricing.md)
- [Message API](./messages-api.md)
- [Agent Loop](../04-agent-engineering/agent-loop.md)
- [Caching](../09-performance/caching.md)
