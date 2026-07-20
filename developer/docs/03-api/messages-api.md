# Messages API

The **Messages API** is the core integration surface for Claude. Every CCDV F application question ultimately maps to how you shape a request, interpret the response, and decide what to do next — especially around `stop_reason`, tool use, and conversation history.

**Domain focus:** Applications and Integration (33%) — request lifecycle, parameters, and control flow.

## Learning Objectives

After this page you should be able to:

- Construct a valid Messages API request (`model`, `max_tokens`, `messages`, optional `system`, `tools`, `tool_choice`, `stream`).
- Explain the response shape: `content` blocks, `stop_reason`, and `usage`.
- Drive an agentic loop using `stop_reason` (`tool_use` → execute → `tool_result` → continue until `end_turn`).
- Choose between synchronous calls, streaming, and Message Batches for a given workload.
- Identify which API parameter solves a scenario question (history, tools, structured output, caching).

## Key Ideas

### Endpoint and mental model

- **Endpoint:** `POST /v1/messages`
- **Stateful conversations live in your app**, not on Anthropic's servers. You send the full message history on every call (subject to context limits).
- **Roles:** `user` and `assistant` alternate in `messages`. The API does not accept a leading `assistant` message.
- **System prompt:** Pass via top-level `system` (string or content blocks), not as a `messages` role.

### Request essentials

| Parameter | Purpose |
| --- | --- |
| `model` | Which Claude model (Opus, Sonnet, Haiku, dated snapshots) |
| `max_tokens` | Hard cap on model output tokens — always required |
| `messages` | Conversation history as role + content blocks |
| `system` | Instructions, persona, policies — cached separately when using prompt caching |
| `tools` | JSON Schema tool definitions the model may call |
| `tool_choice` | How the model selects tools: `auto`, `any`, `tool`, `none` |
| `stream` | `true` for Server-Sent Events token/block streaming |
| `temperature` / `top_p` | Sampling; lower for deterministic tasks |
| `stop_sequences` | Custom strings that halt generation (`stop_reason: stop_sequence`) |

### Response essentials

```json
{
  "id": "msg_...",
  "type": "message",
  "role": "assistant",
  "content": [
    { "type": "text", "text": "..." },
    { "type": "tool_use", "id": "toolu_...", "name": "get_weather", "input": { "city": "Paris" } }
  ],
  "stop_reason": "tool_use",
  "stop_sequence": null,
  "usage": { "input_tokens": 120, "output_tokens": 45 }
}
```

**Content blocks** the model can return:

- `text` — natural language
- `tool_use` — model wants your code to run a tool (`id`, `name`, `input`)
- (When using extended thinking models) thinking blocks may appear — know they are model-internal reasoning, not user-facing by default

### `stop_reason` — the control-flow switch

This is the **#1 exam concept** for developer integration:

| `stop_reason` | Meaning | Your code |
| --- | --- | --- |
| `end_turn` | Model finished its turn | Return response to user; stop the loop |
| `tool_use` | Model emitted one or more `tool_use` blocks | Execute tools, append `tool_result` blocks, call API again |
| `max_tokens` | Hit `max_tokens` before finishing | Increase limit, truncate/retry, or stream partial result |
| `stop_sequence` | Hit a custom stop string | Handle as intentional halt |
| `refusal` | Safety refusal | Do not retry blindly; adjust prompt or escalate |

**Agent loop in one line:** call API → if `stop_reason == "tool_use"` → run tools → append results → call API again → repeat until `end_turn`.

### Conversation history rules

1. **Append the full assistant turn** — including every `tool_use` block exactly as returned.
2. **Append a user message** containing `tool_result` blocks — one per `tool_use`, each with matching `tool_use_id`.
3. Never omit intermediate turns; the model has no memory outside what you send.
4. **Parallel tool calls:** one assistant message may contain multiple `tool_use` blocks; return all matching `tool_result` blocks in the next user message (order can match tool_use order).

### `tool_choice`

| Value | Behavior |
| --- | --- |
| `auto` (default) | Model decides whether to call tools or respond in text |
| `any` | Must use at least one tool |
| `tool` + `name` | Must call that specific tool |
| `none` | Tools visible but model cannot call them — text only |

Use `tool` / `any` when the exam says **guarantee**, **must invoke**, or **force structured extraction**.

### Structured output

- Provide tools with strict JSON Schema, or use dedicated structured-output patterns documented for your SDK.
- **Schema validates structure, not semantic truth** — a field can be the wrong value while still validating. Add app-level validation and evals.

### Sync vs streaming vs batches

| Mode | When |
| --- | --- |
| **Synchronous** | Short responses, server-side aggregation, simplest code path |
| **Streaming** (`stream: true`) | Chat UX, long generations, early cancellation — see [streaming.md](./streaming.md) |
| **Message Batches** | Large offline jobs, 50% cost discount, async completion within ~24h — not for interactive agents |

### Token usage

- `usage.input_tokens` / `usage.output_tokens` on every response — use for cost estimation and context budgeting.
- Count tokens before sending large payloads; see [../09-performance/context-window.md](../09-performance/context-window.md).

## Exam Notes

- **"What happens next after the API returns?"** → Check `stop_reason` first, not the text content.
- **"Model keeps answering instead of calling the tool"** → `tool_choice: { type: "any" }` or `{ type: "tool", name: "..." }`, plus clear tool descriptions.
- **"Tool ran but model is confused"** → History incomplete: missing assistant `tool_use` or mismatched `tool_use_id` on `tool_result`.
- **"Guarantee JSON shape"** → Tool schema + validation in your code; don't rely on prompt alone.
- **Multiple tools in one turn** → Normal; execute all, return all `tool_result` blocks together.
- Distractors often suggest **starting a new conversation** when the fix is **append history**.

## Production Notes

- Set **`max_tokens`** high enough for tool arguments + final answer; too low causes `max_tokens` stop mid-tool-call.
- Use **idempotency keys** (SDK support) for safe retries on network failures.
- Log `id`, `stop_reason`, token usage, and tool names for every turn — essential for debugging agent loops.
- **Pin model versions** (dated snapshots) in production; avoid silent behavior changes on alias updates.
- Separate **system** instructions from user content; use prompt caching on stable system + tool definitions.
- Validate tool **inputs** before execution (schema ≠ safety); treat tool args as untrusted.

## Common Mistakes

- Sending only the latest user message instead of **full history**.
- Putting tool results in a new `assistant` message instead of **`user` role with `tool_result` blocks**.
- Reusing or inventing `tool_use_id` values instead of copying from the API response.
- Ignoring **`stop_reason: tool_use`** and showing partial text to the user.
- Assuming JSON Schema guarantees **correct business data**.
- Using Message Batches for **real-time chat** (wrong — use sync or streaming).

## See Also

- [Streaming](./streaming.md)
- [Tool Use](../04-agent-engineering/tool-use.md)
- [Tool Results](../04-agent-engineering/tool-results.md)
- [Agent Loop](../04-agent-engineering/agent-loop.md)
- [Error Handling](./error-handling.md)
- [Retries](./retries.md)
- [Rate Limits](./rate-limits.md)
- [Vision](./vision.md)
- [Files](./files.md)
