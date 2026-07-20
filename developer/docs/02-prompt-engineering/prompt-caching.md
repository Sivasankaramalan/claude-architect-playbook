# Prompt Caching

Using Anthropic **prompt caching** to reuse long stable prompt prefixes across requests—cutting cost and latency for repeated system prompts, tool definitions, and documents.

> **Verify on official docs:** Cache minimums, TTL, pricing, and header requirements change. Read [prompt caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching) before implementing or relying on exam specifics.

**CCDV F domains:** Prompt and Context Engineering · Model Selection and Optimization · Applications and Integration

## Learning Objectives

After this page, you should be able to:

- Explain how cache reads and writes appear in API requests and `usage`
- Place `cache_control` breakpoints on stable vs volatile content
- Identify workloads that benefit from caching vs one-off prompts
- Relate caching to [token pricing](../01-foundations/token-pricing.md) and performance tuning
- Avoid cache-invalidating mistakes in production and on exams

## Key Ideas

### Problem Caching Solves

Many apps send the **same long prefix** every request:

- System prompt + safety policy
- Large tool catalog JSON
- RAG knowledge base snapshot
- Few-shot example library

Without caching, you pay full input token price and processing time on that prefix **every call**. Prompt caching stores eligible prefix segments so subsequent matching requests pay **reduced cache read pricing** and often lower latency.

### How It Works (Conceptual)

```
Request 1: [==== cached prefix ====][ volatile suffix ]
           ↑ cache WRITE (creation tokens)

Request 2: [==== same prefix ======][ new user message ]
           ↑ cache READ (cheaper)     ↑ only this part full price
```

Cache hits require **byte-identical prefix** up to the breakpoint—including whitespace and tool ordering.

### `cache_control` Breakpoints

Mark cacheable content blocks with `cache_control`:

```json
{
  "system": [
    {
      "type": "text",
      "text": "Long stable system instructions...",
      "cache_control": { "type": "ephemeral" }
    }
  ],
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "Huge reference manual chunk...",
          "cache_control": { "type": "ephemeral" }
        },
        {
          "type": "text",
          "text": "User question: What is section 3 about?"
        }
      ]
    }
  ]
}
```

Place breakpoints on **stable** blocks; keep **volatile** user input **after** the last breakpoint without cache marker.

### Usage Fields

Responses may include (exact names per API version):

| Field | Meaning |
| --- | --- |
| `input_tokens` | Non-cache input |
| `cache_creation_input_tokens` | Tokens written to cache |
| `cache_read_input_tokens` | Tokens read from cache (discounted) |
| `output_tokens` | Generated output (unchanged by caching) |

First request after cache expiry may write again—monitor read/write ratio in dashboards.

### Eligibility Requirements (Check Docs)

Typical constraints (verify current docs):

- Supported **models** only
- Minimum token threshold per cache block (~1024 tokens class—confirm exact number)
- **TTL** (e.g., ephemeral ~5 minutes rolling on activity—confirm)
- Applies to eligible content block types (text; tool defs per docs)

Exams may test **concept** ("reuse stable system prompt") even if numbers change.

### What to Cache

| Good cache candidates | Poor cache candidates |
| --- | --- |
| Static system instructions | Unique user message each time |
| Tool definitions (stable) | Rapidly edited prompts |
| Shared RAG corpus for a tenant | Personal chat history tail |
| [Few-shot](./few-shot.md) libraries | One-shot anonymous queries |

### Multi-Tenant Patterns

| Pattern | Approach |
| --- | --- |
| Shared global prefix | Single cache across customers |
| Per-tenant knowledge | Tenant-specific cached block + shared policy block (multiple breakpoints) |
| Highly unique chats | Caching may never hit—skip complexity |

### Caching vs Other Optimizations

| Technique | Saves |
| --- | --- |
| **Prompt caching** | Repeated prefix tokens |
| **Shorter prompts** | All tokens |
| **Smaller model** | Output + input rates |
| **Message Batches** | Async bulk discount (different feature) |
| **Summarized history** | Volatile tail tokens |

Use caching **with** context pruning—not instead of good architecture.

### Cache Invalidation

Prefix changes when you:

- Edit system prompt text
- Reorder tools or change descriptions
- Toggle few-shot examples
- Upgrade model snapshot (cache not portable)

Version prompts (`system_prompt_v17`) and bump cache intentionally.

## Exam Notes

- **"Same long system prompt on every support ticket"** → prompt caching with breakpoint on system block.
- **"Every query is unique with no shared prefix"** → caching wrong answer.
- First request pays **write** cost; benefit appears on **subsequent** requests—single-call scenarios won't show savings.
- Caching ≠ storing conversation **memory** in Anthropic's servers for free—it's prefix reuse within TTL/eligibility rules.
- Pair with **Sonnet/Opus high-volume** scenarios in cost questions—not necessarily tiny one-liner Haiku calls below minimum thresholds.

## Production Notes

- Metric: `cache_read / (cache_read + cache_creation)` per route.
- Put cached content early in payload; volatile content last.
- Warm caches after deploy ( scripted canary requests) before traffic spike.
- Document cache TTL for on-call—"sudden cost spike" may be mass cache refresh after prompt deploy.
- Test in staging with identical prefixes—verify read tokens > 0 on second call.
- Do not cache secrets you wouldn't send twice anyway—caching doesn't improve security.

## Common Mistakes

- Caching volatile user messages (no hits, pay writes)
- Minor whitespace edit invalidates entire prefix
- One breakpoint mixing stable + dynamic content in same block
- Expecting cache hits across different model IDs
- Ignoring minimum token size—short system prompts won't cache
- Confusing prompt caching with Claude **memory** / session features in products

## See Also

- [Token Pricing](../01-foundations/token-pricing.md)
- [Caching](../09-performance/caching.md)
- [Cost Control](../09-performance/cost-control.md)
- [Prompting](./prompting.md)
- [Few-Shot Examples](./few-shot.md)
- [API Overview](../01-foundations/api-overview.md)
