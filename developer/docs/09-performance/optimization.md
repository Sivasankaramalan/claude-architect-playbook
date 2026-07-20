# Optimization

**Optimization** for Claude applications balances quality, latency, and cost across model choice, prompts, context, caching, tooling, and architecture. This is the capstone page for **Model Selection and Optimization (16.8%)**—expect synthesis questions tying multiple knobs together.

## Learning Objectives

After this page, you should be able to:

- Apply a prioritized optimization workflow (measure → biggest cost/latency → fix → re-eval)
- Select models per task stage in multi-step pipelines
- Optimize prompts, tools, and context jointly—not in isolation
- Use caching, pruning, batching, and routing as a coordinated strategy
- Avoid optimization that hurts correctness without measurement

## Key Ideas

### Optimize in order

1. **Measure** — tokens, latency traces, task success evals
2. **Fix correctness** — broken tools and prompts waste all savings
3. **Reduce unnecessary work** — fewer turns, fewer tools, smaller context
4. **Cheaper/faster paths** — model routing, caching, Haiku for easy steps
5. **Architecture** — parallelize, subagents, async batch for non-interactive

Skipping measurement leads to shaving the wrong 10%.

### Model routing

Not one model per app—one model per **step**:

```text
Input → [Haiku: classify intent]
            │
            ├─ FAQ → [Haiku: answer with retrieval tool]
            ├─ Code → [Sonnet: agent + tools]
            └─ Escalation → [Opus: rare deep analysis]
```

Routing rules can be deterministic (keywords), classifier model, or explicit user tier.

See [Routing](../10-design-patterns/routing.md).

### Prompt and tool optimization

| Lever | Action |
| --- | --- |
| System prompt | Remove redundancy; XML structure |
| Tools | Merge overlapping tools; tighten descriptions |
| Examples | Few-shot only where needed; cache stable examples |
| Output | Tools/schemas instead of verbose prose |
| Thinking | Disable extended thinking unless task requires |

Tool count affects **every turn** in an agent—high ROI to slim manifests.

### Context optimization

Combine [Context Window](./context-window.md), [Pruning](./pruning.md), and [Caching](./caching.md):

- Static → cache breakpoint
- Dynamic → prune + retrieve
- Durable facts → external store

### Agent loop efficiency

- Terminate on `stop_reason: end_turn`—don't extra call
- Cap max turns
- Avoid re-asking model what code can validate
- Subagents for isolated heavy research ([Decomposition](../10-design-patterns/decomposition.md))

### Cost-quality Pareto frontier

Document acceptable tradeoffs:

- Support chat: favor Sonnet + streaming + cache
- Offline codegen batch: Batches + Sonnet, no stream
- Safety-critical: Opus + guardrails + human approval (cost secondary)

Re-run [Evaluations](../08-production/evaluations.md) after each optimization—speed ups that drop pass rate are regressions.

## Exam Notes

- Optimization stem with many knobs → pick **highest impact, simplest** fix (exam golden rule)
- Repeated static prefix cost → **prompt caching**
- Long chat slowdown → **prune/summarize**, not bigger window only
- Easy classification step → **Haiku**, not Opus
- Bulk deferrable jobs → **Message Batches**
- Quality dropped after change → **eval regression**, revert or tune

## Production Notes

### Optimization playbook

- [ ] Baseline: tokens/task, P95 latency, eval pass rate
- [ ] Token breakdown: system, tools, history, user
- [ ] Cache hit rate dashboard
- [ ] Model mix report (% calls by model)
- [ ] A/B prompt versions with eval gate
- [ ] Quarterly tool manifest audit

### Extended thinking

Use when task needs deliberate reasoning and latency budget allows. Disable for high-QPS paths—cost and latency both rise.

## Common Mistakes

- **Optimize tokens before fixing tool bugs** — failed retries multiply cost
- **Single global model** — overpaying on easy steps
- **Shorter prompt that omits constraints** — eval failures
- **Cache without stable prefix** — write-heavy, no read benefit
- **Parallelize dependent tools** — race conditions and wrong answers

## See Also

- [Cost Control](./cost-control.md)
- [Latency](./latency.md)
- [Caching](./caching.md)
- [Pruning](./pruning.md)
- [Claude Models](../01-foundations/claude-models.md)
- [Production Patterns](../10-design-patterns/production-patterns.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
