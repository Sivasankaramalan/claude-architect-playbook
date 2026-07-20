# Supervisor

The **supervisor** pattern uses a coordinator agent that plans, delegates work to worker agents or tools, and synthesizes results—without workers seeing each other's full context unless needed. Central to **Agents and Workflows (14.7%)** and overlaps architect cert multi-agent topics with developer implementation focus.

## Learning Objectives

After this page, you should be able to:

- Describe supervisor vs single-loop vs peer multi-agent models
- Delegate with scoped context and tool permissions per worker
- Merge worker outputs at the supervisor without context blow-up
- Implement termination and error handling at supervisor level
- Recognize when supervisor complexity is unjustified

## Key Ideas

### Roles

| Role | Responsibility |
| --- | --- |
| **Supervisor** | Plan, delegate, monitor progress, final answer |
| **Worker** | Execute focused subtask with narrow tools |
| **Tools** | Deterministic actions workers invoke |

Supervisor is not "just another prompt"—it owns **delegation protocol**.

### Control flow

```text
User → Supervisor
          │
          ├─► Worker A (research) → summary A
          ├─► Worker B (code)     → patch B
          └─► Worker C (review)   → findings C
          │
          ▼
     Supervisor synthesizes → User
```

Workers return **structured summaries**, not full transcripts.

See [Orchestrator-Workers](../04-agent-engineering/orchestrator-workers.md).

### Context isolation

Workers start with:

- Task description from supervisor
- Minimal relevant files/IDs
- Allowed tools only

They **do not** automatically inherit full user chat history—supervisor must pass needed facts explicitly (exam trap).

### Supervisor tools

Supervisor may call:

- `delegate_to_researcher(task)`
- `delegate_to_coder(task)`
- Or spawn SDK subagents

Workers should not expose supervisor-only tools (deploy prod, billing).

### Error handling

If worker fails:

- Supervisor retries with narrower task
- Escalates to human
- Reports partial result with limitation

Supervisor tracks **task state**—pending, running, done, failed.

### vs Manager pattern

**Manager** often assigns sequential subtasks with approval gates. **Supervisor** emphasizes oversight and synthesis of parallel workers. Overlap exists—choose based on parallelism needs.

See [Manager Pattern](../04-agent-engineering/manager-pattern.md).

## Exam Notes

- Multi-domain task → **supervisor + specialized workers**
- Worker lacks user constraint → **supervisor didn't pass context**
- Context too large → **workers isolated**, not one thread
- Simple lookup → **no supervisor** — single tool call
- Write to production → **supervisor doesn't give worker prod credentials**

## Production Notes

### Supervisor checklist

- [ ] Worker I/O schema
- [ ] Per-worker timeouts and token caps
- [ ] Supervisor max delegation depth (no infinite recurse)
- [ ] Trace: supervisor span → worker child spans
- [ ] Eval for delegation accuracy (right worker chosen)

## Common Mistakes

- **Workers share full history** — privacy and token blow-up
- **Supervisor does all work itself** — workers decorative
- **No synthesis step** — user gets dump of worker logs
- **Circular delegation** — worker calls supervisor pattern recursively without cap
- **Supervisor for 2-step linear flow** — use pipeline instead

## See Also

- [Decomposition](./decomposition.md)
- [Parallelization](./parallelization.md)
- [Orchestration](./orchestration.md)
- [Multi-Agent](../04-agent-engineering/multi-agent.md)
- [Pruning](../09-performance/pruning.md)
