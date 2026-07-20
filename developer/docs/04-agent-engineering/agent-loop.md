# Agent Loop

The **agent loop** is the core implementation pattern for Claude agents: call the Messages API, inspect `stop_reason`, execute tools when needed, append results to history, and repeat until the model signals completion.

**Domain focus:** Agents and Workflows (15%) — the highest-yield agent topic on CCDV F.

## Learning Objectives

After this page you should be able to:

- Implement a correct tool-use loop driven by `stop_reason`.
- Append assistant and user messages in the proper format every iteration.
- Handle parallel `tool_use` blocks in a single turn.
- Terminate on `end_turn` and handle `max_tokens`, `refusal`, and errors.
- Debug broken loops from history mistakes on exam scenarios.

## Key Ideas

### Canonical loop

```python
history = [initial_user_message]

while iteration < MAX_ITERATIONS:
    response = client.messages.create(
        model=MODEL,
        max_tokens=4096,
        system=SYSTEM,
        tools=TOOLS,
        tool_choice={"type": "auto"},
        messages=history,
    )

    # 1. Always append full assistant turn
    history.append({"role": "assistant", "content": response.content})

    # 2. Branch on stop_reason
    if response.stop_reason == "end_turn":
        break  # done — extract text for user

    if response.stop_reason == "tool_use":
        tool_results = []
        for block in response.content:
            if block.type == "tool_use":
                output = run_tool(block.name, block.input)
                tool_results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": output,
                })
        history.append({"role": "user", "content": tool_results})
        continue

    # handle max_tokens, refusal, etc.
    break
```

### `stop_reason` state machine

```
                    ┌─────────────┐
                    │  API call   │
                    └──────┬──────┘
                           │
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
      end_turn         tool_use       max_tokens
           │               │               │
           ▼               ▼               ▼
        STOP          run tools      retry/escalate
                           │
                           ▼
                    append tool_result
                           │
                           └──────► API call again
```

**Exam mantra:** `tool_use` → execute → `tool_result` → continue until `end_turn`.

### History append rules (non-negotiable)

1. **Assistant message** — exact `content` from API, including all `tool_use` blocks.
2. **User message** — only `tool_result` blocks (plus optional text in same message per docs).
3. **Every `tool_use.id`** has exactly one matching `tool_result.tool_use_id`.
4. **Never skip** a turn — the model won't know tools ran.

### Parallel tool calls

One response may include:

```json
"content": [
  { "type": "tool_use", "id": "toolu_A", "name": "get_user", "input": {...} },
  { "type": "tool_use", "id": "toolu_B", "name": "get_orders", "input": {...} }
],
"stop_reason": "tool_use"
```

Execute **both** (can parallelize in your infra), return **both** results in **one** user message.

### `tool_choice` in the loop

| Scenario | Setting |
| --- | --- |
| Normal agent | `auto` |
| Must use a tool this turn | `any` or specific `tool` |
| Force final natural answer | `none` (disallow tools for that call) |

Switching `tool_choice` per iteration is valid — e.g., force extraction tool on first pass, `auto` thereafter.

### Streaming variant

Same logic — accumulate streamed blocks until `message_stop`, **then** check `stop_reason`. Don't execute tools on partial streams — [../03-api/streaming.md](../03-api/streaming.md).

### Termination guards

Besides `end_turn`:

| Condition | Action |
| --- | --- |
| `max_iterations` exceeded | Stop; inform user; log state |
| `max_tokens` stop | Increase limit or ask model to continue with summary |
| `refusal` | Stop; safety UX |
| Repeated identical tool calls | Detect loop; break with escalation |

### Tool errors inside the loop

Failed tool → still append `tool_result` with **`is_error: true`** — model may retry or explain. **Do not** break history pairing.

## Exam Notes

- **"Model keeps asking to run tool but app already ran it"** → missing `tool_result` in history.
- **"Two tools requested, only one result sent"** → incomplete parallel handling.
- **"Infinite loop"** → add max iterations; check for missing `end_turn` handling.
- **"When to stop?"** → `stop_reason: end_turn`, not "when text looks complete."
- **Wrong fix:** new conversation each tool round instead of **append full history**.

## Production Notes

- **Structured logging** per iteration: `i`, `stop_reason`, tool names, ms, tokens.
- Run independent tools **concurrently** with timeout per tool.
- Persist `history` after each iteration for crash recovery / resume.
- Consider **human approval** hook between `tool_use` detection and execution for destructive tools.
- Use prompt caching on static system + tools prefix — history suffix changes each iteration.
- Separate read-only and write tools in policy layer — least privilege.

## Common Mistakes

- Checking text for JSON instead of **`stop_reason`**.
- Executing tools before assistant message is appended.
- Wrong message **role** for tool results (must be `user` content blocks).
- Inventing new `tool_use_id` values on retry.
- One API call per tool when model already requested **parallel** calls in one turn.
- Stopping on first `text` block while **`tool_use`** blocks also present — process all blocks.

## See Also

- [Agent Basics](./agent-basics.md)
- [Tool Use](./tool-use.md)
- [Tool Results](./tool-results.md)
- [Messages API](../03-api/messages-api.md)
- [Error Handling](../03-api/error-handling.md)
- [Streaming](../03-api/streaming.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
