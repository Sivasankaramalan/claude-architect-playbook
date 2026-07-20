# MCP Architecture

MCP separates **host applications**, **clients**, and **servers** across a **data layer** (JSON-RPC protocol, primitives) and **transport layer** (how bytes move). Understanding this split is core to CCDV F **Tools and MCPs** questions.

Public reference: [MCP architecture overview](https://modelcontextprotocol.io/docs/learn/architecture).

## Learning Objectives

- Diagram host → client → server relationships
- Explain 1:1 client-to-server connections
- Describe lifecycle: initialize, capability negotiation, invocation, shutdown
- Place tools, resources, prompts, and sampling in the protocol
- Choose stdio vs remote HTTP transport for a scenario

## Key Ideas

### Roles

| Component | Responsibility | Examples |
| --- | --- | --- |
| **Host** | UI + LLM orchestration; may run multiple clients | Claude Code, Claude Desktop, custom agent app |
| **Client** | Maintains connection to **one** server; lists & invokes primitives | Inside host process or subprocess bridge |
| **Server** | Exposes tools/resources/prompts; validates & executes | `@modelcontextprotocol/server-*`, custom Node/Python server |

One host often runs **many clients** — one per MCP server — but each client talks to exactly **one** server.

### Protocol layers

**Data layer (JSON-RPC):**

- Lifecycle (`initialize`, capability exchange)
- `tools/list`, `tools/call`
- `resources/list`, `resources/read`
- `prompts/list`, `prompts/get`
- Notifications (progress, logging)
- **Sampling** — server asks client/host to run an LLM completion (reverse direction)

**Transport layer:**

- **stdio** — local subprocess, stdin/stdout JSON-RPC lines
- **Streamable HTTP** — remote server, POST + optional SSE streams

Transport choice affects deployment, auth, and scaling — not the semantic tool/resource model.

### Typical call flow (tool)

1. Host loads MCP server (spawn subprocess or HTTP connect).
2. Client **initializes** session and discovers `tools/list`.
3. Host merges tool definitions into model context (names, descriptions, input schemas).
4. Model returns `tool_use` for an MCP tool.
5. Host/client sends `tools/call` to server.
6. Server validates input, executes, returns structured **content** or **structured error**.
7. Host appends `tool_result` to conversation; model continues until `end_turn`.

Same loop as native API tools — MCP standardizes step 2–6 for external systems.

### Local vs remote topology

| Pattern | Transport | Typical use |
| --- | --- | --- |
| Local dev integration | stdio | Filesystem, local scripts, single-user |
| Shared team service | Streamable HTTP | Central CRM connector, multi-user auth |
| IDE + Claude Code | Often stdio per server | Each server = child process |

Local stdio servers usually serve **one** client; remote HTTP servers often serve **many**.

### Sampling (awareness)

**Sampling** lets an MCP server request a language model completion through the **client** — useful when server logic needs LLM help but shouldn’t embed API keys. Less common on foundational exams than tools/resources, but listed in official MCP materials.

### Integration with Claude agent loop

MCP tools appear alongside built-in tools in the same **`stop_reason` loop**:

- `tool_use` → execute MCP or built-in → append result → continue
- `end_turn` → stop

Hooks can intercept MCP tool names like any other tool (PreToolUse / PostToolUse).

## Exam Notes

- **Client : Server** = **1 : 1** — not one client to many servers in a single connection.
- **Host** orchestrates the model; **server** does not call Claude directly (except via sampling path).
- **stdio** = local subprocess; **HTTP** = remote/multi-tenant patterns.
- Tool definitions come from **`tools/list`** — stale server deploy = wrong tools in model context.
- Architecture questions favor **separation of concerns** (validation on server, not only in prompt).

## Production Notes

- Run remote MCP behind **TLS**, auth, and rate limits.
- Health-check servers independently of the LLM app.
- Use **circuit breakers** when upstream APIs fail — return structured errors quickly.
- Deploy server and host **version compatibility** matrix (protocol revision).
- For stdio, supervise subprocess lifecycle (restart on crash, resource limits).

## Common Mistakes

- Putting business validation **only** in the host prompt, not on the server
- One client trying to multiplex multiple unrelated backends without separate servers
- Confusing **MCP host** with **MCP server** in exam diagrams
- Using HTTP transport locally when stdio is simpler and faster (over-engineering)
- Ignoring **initialize** failures — silent empty tool list

## See Also

- [MCP Introduction](./introduction.md)
- [MCP Servers](./servers.md)
- [MCP Transport](./transport.md)
- [MCP Tools](./tools.md)
- [MCP Security](./security.md)
- [Agent Loop](../04-agent-engineering/agent-loop.md)
