# Orchestration

**Orchestration** is the control logic that runs agent loops, routes tasks, invokes tools, handles errors, and enforces limits—your code, not the model. **Agents and Workflows (14.7%)** tests whether you know how **`stop_reason`**, retries, and workflow state drive reliable systems.

## Learning Objectives

After this page, you should be able to:

- Implement the standard agent loop with correct `stop_reason` handling
- Combine LLM decisions with deterministic workflow steps
- Manage errors, retries, and human approval in orchestration code
- Integrate SDK sessions, hooks, and MCP into one control plane
- Distinguish orchestration from "let the model figure out the whole workflow"

## Key Ideas

### The canonical agent loop

```text
messages = initial_history
loop:
  response = client.messages.create(..., messages=messages)
  append assistant response to messages
  if response.stop_reason == "tool_use":
      results = execute_tools(response.content)
      append tool_result messages to messages
      continue loop
  elif response.stop_reason == "end_turn":
      return extract_final_output(response)
  else:
      handle max_tokens / other stop reasons
```

Orchestrator owns **when to stop**, **max iterations**, and **tool execution**.

See [Agent Loop](../04-agent-engineering/agent-loop.md).

### Hybrid orchestration

Not every step needs the model:

| Step type | Handler |
| --- | --- |
| Fetch user record | Deterministic API call |
| Decide refund eligibility | Model + policy tool |
| Charge card | Code after approval gate |
| Send email | Tool with idempotency key |

Use **state machines** for known workflows; use **agents** for ambiguous user language.

### Stop reasons as control signals

| `stop_reason` | Orchestrator action |
| --- | --- |
| `tool_use` | Execute tools, continue |
| `end_turn` | Finish or validate output |
| `max_tokens` | Retry with higher limit or ask user to narrow |
| `pause_turn` | Rare; handle per API docs |

Do not invent parallel control channels when `stop_reason` already defines flow.

### Error orchestration

- Tool failure → structured error in `tool_result`; model may recover
- Repeated tool failure → escalate to human or abort
- API 429 → backoff at client; don't spin agent turns
- Validation failure → retry with schema errors (capped)

### Human-in-the-loop orchestration

Pause workflow state machine at `AWAITING_APPROVAL`; resume after UI event—not by hoping model waits.

### Claude Code / SDK orchestration

- **Sessions**: resume vs fresh
- **Hooks**: pre/post tool
- **Subagents**: delegated runs with return channel
- **Headless**: CI with permission flags

Config hierarchy affects orchestration behavior—see [Hooks](../05-sdk/hooks.md).

## Exam Notes

- Agent keeps going after final answer → check **`end_turn` handling**
- Tool never runs → **`tool_use` loop** broken or missing executor
- Production workflow with approvals → **orchestrator state + hooks**, not prompt pleading
- Deterministic step in flow → **code**, not extra LLM call
- Rate limits → **client retry**, not more agent turns

## Production Notes

### Orchestration checklist

- [ ] Max turns enforced
- [ ] Idempotent tool execution where required
- [ ] Central message history builder (single module)
- [ ] Timeouts per tool and per session
- [ ] Structured logging at each loop iteration
- [ ] Feature flags for workflow versions

## Common Mistakes

- **Model orchestrates via prose** ("I'll now call tool X") without `tool_use` block
- **Missing tool_result append**
- **No cap on loop iterations** — cost incidents
- **Mixing multiple workflows in one prompt** — unclear stop conditions
- **Ignoring non-`end_turn` stop reasons**

## See Also

- [Agent Loop](../04-agent-engineering/agent-loop.md)
- [Tool Results](../04-agent-engineering/tool-results.md)
- [Supervisor](./supervisor.md)
- [Workflow Patterns](../04-agent-engineering/workflow-patterns.md)
- [Graph Pattern](../04-agent-engineering/graph-pattern.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
