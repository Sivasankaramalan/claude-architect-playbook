# MCP Transport

MCP messages are **JSON-RPC** framed by a **transport**. CCDV F focuses on **when** to use each transport and deployment implications — not low-level byte protocols.

Public reference: [MCP transports](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports).

## Learning Objectives

- Compare **stdio** and **Streamable HTTP** transports
- Explain how local subprocess servers differ from remote HTTP services
- Relate transport choice to auth, scaling, and latency
- Recognize stderr vs stdout rules for stdio servers
- Avoid obsolete “HTTP+SSE only” patterns where Streamable HTTP applies

## Key Ideas

### Two standard transports

| Transport | Mechanism | Typical deployment |
| --- | --- | --- |
| **stdio** | Client spawns server subprocess; JSON-RPC on **stdin/stdout** | Local dev, Claude Code, single-machine agents |
| **Streamable HTTP** | HTTP **POST** to MCP endpoint; optional **SSE** streams for server push | Remote shared services, multi-client, OAuth |

Both carry the same **data layer** (tools/resources/prompts) — transport is plumbing.

### stdio details

- Client launches server: `command` + `args` (+ `env`) in config.
- Server reads JSON-RPC from **stdin**, writes responses to **stdout**.
- **stderr** only for human logs — never protocol on stdout.
- One subprocess ↔ one client connection — simple, low overhead, no network auth.

Best for: filesystem tools, local scripts, developer machines, CI runners with bundled binaries.

### Streamable HTTP details

- Server exposes single **MCP endpoint** (e.g. `https://api.example.com/mcp`).
- Client sends each JSON-RPC message as **HTTP POST**.
- Server may respond with JSON body or **SSE** stream for multiple messages / notifications.
- Supports standard HTTP auth: bearer tokens, API keys, **OAuth** (recommended for remote servers in spec guidance).

Best for: shared CRM connector, org-wide data plane, SaaS integrations, horizontal scale.

**Note:** Streamable HTTP (2025-03+) supersedes older **HTTP+SSE** split-endpoint patterns from early MCP revisions — exam answers should favor current spec naming.

### Choosing transport (decision table)

| Requirement | Prefer |
| --- | --- |
| Fast local file access | stdio |
| Central creds for whole company | HTTP + OAuth |
| Single developer laptop | stdio |
| Many hosts / users same server | HTTP |
| No network exposure | stdio |
| Load-balanced HA service | HTTP |

### Auth and transport

- **stdio** — inherits OS user env vars; secrets in env, not argv.
- **HTTP** — tokens per request; rotate keys; mTLS for high assurance.

See [MCP Security](./security.md).

### Agent SDK note

SDK supports stdio servers via config and **in-process** MCP (TypeScript) to skip subprocess cost. HTTP servers register with URL + auth headers in options.

### Protocol headers (awareness)

Recent Streamable HTTP revisions define headers like `Mcp-Method` and optional parameter-to-header mappings (`x-mcp-header` in schemas) for gateways — know transports evolve; verify spec version in production.

## Exam Notes

- **Local single-user tool** → **stdio** distractor usually wins over HTTP.
- **Shared remote integration** → **HTTP** / Streamable HTTP.
- **stdout pollution** breaks **stdio** servers — log to stderr.
- Transport questions test **architecture fit**, not memorizing header names.
- Client **spawns** subprocess for stdio — server isn’t manually started separately in typical Claude Code flow.

## Production Notes

- TLS 1.2+ for all remote MCP endpoints.
- Timeouts on HTTP POST — agents block on slow tools.
- Health endpoints separate from MCP path for k8s probes.
- For stdio, set ulimits and kill hung subprocesses.
- Document which transport each environment uses — avoid prod accidentally on local stdio paths.

## Common Mistakes

- `console.log` on stdout in Node stdio servers — breaks JSON-RPC
- Exposing unauthenticated HTTP MCP to the public internet
- Using remote HTTP for purely local file reads — unnecessary latency and auth burden
- Confusing MCP transport with **Messages API** streaming — different systems
- Assuming sticky sessions required for all HTTP MCP (newer spec trends stateless — verify your revision)

## See Also

- [MCP Architecture](./architecture.md)
- [MCP Servers](./servers.md)
- [MCP Security](./security.md)
- [Agent SDK](../05-sdk/agent-sdk.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
