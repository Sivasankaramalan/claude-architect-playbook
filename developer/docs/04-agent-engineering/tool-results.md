# Tool Results

After Claude emits `tool_use` blocks (`stop_reason: tool_use`), your application executes the tools and returns **`tool_result`** content blocks in the next **user** message. Correct pairing and error formatting keep the agent loop stable.

**Domain focus:** Agents and Workflows (15%) + Applications and Integration (33%).

## Learning Objectives

After this page you should be able to:

- Format `tool_result` blocks with matching `tool_use_id`.
- Use `is_error: true` for failed tool executions.
- Return results for parallel tool calls in one user message.
- Choose string vs structured content in `tool_result`.
- Diagnose exam scenarios involving broken history or silent failures.

## Key Ideas

### The pairing contract

Every assistant `tool_use`:

```json
{ "type": "tool_use", "id": "toolu_01ABC", "name": "search", "input": { "q": "rates" } }
```

Requires a user `tool_result`:

```json
{
  "type": "tool_result",
  "tool_use_id": "toolu_01ABC",
  "content": "[{\"title\":\"...\", \"url\":\"...\"}]"
}
```

| Rule | Detail |
| --- | --- |
| `tool_use_id` | **Exact match** to `tool_use.id` from prior assistant turn |
| Role | `tool_result` lives in a **`user`** message |
| Timing | Immediately after assistant message containing the `tool_use` |
| Count | One result per `tool_use` — no orphans either direction |

### Successful result content

`content` can be:

- **String** — JSON serialized, plain text, markdown tables
- **Array of content blocks** — e.g., text + image for multimodal tool outputs (per API support)

Keep payloads **focused** — truncate huge DB dumps; summarize in tool layer to save tokens.

### Error results: `is_error`

When execution fails:

```json
{
  "type": "tool_result",
  "tool_use_id": "toolu_01ABC",
  "is_error": true,
  "content": "HTTP 503 from payments API. Ref: req_8821. Safe to retry."
}
```

| Without `is_error` | With `is_error: true` |
| --- | --- |
| Model treats output as ground truth | Model knows call failed; can retry or explain |

Use for: exceptions, HTTP errors, validation failures, permission denied.

**Schema ≠ semantic truth:** even successful results may be wrong — your validator can return `is_error: true` for business rule violations.

### Parallel tool results

Assistant turn with `toolu_A` and `toolu_B` → one user message:

```json
{
  "role": "user",
  "content": [
    { "type": "tool_result", "tool_use_id": "toolu_A", "content": "..." },
    { "type": "tool_result", "tool_use_id": "toolu_B", "content": "...", "is_error": true }
  ]
}
```

Order typically matches tool_use order; IDs are the authoritative link.

### Optional user text in same message

Some patterns add a text block alongside tool results ("Here are the tool outputs") — follow current API docs for mixing blocks. Exam focus: **IDs must match**, role must be **user**.

### After tool_result: next API call

Append tool results → call Messages API again with **full history** including:

- All prior turns
- Latest assistant `tool_use` message
- Latest user `tool_result` message

Model continues until **`stop_reason: end_turn`**.

### What not to send

- Don't send tool results as **`assistant` role**
- Don't embed results only in a new user **text** message without `tool_result` blocks
- Don't change `tool_use_id` on retry — pair to original ID or re-run full turn

## Exam Notes

- **"Tool ran but next call fails invalid_request"** → mismatched or missing `tool_use_id`.
- **"Model assumes payment succeeded after API error"** → forgot `is_error: true`.
- **"Two tools, one result"** → incomplete parallel handling.
- **"Should you start a new conversation after tool failure?"** → No — append error `tool_result`, continue loop.
- **Validation failed but JSON parsed** → return `is_error: true` with explanation.

## Production Notes

- **Sanitize** tool output before returning — strip secrets, PII.
- Cap result size (e.g., 50KB); offer pagination tools for large datasets.
- Include machine-readable error codes in `content` for model recovery ("ERROR_CODE: TIMEOUT").
- Log tool latency and success separately from API latency.
- For idempotent tools, note retry safety in error content.
- Unit test history serializer — most agent bugs live here.

## Common Mistakes

- **`tool_use_id` typo** — #1 production bug.
- Skipping tool_result when tool throws exception — breaks next API call.
- Returning raw stack traces to model **and** user without sanitization.
- Success `tool_result` with empty string — model lacks context.
- Re-executing tools without appending prior round to history ("lost context" loops).
- Treating MCP tool results differently from API tool results — same `tool_result` shape in Messages API loop.

## See Also

- [Tool Use](./tool-use.md)
- [Agent Loop](./agent-loop.md)
- [Error Handling](../03-api/error-handling.md)
- [Messages API](../03-api/messages-api.md)
- [Agent Best Practices](./agent-best-practices.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
