# Decomposition

**Decomposition** breaks a complex task into smaller subtasks—each with focused prompts, tools, and context—so agents stay accurate, within token budgets, and easier to test. Core pattern for **Agents and Workflows (14.7%)**.

## Learning Objectives

After this page, you should be able to:

- Decide when to decompose vs keep a single agent loop
- Design subtasks with clear inputs, outputs, and contracts
- Use subagents and workers for context isolation
- Merge subtask results into coordinator context efficiently
- Avoid decomposition overhead on simple workflows

## Key Ideas

### Why decompose

Single monolithic agents struggle when:

- Task spans unrelated domains (legal + code + design)
- Context fills with research noise before execution
- Different steps need different models or tools
- Parallel independent work speeds completion

Decomposition trades **coordination cost** for **focus**.

### Decomposition patterns

**1. Sequential pipeline**

```text
Research → Plan → Execute → Review
```

Each stage is a separate call or agent with structured handoff.

**2. Map-reduce**

```text
Coordinator splits items → workers process in parallel → aggregator synthesizes
```

Good for multi-file analysis, batch Q&A.

**3. Subagent isolation**

Coordinator spawns worker with **fresh context** + narrow tools; worker returns summary only.

**4. Tool-first decomposition**

Complex operations exposed as tools (`analyze_repo`, `write_tests`) rather than free-form subagents—simpler when steps are deterministic.

### Handoff contracts

Each subtask output should be structured:

```json
{
  "status": "complete",
  "artifacts": ["plan.md"],
  "decisions": ["use API v2"],
  "open_questions": []
}
```

Free-text novels between agents reintroduce noise—compress.

### Context isolation (exam favorite)

Subagents **do not automatically inherit** full coordinator history unless you inject it. Pass:

- Relevant excerpt or summary
- Explicit constraints
- Tool allowlist for worker

Prevents leakage of irrelevant secrets and saves tokens.

See [Multi-Agent](../04-agent-engineering/multi-agent.md).

### When not to decompose

- Simple single-tool lookup
- Short codegen with one file
- Adding multi-agent latency hurts UX without quality gain

Exam trap: fancy decomposition when **one loop + one tool** suffices.

## Exam Notes

- Large research then narrow execution → **subagent/worker returns summary**
- Coordinator loses track → **structured handoff**, not full log forward
- Different tool sets per phase → **decompose by tool manifest**
- Parallel doc analysis → **map-reduce / parallelization**
- Subagent "should know" prior chat → you must **pass context explicitly**

## Production Notes

### Decomposition checklist

- [ ] Subtask I/O schema defined
- [ ] Worker token/time limits
- [ ] Coordinator validates worker output before next phase
- [ ] Eval per subtask + end-to-end golden path
- [ ] Trace spans per subagent for debugging

## Common Mistakes

- **Forwarding entire subagent transcript** — defeats isolation
- **Too many micro-agents** — coordination failures
- **No validation between stages** — error propagates
- **Same tools on all agents** — no isolation benefit
- **Decomposing without parallel or context win** — pure overhead

## See Also

- [Orchestration](./orchestration.md)
- [Supervisor](./supervisor.md)
- [Parallelization](./parallelization.md)
- [Pruning](../09-performance/pruning.md)
- [Orchestrator-Workers](../04-agent-engineering/orchestrator-workers.md)
