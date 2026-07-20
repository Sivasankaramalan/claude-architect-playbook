# Tool Use

**Tool use** (function calling) lets Claude request structured actions via `tool_use` content blocks. You define tools with names, descriptions, and JSON Schema inputs; the model decides when and how to call them — unless you override with `tool_choice`.

**Domain focus:** Applications and Integration (33%) + Tools and MCPs (11%) + Agents (15%).

## Learning Objectives

After this page you should be able to:

- Define tools in the Messages API `tools` array with JSON Schema inputs.
- Read `tool_use` blocks from assistant responses.
- Configure `tool_choice`: `auto`, `any`, `tool`, `none`.
- Explain parallel tool calls and forced invocation patterns.
- Articulate that **schema validates structure, not semantic truth**.

## Key Ideas

### Tool definition shape

Each tool in the `tools` parameter:

```json
{
  "name": "get_weather",
  "description": "Get current weather for a city. Use when user asks about weather.",
  "input_schema": {
    "type": "object",
    "properties": {
      "city": { "type": "string", "description": "City name, e.g. Paris" },
      "units": { "type": "string", "enum": ["celsius", "fahrenheit"] }
    },
    "required": ["city"]
  }
}
```

| Field | Purpose |
| --- | --- |
| `name` | Stable identifier; matches `tool_use.name` |
| `description` | **Critical for model routing** — when to use this tool |
| `input_schema` | JSON Schema for arguments — types, required fields, enums |

Better descriptions beat clever prompts for tool selection accuracy.

### Model emits `tool_use`

Assistant `content` block:

```json
{
  "type": "tool_use",
  "id": "toolu_01XYZ...",
  "name": "get_weather",
  "input": { "city": "Paris", "units": "celsius" }
}
```

Response **`stop_reason: "tool_use"`** — your signal to execute and return [tool-results.md](./tool-results.md).

### `tool_choice`

| Value | Behavior |
| --- | --- |
| `{ "type": "auto" }` | Default — model may text or call tools |
| `{ "type": "any" }` | Must invoke at least one tool |
| `{ "type": "tool", "name": "get_weather" }` | Must call that specific tool |
| `{ "type": "none" }` | Cannot call tools this turn |

**Exam:** "Must guarantee JSON extraction" → define extraction tool + `tool_choice: any` or named `tool`.

### Parallel tool calls

The model may return **multiple `tool_use` blocks** in one assistant message:

- Efficient — one API round for independent lookups
- Your executor runs each tool; all `tool_result` blocks go in the **next user message**
- Each result **`tool_use_id`** must match the corresponding block `id`

### Schema vs semantics

JSON Schema ensures:

- Required fields present
- Types roughly correct (string vs number)

It does **NOT** ensure:

- Values are **factually correct** (wrong city, hallucinated ID)
- Business rules hold (negative price, unauthorized ID)

**Always validate in application code** after tool execution and before trusting data for side effects.

### Tools vs MCP

- **Tools parameter** — inline definitions on Messages API call.
- **MCP (Model Context Protocol)** — standardized way to expose tools/resources from servers; client still runs agent loop — [../06-mcp/tools.md](../06-mcp/tools.md).

Same lifecycle: `tool_use` → execute → `tool_result` → continue.

### Structured output pattern

Define a tool like `record_findings` with strict schema instead of asking for raw JSON in text — combine with `tool_choice` for reliability.

### Token cost

Full tool schemas ship on **every request** — keep descriptions clear but concise; avoid megabyte schemas.

## Exam Notes

- **"Model answers instead of querying DB"** → improve tool description + `tool_choice: any`.
- **"Force specific API call"** → `tool_choice: { type: "tool", name: "..." }`.
- **"Disable tools for final summary turn"** → `tool_choice: none`.
- **"Valid JSON but wrong data"** → schema ≠ truth — add validation layer.
- **Multiple lookups** → parallel tool calls in one turn beats serial agent iterations when independent.

## Production Notes

- **Validate `input`** before executing — treat as untrusted (injection into SQL, shell, paths).
- Version tool schemas; log `name` + hashed input for audit.
- Implement timeouts and circuit breakers per tool.
- Separate read vs write tools; require approval for writes.
- Use enums and `required` aggressively in schema — reduces malformed calls.
- Cache idempotent read tool results within a single user request when safe.

## Common Mistakes

- Vague `description` ("helper tool") → model never calls it.
- Omitting **`required`** fields → model skips important args.
- Expecting schema to prevent **hallucinated IDs**.
- Defining tools but never passing them on **subsequent** loop iterations (tools must stay in request unless intentionally removed).
- Confusing **`tool_choice: none`** with removing tools from the request entirely.
- Returning tool output as plain assistant text instead of **`tool_result`** blocks.

## See Also

- [Tool Results](./tool-results.md)
- [Agent Loop](./agent-loop.md)
- [Messages API](../03-api/messages-api.md)
- [MCP Tools](../06-mcp/tools.md)
- [Error Handling](../03-api/error-handling.md)
