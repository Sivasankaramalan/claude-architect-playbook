# Caching

**Prompt caching** lets you mark stable prefix portions of a request so Claude reuses precomputed context on subsequent calls—reducing latency and input cost for repeated system prompts, tool definitions, documents, and few-shot examples. This is high-yield for **Model Selection and Optimization (16.8%)** and **Applications and Integration (33.1%)**.

## Learning Objectives

After this page, you should be able to:

- Explain cache read vs cache write tokens and billing impact
- Place cache breakpoints on stable vs dynamic content correctly
- Identify workloads that benefit (agents, RAG with static corpus prefix, repeated tools)
- Avoid cache invalidation from prefix drift
- Monitor cache effectiveness in production

## Key Ideas

### How prompt caching works (conceptual)

1. First request with a **cacheable prefix** pays **cache creation** (write) cost for that prefix
2. Subsequent requests with an **identical prefix** hit **cache read**—faster and cheaper reads
3. Content **after** the breakpoint can change each request (user message, recent history)

Order matters: static content first, dynamic content last.

### Typical cacheable blocks

| Block | Usually cacheable? |
| --- | --- |
| System prompt / policy | Yes |
| Tool definitions | Yes (if stable) |
| Large reference document | Yes (if unchanged) |
| Few-shot examples | Yes |
| Conversation history | Partially (prefix of history if stable) |
| Current user message | No (dynamic) |

Use `cache_control` breakpoints in the Messages API (see official docs for exact parameter shapes).

### Agent loops and caching

Multi-turn agents resend tools + system every turn. Caching the static prefix amortizes cost across turns:

```text
[ system + tools + doc ] ← cache breakpoint
[ turn 1 history ]
[ turn 2 history ]
...
[ latest user message ]
```

Without caching, tool JSON alone can dominate spend.

### Cache invalidation

Cache misses when **any byte** in the cached prefix changes:

- Prompt typo fix
- Tool description edit
- Reordered tools
- Different model (cache tied to model family—verify current rules)
- Timestamp injected into "static" prompt

Version prompts explicitly; expect temporary write cost after deploys.

### Cache vs other optimizations

| Technique | Saves |
| --- | --- |
| Prompt caching | Repeated **identical prefix** input tokens |
| Shorter prompts | All requests |
| Smaller model | Output + input on every call |
| Pruning | Dynamic history portion |
| Message Batches | Bulk pricing/latency profile, not same as prefix cache |

Combine caching with pruning—not either/or.

### Monitoring cache

From API `usage`:

- `cache_creation_input_tokens`
- `cache_read_input_tokens`

Low read ratio after stable workload → prefix drift or breakpoint placement bug.

See [Monitoring](../08-production/monitoring.md).

## Exam Notes

- Repeated large system prompt + tools each turn → **prompt caching**, not "use shorter prompts only"
- Static knowledge base in every request → cache **document prefix**, dynamic query after breakpoint
- Cost and latency for stable prefix → **cache read tokens**
- First request after deploy → expect **cache write** spike (normal)
- Caching does not replace **correct context engineering** — dynamic facts still needed in non-cached suffix

## Production Notes

### Caching checklist

- [ ] Breakpoint after full static system + tools block
- [ ] No dynamic values (time, random IDs) in cached prefix
- [ ] Tool manifest versioned; accept write cost on change
- [ ] Dashboard for read/write ratio and savings estimate
- [ ] Staging test: second identical-prefix call shows cache read > 0
- [ ] Document TTL behavior (cached prefixes expire—know idle timeout)

### When not to cache

- Single-turn-only API with unique input each time
- Constantly mutating tool list
- Tiny prompts where overhead negligible

## Common Mistakes

- **Breakpoint after user message** — nothing stable to cache
- **Dynamic system prompt** ("Today is …") — never hits cache
- **Ignoring cache metrics** — miss regressions after prompt edit
- **Assuming cache persists forever** — expired cache re-writes
- **Caching secrets** — prefix in cache infrastructure; minimize sensitive content in prompts regardless

## See Also

- [Prompt Caching](../02-prompt-engineering/prompt-caching.md)
- [Cost Control](./cost-control.md)
- [Latency](./latency.md)
- [Context Window](./context-window.md)
- [Optimization](./optimization.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
