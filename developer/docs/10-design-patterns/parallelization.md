# Parallelization

**Parallelization** runs independent model calls or tool invocations concurrently to reduce wall-clock latency. Key pattern for **Agents and Workflows (14.7%)** and **Model Selection and Optimization (16.8%)** when tasks decompose cleanly.

## Learning Objectives

After this page, you should be able to:

- Identify which subtasks are safe to parallelize vs must stay serial
- Parallelize tool calls and map-reduce worker patterns correctly
- Merge parallel results without losing consistency
- Respect rate limits and concurrency caps when scaling parallel calls
- Avoid exam answers that parallelize dependent steps

## Key Ideas

### Parallelism opportunities

| Safe to parallelize | Keep serial |
| --- | --- |
| Read-only queries to different APIs | Write then read same record |
| Per-file code analysis | Steps needing prior step output |
| Multiple search queries | Payment then confirmation |
| Independent eval cases | Tool B depends on tool A result |

Dependency graph first; parallelize leaves.

### Parallel tool execution

When model returns multiple `tool_use` blocks with no interdependency, orchestrator may execute tools concurrently:

```text
tool_use: [fetch_user, fetch_orders] → Promise.all / asyncio.gather
```

Still append all `tool_result` messages before next model call.

If tools share resources (DB write locks), serialize.

### Map-reduce with LLMs

```text
       ┌─ worker 1 ─┐
Input ─┼─ worker 2 ─┼─► aggregator (Sonnet/Opus)
       └─ worker 3 ─┘
```

Workers get isolated context; aggregator synthesizes—see [Decomposition](./decomposition.md).

### Parallel model calls

Multiple independent classifications (e.g., moderate 100 comments):

- Batch with concurrency limit (10 at a time)
- Watch **rate limits** and **cost spikes**
- Message Batches for offline bulk, different from live parallel

### Aggregation strategies

- **Structured merge** — union JSON arrays with dedupe keys
- **Synthesis call** — coordinator summarizes worker outputs
- **Voting** — multiple graders; majority wins (eval patterns)

Pass **summaries** to aggregator, not raw worker logs.

## Exam Notes

- Slow agent with independent reads → **parallel tool execution**
- Must not race on same invoice update → **serialize writes**
- Analyze 20 files fast → **map-reduce workers**, not one huge context
- Hit rate limits when parallelizing → **concurrency cap + backoff**
- Parallel subagents → each needs **explicit scoped input**

## Production Notes

### Parallelization checklist

- [ ] Dependency analysis documented
- [ ] Concurrency limit per tenant
- [ ] Idempotent read tools
- [ ] Timeout on whole parallel batch (not per task only)
- [ ] Trace spans show parallel fan-out
- [ ] Aggregator eval for quality regression

## Common Mistakes

- **Parallel writes** to same entity — data corruption
- **Unbounded parallelism** — 429 storms
- **Aggregator context explosion** — feed summaries only
- **Assuming model parallelizes tools** — your executor must
- **Parallel without merge logic** — inconsistent user answer

## See Also

- [Decomposition](./decomposition.md)
- [Supervisor](./supervisor.md)
- [Latency](../09-performance/latency.md)
- [Orchestrator-Workers](../04-agent-engineering/orchestrator-workers.md)
- [Rate Limits](../03-api/rate-limits.md)
