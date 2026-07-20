# Pricing

Understanding **token-based pricing** helps you pass CCDV F optimization questions and build cost-aware applications. Charges derive from **input tokens**, **output tokens**, and feature-specific modifiers (caching, batches, long context).

**Domain focus:** Applications and Integration (33%) + Model Selection and Optimization (17%).

## Learning Objectives

After this page you should be able to:

- Explain how input vs output tokens are billed on Messages API calls.
- Estimate relative cost drivers in agent, vision, and PDF workflows.
- Identify cost optimizations: model choice, caching, batching, context pruning.
- Interpret `usage` fields in API responses for metering.
- Select the cheapest **correct** solution on exam tradeoff questions.

## Key Ideas

### Core billing unit: tokens

Every Messages API response includes:

```json
"usage": {
  "input_tokens": 1520,
  "output_tokens": 340,
  "cache_creation_input_tokens": 0,
  "cache_read_input_tokens": 0
}
```

- **Input tokens** — everything sent: system prompt, tools JSON, full `messages` history, images/PDFs (encoded as tokens).
- **Output tokens** — model generation including tool `input` JSON in assistant content.

Prices are **per million tokens** and differ by model (Opus > Sonnet > Haiku typically).

### Model tier tradeoffs (exam framing)

| Model class | Cost | Best when |
| --- | --- | --- |
| **Haiku** | Lowest | Classification, routing, high-volume subtasks |
| **Sonnet** | Mid | Balanced coding, agents, production default |
| **Opus** | Highest | Hard reasoning, complex multi-step analysis |

**Developer exam:** pick the **cheapest model that meets reliability requirements** — not Opus by default.

### Feature-specific pricing concepts

| Feature | Cost note |
| --- | --- |
| **Prompt caching** | Pay to write cache once; **discounted reads** on cache hits — see [../09-performance/caching.md](../09-performance/caching.md) |
| **Message Batches** | ~**50% discount** vs sync — async only |
| **Long context** | Higher input rates above threshold (e.g., 200k+) on some models — verify current pricing page |
| **Vision / PDF** | Billed as input tokens for encoded content — large files dominate cost |
| **Tool definitions** | Tool schemas in every request consume input tokens — keep descriptions concise |

### Agent loop cost model

One user message can cost **many API calls**:

```
Call 1: input = system + tools + history + user
Call 2: input = system + tools + history + assistant(tool_use) + user(tool_result)
...
Each call re-sends growing history → input tokens climb
```

**Mitigations:** Haiku for routing, history pruning, caching stable prefixes, parallel tools in one turn.

### Streaming vs sync pricing

**Same token charges** — streaming affects UX, not price per token.

### Structured output / tools

Tool call **arguments** appear in **output tokens**. Forcing verbose tool JSON increases output cost — design minimal schemas.

### What pricing is NOT

- **RPM limits** are throughput, not dollars — but retries waste money — [rate-limits.md](./rate-limits.md).
- **Failed requests** may still incur partial charges depending on failure point — design idempotent retries carefully.

## Exam Notes

- **"Reduce cost for repeated system prompt"** → prompt caching, not shorter user messages alone.
- **"10k overnight summarization jobs"** → Message Batches (discount + appropriate pattern).
- **"Router before expensive model"** → small model (Haiku) classifies, Sonnet/Opus handles hard path.
- **"Agent too expensive"** → prune history, cache tools+system, reduce tool iterations — not "disable tools" if task requires them.
- **Images in every turn** → token-heavy; distractor "switch to streaming" doesn't cut cost.

## Production Notes

- Emit **per-request cost estimates** from `usage` × price table in your observability stack.
- Budget alerts on daily token spend per customer/feature.
- A/B test Haiku vs Sonnet on agent subtasks with quality metrics — optimize $/success, not $/call alone.
- Cache **tool definitions** block when using prompt caching — large schemas add up.
- Store extracted PDF text locally to avoid re-tokenizing full documents.
- Pin model versions for predictable unit economics.

## Common Mistakes

- Optimizing **output max_tokens** alone while ignoring **bloated history** (main agent cost driver).
- Assuming **streaming** or **JSON mode** reduces token billing.
- Using Opus for **simple extraction** Haiku handles with tools + validation.
- Ignoring **cache_read** fields — missing free/discounted input on cache hits.
- Choosing Message Batches for **interactive** features to save money (wrong latency tradeoff).

## See Also

- [Messages API](./messages-api.md)
- [Rate Limits](./rate-limits.md)
- [Token Pricing](../01-foundations/token-pricing.md)
- [Caching](../09-performance/caching.md)
- [Cost Control](../09-performance/cost-control.md)
- [Model Selection](../01-foundations/claude-models.md)
