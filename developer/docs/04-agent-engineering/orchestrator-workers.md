# Orchestrator–Workers

The **orchestrator–workers** pattern splits work between a lead model (orchestrator) that plans and delegates, and worker models or tools that execute subtasks in parallel or sequence. It is a core multi-agent architecture on Claude developer exams.

**Domain focus:** Agents and Workflows (15%) + design pattern recognition.

## Learning Objectives

After this page you should be able to:

- Describe orchestrator vs worker responsibilities.
- Explain when to parallelize workers vs run sequentially.
- Map the pattern to Messages API calls, tools, and subagents.
- Compare orchestrator–workers to manager and graph patterns.
- Choose this pattern in exam scenarios involving decomposition and parallelism.

## Key Ideas

### Roles

| Role | Responsibility |
| --- | --- |
| **Orchestrator** | Understand goal, decompose task, assign subtasks, synthesize results |
| **Worker** | Execute one subtask (search, code, summarize chunk) with narrow context |

Workers can be:

- **Separate Claude calls** with isolated message histories
- **Tools** invoked by orchestrator (`tool_use` → your code spins worker)
- **Subagents** in Claude Code / Agent SDK with own system prompts

### Typical flow

```
User request
    → Orchestrator (plan + delegate)
        → Worker A (parallel)
        → Worker B (parallel)
        → Worker C (parallel)
    → Orchestrator (synthesize)
    → User answer (stop_reason: end_turn)
```

Each worker may run its **own agent loop** internally (tools until `end_turn`).

### Implementation with tool use

Common production mapping:

1. Orchestrator has tools: `spawn_research_worker`, `spawn_code_worker`, `aggregate`.
2. Orchestrator returns `tool_use` for three workers — **parallel tool calls** in one turn when independent.
3. Your executor runs workers (possibly concurrent API calls).
4. Return all `tool_result` blocks to orchestrator.
5. Orchestrator synthesizes final answer.

This reuses standard **`stop_reason: tool_use`** mechanics — [agent-loop.md](./agent-loop.md).

### Context isolation benefit

Workers receive **only** subtask-relevant context:

- Prevents cross-contamination between subtasks
- Reduces token load vs one giant thread
- Limits blast radius of tool errors

Orchestrator keeps **summary-level** state, not full worker traces (unless debugging).

### Parallel vs sequential workers

| Parallel | Sequential |
| --- | --- |
| Independent research on topics A, B, C | Step B needs output of step A |
| Fan-out via parallel `tool_use` | Chain worker calls across loop iterations |
| Lower latency | Stricter dependencies |

Exam: **"Analyze 5 vendors independently"** → parallel workers.

### Orchestrator failure modes

- Over-decomposition → RPM/token blowup
- Under-decomposition → worker context too large
- Skipping synthesis → user gets raw worker dumps

Always include **aggregation step** in orchestrator prompt or final tool.

### vs single monolithic agent

| Monolithic agent | Orchestrator–workers |
| --- | --- |
| One history, many tools | Multiple histories, scoped tools |
| Simpler code | Better scale and isolation |
| Context fills fast | Subtasks stay small |

## Exam Notes

- **"Research 10 companies concurrently"** → orchestrator + parallel workers, not one long chain.
- **"Prevent workers from seeing each other's raw data"** → separate contexts per worker.
- **"Orchestrator receives tool results"** → standard `tool_result` / `tool_use_id` rules apply.
- **"Which pattern for map-reduce over documents?"** → orchestrator assigns chunks to workers, aggregates.
- Distractor: one Opus call with entire corpus when **chunk workers** is correct.

## Production Notes

- Timeout and **partial failure** policy — return worker failures as `is_error` tool_results to orchestrator.
- Limit parallel worker count to respect **RPM/TPM** — [../03-api/rate-limits.md](../03-api/rate-limits.md).
- Use Haiku workers + Sonnet orchestrator for cost control.
- Pass structured handoff format between orchestrator and workers (JSON schema).
- Trace worker IDs in observability for debugging.
- Idempotent worker tasks simplify retries.

## Common Mistakes

- Sharing **full user history** with every worker — defeats isolation.
- No **synthesis** step — user gets fragmented output.
- Serializing work that exam clearly marks **independent** (miss parallel tool call opportunity).
- Orchestrator with **too many tools** — workers should have narrow tool sets.
- Ignoring **`max_iterations`** on nested worker loops.

## See Also

- [Manager Pattern](./manager-pattern.md)
- [Multi-Agent](./multi-agent.md)
- [Workflow Patterns](./workflow-patterns.md)
- [Parallelization](../10-design-patterns/parallelization.md)
- [Agent Loop](./agent-loop.md)
- [Tool Use](./tool-use.md)
