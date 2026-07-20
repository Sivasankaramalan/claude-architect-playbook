# Token Pricing

How Claude billing works in tokens, how to estimate cost, and how pricing-related API features (caching, batches, thinking tokens) affect your architecture decisions.

> **Verify on official docs:** Per-million-token rates, cache pricing, and model IDs change. Always check [Anthropic pricing](https://www.anthropic.com/pricing) and the [API pricing docs](https://docs.anthropic.com/en/docs/about-claude/pricing) before budgeting or answering cost questions on the exam.

**CCDV F domains:** Model Selection and Optimization · Applications and Integration · Prompt and Context Engineering

## Learning Objectives

After this page, you should be able to:

- Explain input vs output token billing and where each appears in API responses
- Estimate cost mentally using per-million-token rates (directionally)
- Describe how **prompt caching**, **Message Batches**, and **extended thinking** alter cost
- Use the token counting API and `usage` fields for observability
- Apply cost-control patterns without sacrificing required quality

## Key Ideas

### Billing Unit: Tokens

Anthropic charges by **tokens processed**, not by API calls or wall-clock time.

| Usage type | Billed as | Notes |
| --- | --- | --- |
| **Input tokens** | Standard input rate | System prompt, history, tools, documents, images |
| **Output tokens** | Output rate (often higher than input) | Generated text and visible completion |
| **Cache write tokens** | Cache creation rate (when applicable) | First time a cacheable prefix is stored |
| **Cache read tokens** | Reduced cache read rate | Subsequent hits on same prefix |
| **Thinking tokens** | Model-specific (extended thinking) | Internal reasoning—not always shown to users |

Every Messages response includes:

```json
"usage": {
  "input_tokens": 1523,
  "output_tokens": 287
}
```

With caching enabled, expect additional fields such as `cache_creation_input_tokens` and `cache_read_input_tokens`—exact names per current API version.

### Typical Price Pattern (Illustrative Only)

Rates differ by model tier and change over time. The **relationship** is what exams test:

| Tier | Input $/MTok | Output $/MTok | Pattern |
| --- | ---: | ---: | --- |
| Opus-class | Highest | Highest | Pay for depth |
| Sonnet-class | Mid | Mid | Default balance |
| Haiku-class | Lowest | Lowest | Volume workloads |

**Do not memorize stale numbers.** If an exam question cites specific dollars, use those numbers in the question. Otherwise, reason ordinally: Haiku < Sonnet < Opus.

### Cost Formula

```
session_cost ≈ (input_tokens × input_rate) + (output_tokens × output_rate)
             + cache_write + cache_read + thinking_tokens (if any)
```

Convert tokens to millions when applying $/MTok rates:

```
cost = (tokens / 1_000_000) × price_per_million
```

### What Consumes the Most Tokens?

| Source | Often heavy? | Mitigation |
| --- | --- | --- |
| Long chat history | Yes | Summarize, sliding window, store facts externally |
| Tool definitions | Yes | Remove unused tools; concise descriptions |
| Pasted documents | Yes | RAG snippets vs full corpus |
| Large JSON in prompts | Yes | Pass IDs; fetch via tools |
| Verbose system prompts | Moderate | Tighten wording; cache stable prefix |
| Image inputs | Yes | Resize; crop; use text extraction when enough |

### Prompt Caching Economics

Prompt caching trades **upfront cache write cost** for **cheaper reads** on repeated prefixes (system prompts, tool defs, long docs).

```
Request 1: pay full input + cache WRITE on eligible prefix
Request 2+: pay reduced CACHE READ for matching prefix + new suffix tokens
```

Best when: many requests share an identical long prefix (support bots, agent templates, shared tool catalogs).

Not worth it when: every request is unique one-shots with no shared prefix.

See [prompt caching](../02-prompt-engineering/prompt-caching.md).

### Message Batches Discount

The Batches API processes jobs asynchronously—public docs have advertised ~**50% discount** vs synchronous Messages for eligible workloads. Tradeoff: latency (minutes to hours), not for live chat.

Exam cue: **"Process 50,000 invoices overnight"** → Batches + efficient model.

### Extended Thinking Cost

Extended thinking adds **thinking tokens** billed separately from visible output. Improves hard tasks but can dominate cost if overused. Enable selectively on routes that need deep reasoning.

### Model Selection as Cost Lever

Before micro-optimizing prompts, pick the **lowest tier** that meets quality SLAs:

```
Router (Haiku) → Main agent (Sonnet) → Formatter (Haiku)
```

Often beats single Opus thread on cost at similar user-visible quality.

### Token Counting API

Call the counting endpoint with the same payload you plan to send—includes system, tools, and multimodal blocks. Use in CI to prevent context overflow regressions.

### Observability

Track per feature / customer / model:

- Tokens in/out per request
- Cache hit ratio
- Cost per successful task (not per call)
- Truncation rate (`stop_reason: max_tokens`)

## Exam Notes

- **Output tokens usually cost more than input**—long answers cost more than long prompts (directionally).
- **Caching** helps repeated identical prefixes—not one-off unique prompts.
- **Batches** = async + discount + wrong for real-time UX.
- **Thinking tokens** add cost even if user never sees them.
- Questions asking "reduce cost without changing model" → shorten context, cache system prompt, summarize history, remove unused tools.
- Questions allowing architecture change → smaller model, routing, RAG instead of full doc paste.

## Production Notes

- Set per-user/org budgets and rate limits in *your* layer—API keys alone don't cap spend.
- Alert when p95 input tokens spike (often a logging bug resending huge history).
- Default `max_tokens` to task-appropriate ceilings—not always model maximum.
- Precompute and cache embeddings/RAG offline; don't re-embed entire DB per chat turn.
- Review tool schemas quarterly—bloated descriptions are silent cost leaks.
- Reconcile provider invoices against logged `usage` monthly.

## Common Mistakes

- Estimating cost from character count instead of token counting
- Enabling prompt caching on entirely unique prompts (pay write, no reads)
- Using Opus with extended thinking for high-volume simple classification
- Sending full conversation forever without summarization
- Ignoring cache read metrics—assuming caching "didn't work" after one request
- Setting `max_tokens` to max context "just in case"

## See Also

- [LLM Basics](./llm-basics.md)
- [Claude Models](./claude-models.md)
- [Prompt Caching](../02-prompt-engineering/prompt-caching.md)
- [Cost Control](../09-performance/cost-control.md)
- [Context Window](../09-performance/context-window.md)
- [Optimization](../09-performance/optimization.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
