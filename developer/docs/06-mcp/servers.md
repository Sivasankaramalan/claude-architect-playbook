# MCP Servers

An **MCP server** is the process (or service) that **implements** tools, resources, and prompts and enforces **auth, validation, and error shape** at the trust boundary. The host’s client discovers capabilities; the server owns side effects.

## Learning Objectives

- List what a production MCP server must implement
- Register servers in Claude Code / Agent SDK configuration
- Separate server responsibilities from host prompt engineering
- Apply least privilege at the server credential layer
- Return structured errors consumable by the model and hooks

## Key Ideas

### Server responsibilities

| Concern | Server-side (required) | Host-side (supplement) |
| --- | --- | --- |
| Input validation | JSON Schema + business rules | Tool allowlists, hooks |
| Authorization | OAuth tokens, API keys via env | User approval UI |
| Side effects | Execute mutations | PreToolUse deny |
| Error shape | Structured MCP errors | Logging, retries |
| Auditing | Server logs with actor/context | Session tracing |

**Exam bias:** Side effects and secrets stop at the **server** — not “trust the model.”

### What servers expose

Implement handlers for protocol methods such as:

- `tools/list` + `tools/call`
- `resources/list` + `resources/read` (+ optional templates/subscribe)
- `prompts/list` + `prompts/get`

Use official SDKs (`@modelcontextprotocol/sdk` for TypeScript, Python SDK) to reduce boilerplate.

### Registration in Claude Code (conceptual)

Project-scoped config (committed) vs user-scoped (personal debug):

```json
{
  "mcpServers": {
    "issue-tracker": {
      "command": "npx",
      "args": ["-y", "@company/mcp-jira"],
      "env": {
        "JIRA_TOKEN": "${JIRA_TOKEN}"
      }
    }
  }
}
```

Exact file location may be `.mcp.json`, `.claude/settings.json`, or merged settings — follow current Claude Code docs. **Team servers → project config.**

### Agent SDK registration

Pass `mcpServers` / `mcp_servers` in `query()` options — same conceptual map: command/args/env for stdio, or URL + headers for HTTP.

TypeScript also supports **in-process** MCP servers (no subprocess) for lower latency.

### Server design principles

1. **Narrow tools** — `create_ticket` not `jira_do_everything`
2. **Typed inputs** — JSON Schema with descriptions per field
3. **Predictable outputs** — JSON the model can chain on
4. **Structured errors** — category, retryable flag, safe message
5. **Idempotent** where possible — agents retry

### Process model (stdio)

Host/client **spawns** server as subprocess:

- stdin/stdout carry JSON-RPC
- Server logs diagnostics to **stderr** (not stdout — corrupts protocol)
- Crash or hang → host sees missing tools; design restart policy

### Service model (HTTP)

Server runs independently:

- Multiple hosts connect with auth
- Scale horizontally behind load balancer
- OAuth / bearer tokens per MCP HTTP spec guidance

## Exam Notes

- **Structured errors** are implemented on the **server**, not by pasting stack traces into tool results.
- **Secrets** via environment variables referenced in config — never committed literals.
- **Personal experimental server** → user config; **team CRM connector** → project config.
- Server validates **even if** the model sent well-formed JSON — semantic validation still required.
- Vague tools are a **design** defect at list time — fix descriptions in `tools/list`.

## Production Notes

- Scope OAuth tokens to minimum API scopes per tool group.
- Rate-limit per tenant; agents can hammer tools in loops.
- Version your server package; announce breaking schema changes.
- Integration tests: call each tool with golden inputs + forbidden inputs.
- Run server with read-only DB roles where tools only query.
- Separate **dev/staging/prod** server endpoints and credentials.

## Common Mistakes

- Returning raw exception strings as tool content
- One server with 40 overlapping tools
- stdout logging in stdio servers (breaks JSON-RPC)
- No auth on HTTP MCP endpoint exposed to network
- Duplicating the same integration as both MCP tool and unrelated REST call without single validation path

## See Also

- [MCP Architecture](./architecture.md)
- [MCP Tools](./tools.md)
- [MCP Transport](./transport.md)
- [MCP Security](./security.md)
- [MCP Best Practices](./best-practices.md)
- [Agent SDK](../05-sdk/agent-sdk.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
