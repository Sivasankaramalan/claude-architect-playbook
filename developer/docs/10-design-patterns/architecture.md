# Architecture

**Architecture** for Claude applications describes how user requests flow through APIs, agents, tools, MCP servers, memory, and guardrails—not just which model you call. Developer exam questions in **Agents and Workflows (14.7%)** and **Applications and Integration (33.1%)** often ask you to pick a **structure** that meets reliability, security, and cost goals.

## Learning Objectives

After this page, you should be able to:

- Compare monolithic agent, router + workers, and supervisor topologies
- Place the Messages API, SDK, MCP, and Claude Code in a reference architecture
- Identify boundaries for state, secrets, and validation
- Choose the simplest architecture that satisfies requirements (exam principle)
- Map design choices to observability and deployment practices

## Key Ideas

### Reference layers

```text
┌─────────────────────────────────────────┐
│  Client (UI, CLI, webhook)              │
└─────────────────┬───────────────────────┘
                  ▼
┌─────────────────────────────────────────┐
│  Orchestration (agent loop, routing)    │
└─────────────────┬───────────────────────┘
                  ▼
┌─────────────────────────────────────────┐
│  Claude (Messages API / Agent SDK)      │
└─────────────────┬───────────────────────┘
                  ▼
┌─────────────────────────────────────────┐
│  Tools & MCP (data, actions, integrations)│
└─────────────────┬───────────────────────┘
                  ▼
┌─────────────────────────────────────────┐
│  Data & policy (DB, secrets, guardrails) │
└─────────────────────────────────────────┘
```

Your code owns orchestration and tool execution; Claude owns language reasoning within constraints you define.

### Common architectural shapes

| Shape | Description | Best for |
| --- | --- | --- |
| **Single agent loop** | One model + tools until `end_turn` | Most CRUD assistants, coding agents |
| **Router + specialists** | Classify → delegate model/workflow | Mixed intents, cost optimization |
| **Supervisor + workers** | Coordinator assigns subtasks | Parallel research, multi-domain tasks |
| **Pipeline** | Fixed stages without LLM routing | Predictable ETL-style flows |
| **Human-on-rail** | Model proposes; code/human commits | High-risk domains |

Start with **single loop**; add patterns only when metrics prove need.

### State and session architecture

- **Conversation state** — messages array, session id, resume tokens
- **Durable state** — database, object store (facts, user prefs)
- **Ephemeral compute** — tool results trimmed from history after persist

Never rely on model weights for memory—persist explicitly.

See [Sessions](../05-sdk/sessions.md) and [Memory](../05-sdk/memory.md).

### Security architecture

- Secrets never in prompts; tools use server-side credentials
- MCP servers on private networks with auth
- Hooks/guardrails at orchestration boundary
- Least-privilege tool sets per role

See [Safety](../08-production/safety.md).

### Cross-cutting concerns

| Concern | Where it lives |
| --- | --- |
| Retries/rate limits | API client layer |
| Tracing | Orchestrator + tool wrappers |
| Evals | CI + staging gates |
| Config | Versioned repo, not laptops only |

## Exam Notes

- "Simplest reliable solution" → often **single tool-using agent**, not multi-agent theater
- Mixed easy/hard tasks → **router + model selection**, not Opus-only
- High-risk writes → **human approval layer** in architecture diagram
- Team coding agent → **Claude Code / SDK + MCP tools + hooks**, not raw chat-only
- Context isolation for subtasks → **subagents/workers**, not one ever-growing thread

## Production Notes

### Architecture review checklist

- [ ] Can you draw data flow including tool credentials?
- [ ] Single owner for conversation history assembly
- [ ] Guardrails before and after model calls where needed
- [ ] Max turns and timeouts at orchestration layer
- [ ] Deploy ties prompt, code, and tool versions
- [ ] Failure modes: degrade gracefully, not silent wrong answers

## Common Mistakes

- **Multi-agent by default** — coordination overhead and cost
- **Business logic only in prompts** — not testable or enforceable
- **Tools as thin wrappers over admin credentials** — blast radius
- **No session boundary** — poisoned context persists forever
- **MCP as monolith** — one server with 40 tools

## See Also

- [Decomposition](./decomposition.md)
- [Orchestration](./orchestration.md)
- [Supervisor](./supervisor.md)
- [Production Patterns](./production-patterns.md)
- [Workflow Patterns](../04-agent-engineering/workflow-patterns.md)
- [MCP Architecture](../06-mcp/architecture.md)
