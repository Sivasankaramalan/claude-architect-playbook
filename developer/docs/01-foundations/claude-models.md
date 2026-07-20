# Claude Models

How Anthropic's Claude model families differ, when to choose each, and how model choice shows up in API requests and production tradeoffs.

> **Verify on official docs:** Model IDs, context windows, pricing, and feature flags (vision, extended thinking, prompt caching) change frequently. Always confirm current names on [Claude models overview](https://docs.anthropic.com/en/docs/about-claude/models) before coding or exam day.

**CCDV F domains:** Model Selection and Optimization · Applications and Integration

## Learning Objectives

After this page, you should be able to:

- Map workload types (latency, cost, reasoning, coding, vision) to appropriate Claude tiers
- Set the `model` parameter correctly and recognize deprecated IDs
- Explain tradeoffs between **quality**, **speed**, **cost**, and **context size**
- Know when **extended thinking**, **prompt caching**, and **Message Batches** affect model choice
- Avoid common exam distractors (e.g., using Opus for every trivial task)

## Key Ideas

### Model Tiers (Mental Model)

Anthropic organizes Claude into tiers optimized for different jobs. Exact IDs evolve, but the **pattern** is stable:

| Tier | Typical use | Strengths | Watch-outs |
| --- | --- | --- | --- |
| **Opus-class** | Hardest reasoning, complex agents, deep analysis | Highest capability, long-horizon tasks | Highest cost and latency |
| **Sonnet-class** | Default for most apps: coding, agents, balanced IQ/speed | Strong reasoning + good speed | Not always cheapest for high volume |
| **Haiku-class** | Classification, routing, extraction, high-volume chat | Fast, cost-efficient | Less depth on multi-step reasoning |

Exam questions usually test **fit**, not memorizing a single "best" model for all cases.

### Choosing a Model: Decision Table

| Scenario | Lean toward | Why |
| --- | --- | --- |
| Sub-100ms routing / intent detection | Haiku-class | Volume + latency |
| Customer support with tools | Sonnet-class | Balance of quality and cost |
| Multi-file codebase refactor agent | Opus or Sonnet (per budget) | Reasoning + tool loops |
| Batch overnight summarization of 10k docs | Haiku or Sonnet + **Message Batches** | Cost at scale |
| Complex math / planning with budget | Opus + extended thinking (if supported) | Depth over speed |
| Real-time UI typing effect | Sonnet/Haiku + **streaming** | Perceived latency |

### The `model` Request Field

Every Messages API call requires:

```json
{
  "model": "claude-sonnet-4-20250514",
  "max_tokens": 1024,
  "messages": [{ "role": "user", "content": "Hello" }]
}
```

Use **snapshot IDs** (dated suffixes) in production for reproducibility. Anthropic publishes aliases (e.g., `claude-3-5-sonnet-latest`) for convenience; exams may mention either—know that **aliases can change underlying snapshots**.

### Context Windows

Models advertise large context windows (e.g., 200k tokens on many current models). Larger context enables:

- Whole codebases or long PDFs in one call
- Long agent trajectories without aggressive summarization

Costs and latency still scale with tokens processed—**large context ≠ free context**. See [context window](../09-performance/context-window.md) and [token pricing](./token-pricing.md).

### Feature Availability by Model

Not every feature exists on every model. Check docs for your target ID:

| Feature | Typical pattern |
| --- | --- |
| **Vision** | Supported on major Sonnet/Opus/Haiku models—verify per ID |
| **Tool use** | Broadly available on recent models |
| **Structured outputs / JSON Schema** | Model-specific; confirm in docs |
| **Prompt caching** | Supported on eligible models; separate cache read/write pricing |
| **Extended thinking** | Newer high-end models; extra thinking tokens billed |
| **Message Batches** | API feature (async 50% discount class)—not a separate "model" |

### Extended Thinking

For difficult reasoning, some models expose **extended thinking**: Claude generates internal reasoning before the user-visible answer. Improves accuracy on hard tasks at higher **latency** and **token cost**. Use when correctness matters more than speed—not for simple FAQ bots.

### Model Selection in Multi-Step Systems

Production apps often use **routing**:

```
User message → Haiku (classify intent) → Sonnet (main agent) → Haiku (format JSON)
```

This pattern optimizes **cost and latency** while keeping quality where it matters. Exams may present this as the best developer answer vs single-model Opus for everything.

### Deprecation and Migration

Anthropic retires older model snapshots. Production checklist:

1. Pin snapshot IDs in config
2. Monitor deprecation notices
3. Regression-test prompts on new snapshots before cutover
4. Update `model` string—behavior can shift slightly

## Exam Notes

- **Simple + high volume → smaller/faster model.** **Complex + low volume → larger model.**
- If the question mentions **overnight bulk processing**, think **Message Batches** + cost-efficient model—not synchronous Opus loops.
- **Streaming** improves UX latency but does not change which model understands the task.
- **Extended thinking** is for hard reasoning—not a substitute for tools/retrieval.
- When options differ only by model tier, re-read the scenario's constraints: **SLA**, **budget**, **accuracy**, **volume**.

## Production Notes

- Externalize `model` as configuration; route by task type or customer tier.
- Benchmark on *your* prompts—public benchmarks ≠ your JSON extraction schema.
- Log model ID per request for cost attribution and debugging regressions after upgrades.
- Pair model choice with **prompt caching** for stable system prompts on Sonnet/Opus workloads.
- Set fallback models only when you accept behavioral differences; test thoroughly.

## Common Mistakes

- Using Opus for all requests "because it's smartest"
- Using Haiku for multi-step agent loops requiring deep planning
- Hardcoding deprecated model strings in source instead of config
- Assuming all models support extended thinking or the same context size
- Ignoring batch/async paths for large offline jobs
- Changing model without re-running evals (silent quality drift)

## See Also

- [LLM Basics](./llm-basics.md)
- [Token Pricing](./token-pricing.md)
- [API Overview](./api-overview.md)
- [Optimization](../09-performance/optimization.md)
- [Cost Control](../09-performance/cost-control.md)
- [Caching](../09-performance/caching.md)
