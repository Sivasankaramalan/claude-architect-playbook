# Agent Basics

In Claude developer workflows, an **agent** is an application pattern: a model in a **loop** that can **use tools**, observe results, and continue reasoning until the task completes. It is not a separate product — it is how you orchestrate the Messages API for autonomous multi-step work.

**Domain focus:** Agents and Workflows (15%) + Applications and Integration (33%).

## Learning Objectives

After this page you should be able to:

- Define an agent as LLM + tools + control loop + conversation state (in your app).
- Contrast agents with single-shot prompting and fixed workflows.
- Identify the components: system prompt, tools, history, executor, termination condition.
- Explain why `stop_reason` is the agent's "program counter."
- Recognize when an exam scenario needs an agent vs a simple API call.

## Key Ideas

### Minimal agent definition

```
Agent = Model + Tools + Loop + State (messages history)
```

| Component | Role |
| --- | --- |
| **Model** | Reasoning and planning (Claude via Messages API) |
| **Tools** | Functions the model can invoke (APIs, DB, code, search) |
| **Loop** | Repeated call → act → observe until done |
| **State** | Full `messages` array your application persists |

Claude does not run the loop for you in basic API usage — **your code** (or SDK/agent framework) does.

### Single-shot vs agent

| | Single-shot | Agent |
| --- | --- | --- |
| API calls per user request | 1 | 1 to N |
| External actions | None or manual | Model-driven via tools |
| Termination | Always after one response | When `stop_reason: end_turn` |
| Best for | Q&A, rewrite, classify | Research, coding tasks, multi-step ops |

### The agent control loop (preview)

```
while True:
    response = messages.create(..., messages=history, tools=tools)
    append assistant response to history

    if response.stop_reason == "end_turn":
        return final answer to user

    if response.stop_reason == "tool_use":
        for each tool_use in response.content:
            result = execute_tool(tool_use)
            append user message with tool_result(s) to history
        continue

    handle max_tokens / refusal / other stop reasons
```

Deep dive: [agent-loop.md](./agent-loop.md).

### What makes Claude "agentic"

- **Tool use** — model emits structured `tool_use` blocks
- **Replanning** — after `tool_result`, model can call different tools
- **Parallel tool calls** — multiple `tool_use` blocks in one assistant turn
- **Long horizon** — many iterations until `end_turn`

### Agents vs workflows

- **Agent:** model decides **which** tool next (dynamic).
- **Workflow:** developer defines **fixed** steps (deterministic pipeline).
- Production systems often **hybrid** — workflow skeleton with agentic steps inside. See [workflow-patterns.md](./workflow-patterns.md).

### State and sessions

- **Session** = your stored `messages` + metadata (user id, file refs).
- API is **stateless** — resume/continue means **resending history**, not a server session ID (unless using platform-specific session features in SDK products).
- **Memory** patterns (summarize old turns, vector store) sit **above** the raw API.

### Human in the loop

Agents that mutate production data often require:

- Approval before destructive tools
- Display of planned tool calls
- Ability to cancel mid-loop

Exam scenarios with **"production-safe"** often imply approval gates — [agent-best-practices.md](./agent-best-practices.md).

## Exam Notes

- **"Multi-step task with API lookups"** → agent with tools, not one giant prompt.
- **"How does the app know to keep going?"** → `stop_reason == "tool_use"`.
- **"Where is conversation stored?"** → developer appends to `messages`, not Anthropic session by default.
- **"Guarantee tool is called"** → `tool_choice`, not "better prompt" alone.
- Agent question distractors: **MCP replaces the loop** (MCP provides tools; you still run the loop).

## Production Notes

- Cap **max iterations** per user request (e.g., 10–25) to control cost and RPM — [../03-api/rate-limits.md](../03-api/rate-limits.md).
- Separate **orchestrator** process from tool executors for isolation and scaling.
- Log every turn: `stop_reason`, tools called, latency, token usage.
- Use smaller models for cheap routing subtasks in multi-agent setups — [multi-agent.md](./multi-agent.md).
- Tool definitions belong in stable, cacheable system/tool blocks when possible.

## Common Mistakes

- Calling an agent anything that **does one API call** without a loop.
- Forgetting to **persist and resend history** between turns.
- No termination condition → infinite loop on repeated `tool_use`.
- Trusting model text that says "I ran the query" without **`tool_use`** actually occurring.
- Building agents without **`is_error`** handling on tool failures — [tool-results.md](./tool-results.md).

## See Also

- [Agent Loop](./agent-loop.md)
- [Tool Use](./tool-use.md)
- [Tool Results](./tool-results.md)
- [Workflow Patterns](./workflow-patterns.md)
- [Messages API](../03-api/messages-api.md)
- [Agent Best Practices](./agent-best-practices.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
