# Multi-Agent

**Multi-agent** systems use multiple Claude-powered agents (or mixed human/agent roles) collaborating on a task — typically with **separate prompts, tools, and contexts** coordinated by an orchestrator, manager, or graph.

**Domain focus:** Agents and Workflows (15%) — decomposition, isolation, and coordination.

## Learning Objectives

After this page you should be able to:

- Explain why multi-agent beats one monolithic agent for complex workloads.
- Describe coordination patterns: orchestrator, manager, peer handoff.
- Implement multi-agent flows using tool delegation and isolated histories.
- Manage cost, rate limits, and failure propagation across agents.
- Answer exam questions on context isolation and subagents.

## Key Ideas

### Why multiple agents?

| Problem in monolithic agent | Multi-agent fix |
| --- | --- |
| Context window fills with irrelevant tool traces | Scoped context per agent |
| Tool sprawl confuses model | Narrow tool sets per role |
| One failure corrupts whole thread | Isolate worker failures |
| Hard to parallelize | Independent agents run concurrently |

### Agent roles (typical team)

| Agent | Role |
| --- | --- |
| **Planner** | Breaks down task, assigns work |
| **Researcher** | Retrieval, web, docs |
| **Coder** | Writes/edits code via tools |
| **Reviewer** | Critiques output, security check |
| **User proxy** | Clarifying questions (optional) |

Roles map to **system prompts + tools**, not different API products.

### Coordination mechanisms

1. **Orchestrator delegation** — [orchestrator-workers.md](./orchestrator-workers.md)
2. **Manager supervision** — [manager-pattern.md](./manager-pattern.md)
3. **Graph transitions** — [graph-pattern.md](./graph-pattern.md)
4. **Peer handoff** — Agent A completes, passes structured state to Agent B via your runtime (not direct API-to-API)

All reduce to: **your orchestration code** + multiple Messages API sessions.

### Subagent context isolation

Each subagent call includes:

- Role-specific **system** prompt
- **Subset** of task state (not necessarily full user chat)
- **Allowed tools** only for that role

Subagent runs own loop until **`stop_reason: end_turn`**, returns compact result to parent.

Parent receives result via **`tool_result`** if parent invoked subagent as tool — preserves [tool-results.md](./tool-results.md) pairing.

### Parallel multi-agent

Independent subagents → parallel execution:

- Parallel **API requests** from your worker pool
- Or parallel **`tool_use`** in orchestrator turn

Watch **RPM/TPM** — N agents × M loop iterations multiplies calls quickly.

### Model selection per agent

| Agent | Model choice |
| --- | --- |
| Router / classifier | Haiku — fast, cheap |
| Deep reasoning planner | Sonnet / Opus |
| Bulk summarizer workers | Haiku / Sonnet |
| Code implementation | Sonnet (typical default) |

Optimize **quality per dollar**, not one model everywhere.

### Failure across agents

Worker agent fails:

- Return **`is_error: true`** to parent via tool_result
- Parent replans, retries another worker, or asks user

Don't silently drop failed agent — parent needs structured failure.

### Multi-agent vs multi-turn single agent

| Multi-turn single agent | Multi-agent |
| --- | --- |
| One `messages` history | Multiple histories |
| One system prompt | Many specialized prompts |
| Simpler | Scales better for teams of skills |

Exam trick: "subagents" almost always implies **isolation**, not just multiple prompts in one thread.

### Claude Code / SDK subagents

Products may expose subagents as first-class (spawn, resume). Conceptually identical: parent delegates, child runs with own config, returns result.

Know **resume/continue** applies per session your app stores — not magic cross-agent memory.

## Exam Notes

- **"Specialist agents shouldn't see HR data"** → isolation + tool permissions.
- **"Parallel research on 8 topics"** → multi-agent parallel, not sequential single agent.
- **"Parent agent confused by worker tool logs"** → pass summaries up, not raw traces.
- **"Same tool_use rules?"** → yes — delegation tools still use `tool_result` / `tool_use_id`.
- **"Simple FAQ bot"** → single agent or router — multi-agent overkill (simplest fix wins).

## Production Notes

- Standardize **handoff JSON schema** between agents.
- Global **max delegation budget** (tokens, dollars, wall clock).
- Trace correlation IDs across agent spans in observability.
- Avoid circular delegation (A → B → A) — cap depth.
- Security: subagents inherit **least privilege** credentials, not admin keys.
- Eval multi-agent systems end-to-end — unit tests per agent aren't sufficient.

## Common Mistakes

- All agents share **one history** — negates multi-agent benefits.
- **Too many agents** for linear tasks — coordination overhead dominates.
- Returning worker **full chat logs** to parent — blows context.
- Forgetting parent still needs **`stop_reason: end_turn`** before user sees final answer.
- No plan for **partial parallel failure** (3/5 workers succeed).

## See Also

- [Orchestrator–Workers](./orchestrator-workers.md)
- [Manager Pattern](./manager-pattern.md)
- [Graph Pattern](./graph-pattern.md)
- [Agent Basics](./agent-basics.md)
- [Rate Limits](../03-api/rate-limits.md)
- [Sessions](../05-sdk/sessions.md)
