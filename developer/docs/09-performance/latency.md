# Latency

**Latency** in Claude applications is perceived end-to-end: network, queueing, time to first token (streaming), full generation, tool execution, MCP hops, and validation. **Model Selection and Optimization (16.8%)** and **Applications and Integration (33.1%)** frequently test latency tradeoffs.

## Learning Objectives

After this page, you should be able to:

- Decompose latency into model, tool, and orchestration components
- Choose streaming, model tier, and parallelization appropriately
- Reduce time-to-first-byte for interactive UX without sacrificing correctness
- Set realistic SLOs and identify bottlenecks with traces
- Answer exam questions on Haiku vs Sonnet vs Opus speed tradeoffs

## Key Ideas

### Latency stack

```text
Client
  → Your API gateway
      → Orchestrator / agent loop
          → Messages API (queue + inference)
          → Tool / MCP / DB
          → Validation
  ← Response
```

Optimize the **largest slice** first—often tools, not the model.

### Time to first token (TTFT)

**Streaming** emits partial output before completion:

- Improves **perceived** latency for chat UIs
- Does not reduce total generation time for long answers
- Essential for interactive products; optional for batch

Use `stream: true` when user waits on screen.

See [Streaming](../03-api/streaming.md).

### Model selection for speed

| Model class | Typical use |
| --- | --- |
| **Haiku** | Routing, classification, simple extraction, high-volume steps |
| **Sonnet** | Balanced production default, coding, agents |
| **Opus** | Hard reasoning, complex synthesis—slower, higher cost |

**Multi-model pipelines**: Haiku classifies intent → Sonnet executes—beats Opus-only for mixed workloads.

Extended thinking modes increase latency intentionally for quality—use only when stem requires deep reasoning.

### Tool and MCP latency

Each agent turn may add:

- 1+ model calls
- N serial tool calls (bad) vs parallel where safe (good)
- Cold-start MCP servers

Patterns:

- Cache tool results with TTL
- Parallelize independent reads
- Move heavy compute out of synchronous user path (async jobs)

### Context size and latency

Larger input contexts increase prefill time. **Prune**, **retrieve**, and **cache** static prefixes to shrink dynamic portion.

### Retries and latency

Exponential backoff on 429/529 adds latency but prevents failure storms. Cap retries; shed load under saturation.

See [Retries](../03-api/retries.md).

### Message Batches

Non-interactive bulk work should not compete with user-facing latency SLOs—use batches for offline processing.

## Exam Notes

- Interactive chat feels slow → **streaming** first for UX
- Simple triage step in pipeline → **Haiku-class**, not Opus
- Slow because 5 tools run serially → **parallelize** independent tools
- Same quality, repeated static prefix → **prompt caching** (latency + cost)
- Bulk overnight reports → **Message Batches**, not streaming sync API

## Production Notes

### Latency optimization checklist

- [ ] Trace P50/P95 per span (model vs tools)
- [ ] Stream user-facing responses
- [ ] Route steps to smallest sufficient model
- [ ] Parallel independent tool calls
- [ ] Keep tool payloads small
- [ ] Prompt cache on static prefixes
- [ ] Timeout tools; fail fast with structured errors
- [ ] Avoid unbounded agent turns

### SLO framing

Separate **TTFT** (streaming) from **task completion** (agent). Agent SLOs include tool time—don't blame model alone.

## Common Mistakes

- **Opus for every step** — cost and latency multiply
- **Streaming assumed to halve total time** — only helps perception until short replies
- **Serial tools that could parallelize**
- **Giant context every turn** — slow prefill
- **No tool timeouts** — one hung DB query blocks session

## See Also

- [Optimization](./optimization.md)
- [Caching](./caching.md)
- [Parallelization](../10-design-patterns/parallelization.md)
- [Routing](../10-design-patterns/routing.md)
- [Streaming](../03-api/streaming.md)
- [Claude Models](../01-foundations/claude-models.md)
