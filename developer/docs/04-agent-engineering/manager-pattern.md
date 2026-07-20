# Manager Pattern

The **manager pattern** (supervisor) places a central **manager** model in charge of specialized **subagents**, routing tasks, reviewing output, and deciding when the job is complete. It emphasizes **control and quality gates** over blind parallel fan-out.

**Domain focus:** Agents and Workflows (15%) — pattern selection and implementation.

## Learning Objectives

After this page you should be able to:

- Distinguish manager pattern from orchestrator–workers and fixed workflows.
- Define manager responsibilities: routing, review, retry, termination.
- Implement manager loops using tool use and isolated subagent contexts.
- Identify exam scenarios favoring supervision vs simple delegation.
- Apply human-in-the-loop at manager approval points.

## Key Ideas

### Architecture

```
User → Manager (supervisor)
         ├─► Subagent: Researcher
         ├─► Subagent: Coder
         └─► Subagent: Reviewer
       Manager → User (final)
```

**Manager** maintains global objective and conversation with user. **Subagents** receive scoped assignments and return structured results to the manager — not directly to the user (usually).

### Manager vs orchestrator–workers

| Aspect | Manager pattern | Orchestrator–workers |
| --- | --- | --- |
| Focus | Routing, QA, retries | Parallel decomposition |
| Subagent visibility | Manager reviews before proceeding | Orchestrator may merge quickly |
| Iteration | Manager may reject worker output | Workers often one-shot per subtask |
| Best for | Quality-critical, mixed skills | Independent parallel subtasks |

See [orchestrator-workers.md](./orchestrator-workers.md) for parallel map-reduce style work.

### Implementation via tools

Manager's tool set often includes:

| Tool | Purpose |
| --- | --- |
| `delegate_to_researcher` | Spawn researcher subagent with task JSON |
| `delegate_to_coder` | Implementation subagent |
| `request_human_approval` | Pause for human gate |
| `finalize_response` | Mark completion |

Each delegation:

1. Manager `tool_use` → `stop_reason: tool_use`
2. Your runtime runs subagent (possibly full agent loop)
3. `tool_result` back to manager with structured output
4. Manager decides next step until `end_turn`

### Review and retry loop

Manager examines worker `tool_result`:

- If inadequate → delegate again with refined instructions (new subagent call)
- If `is_error` → manager handles recovery or escalates
- If acceptable → next subagent or final answer

**Schema ≠ semantic truth** — manager should validate worker JSON before trusting.

### Context isolation

Subagents get:

- Task description from manager
- Minimal attachments (not full user relationship history)

Manager gets:

- User goal
- Summaries of worker outputs
- Optional full worker trace for debugging (production: usually summarized)

### Termination

Manager stops when:

- `stop_reason: end_turn` on manager's API call
- Internal policy: all checklist items satisfied
- Human approval received for sensitive actions

Subagent `end_turn` alone does **not** finish user request — manager must synthesize.

### Claude Code / SDK mapping

- **Subagents** configured with separate prompts and tool allowlists
- **Manager** = main session with delegation tools or `@agent` invocations
- Hooks can enforce approval before manager executes write tools

## Exam Notes

- **"Review code before merge"** → manager + reviewer subagent, not single pass.
- **"Route customer intent to specialist"** → manager routing, not one agent with 50 tools.
- **"Worker failed — who handles?"** → manager receives `is_error` tool_result and replans.
- **"User talks to multiple agents directly"** → usually wrong; manager consolidates UX.
- vs **orchestrator**: "parallel independent research" → orchestrator; "supervised pipeline with QA" → manager.

## Production Notes

- Cap **delegation depth** (manager → worker → sub-worker) to avoid cost explosions.
- Structured handoff schema between manager and workers (task_id, acceptance_criteria, output_format).
- Log manager decisions (which subagent, why) for audit.
- Rate-limit delegations per user request.
- Use cheaper models for workers, stronger model for manager when reasoning about failures.
- Human approval on manager tool calls that trigger external side effects.

## Common Mistakes

- Exposing **all tools** to every subagent — breaks least privilege.
- Subagents returning directly to **user** — bypasses manager QA.
- No retry path when worker output fails validation.
- Manager context bloated with **full worker transcripts** every turn — summarize.
- Confusing manager pattern with **graph** explicit state machine — manager is often LLM-driven routing.

## See Also

- [Orchestrator–Workers](./orchestrator-workers.md)
- [Graph Pattern](./graph-pattern.md)
- [Multi-Agent](./multi-agent.md)
- [Supervisor](../10-design-patterns/supervisor.md)
- [Agent Loop](./agent-loop.md)
- [Agent Best Practices](./agent-best-practices.md)
