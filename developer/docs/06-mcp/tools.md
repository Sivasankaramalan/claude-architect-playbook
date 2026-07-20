# MCP Tools

**MCP tools** are executable functions the **model** can invoke via `tool_use` — search, create, update, call APIs, run queries. Tool **names**, **descriptions**, and **input schemas** are the primary mechanism for correct tool selection in agent loops.

Public reference: MCP tool definitions in the [specification](https://modelcontextprotocol.io/specification/2025-11-25/server/tools).

## Learning Objectives

- Design narrow tools with JSON Schema inputs and clear descriptions
- Return structured content and **structured errors** from `tools/call`
- Relate MCP tools to Messages API tool use and `stop_reason`
- Scope tools per agent role (4–5 tools, not entire catalog)
- Validate business rules server-side after schema validation

## Key Ideas

### Tool definition anatomy

Each tool in `tools/list` includes:

| Field | Purpose |
| --- | --- |
| `name` | Stable identifier for `tool_use` |
| `description` | **Primary routing signal** for the model |
| `inputSchema` | JSON Schema — types, required fields, per-field descriptions |

Descriptions should state:

- **What** the tool does
- **When to use** it
- **When NOT to use** it (point to sibling tools)
- **Input expectations** and **output shape**
- **Edge cases** (empty results, idempotency)

### Tool call flow

1. Model emits `tool_use` with `name` + `input`.
2. Client sends `tools/call` to MCP server.
3. Server validates, executes, returns **content** (text/image/resource refs) or **isError** structured result.
4. Host wraps as `tool_result` in conversation history.
5. Loop continues while `stop_reason == tool_use`.

### Structured errors (exam favorite)

Return errors the model can act on — not raw stack traces:

```json
{
  "isError": true,
  "content": [
    {
      "type": "text",
      "text": "{\"category\":\"AUTH\",\"retryable\":false,\"message\":\"Token expired. Re-authenticate.\",\"hint\":\"Run auth tool first.\"}"
    }
  ]
}
```

Include:

- **category** — AUTH, VALIDATION, UPSTREAM, RATE_LIMIT, …
- **retryable** — whether agent should retry same call
- **message** — user-safe text
- **hint** — recovery guidance for the model

### Schema design tips

- Prefer **nullable optional** fields over required fields when data may be absent — required missing data forces hallucination (`N/A`, `UNKNOWN`).
- **`tool_use` guarantees JSON shape**, not semantic truth — validate business rules in code.
- Split overlapping tools — `search_users` vs `get_user_by_id` beats `users`.

### Overlap with Messages API tools

MCP tools surface to the model like native tools — same loop semantics:

- `tool_choice: auto` — model decides
- `tool_choice: any` / forced tool — when exam requires guaranteed call
- Hooks apply to MCP tool names in PreToolUse matchers

### Tool permissions

Combine:

- **Server** — credential scope, input validation
- **Host allowlists** — SDK `allowedTools`, Claude Code permissions
- **Hooks** — deny prod writes, secret paths

Agents should not all receive every MCP tool — **scope by responsibility**.

## Exam Notes

- **Wrong tool selected** → improve **descriptions** and reduce overlap first.
- **Hallucinated field values** → too many **required** schema fields.
- **Production reliability** → structured errors + retryable flag — not “tell model to try again.”
- **`tool_choice: auto`** while stem requires **guaranteed** tool → wrong unless hooks force path.
- Semantic validation → **server code**, not description alone.

## Production Notes

- Integration-test each tool with invalid inputs before launch.
- Log tool name, duration, error category — not full PII payloads.
- Idempotent tools for create operations where agents retry loops.
- Document breaking schema changes; version server package.
- Rate-limit destructive tools separately from reads.
- PostToolUse hooks normalize MCP JSON before model sees secrets.

## Common Mistakes

- Giant `do_everything` tool — model routing fails randomly
- Identical descriptions on different tools
- Returning 500 HTML pages as tool text
- Required fields for data the model must discover via another call first
- Giving every subagent all MCP tools “for flexibility”

## See Also

- [MCP Servers](./servers.md)
- [MCP Resources](./resources.md)
- [MCP Security](./security.md)
- [MCP Best Practices](./best-practices.md)
- [Tool Use](../04-agent-engineering/tool-use.md)
- [Tool Results](../04-agent-engineering/tool-results.md)
- [Hooks](../05-sdk/hooks.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
