# MCP Introduction

The **Model Context Protocol (MCP)** is an open standard for connecting AI applications to external **tools**, **resources**, and **prompts**. Anthropic introduced MCP so Claude (and other hosts) can integrate with databases, APIs, and enterprise systems through a shared protocol instead of one-off plugins.

Public reference: [modelcontextprotocol.io](https://modelcontextprotocol.io/docs/getting-started/intro).

## Learning Objectives

- Define MCP and why it exists (standardized context + actions for LLM apps)
- Name the three MCP primitives: **tools**, **resources**, **prompts**
- Identify **host**, **client**, and **server** roles
- Map MCP to CCDV F domain **Tools and MCPs (10.6%)**
- Contrast MCP with ad-hoc function calling and with Claude Code Skills

## Key Ideas

### Problem MCP solves

Before MCP, every IDE or agent framework built custom connectors. MCP standardizes:

- **Discovery** — list available tools/resources/prompts
- **Invocation** — JSON-RPC messages over a defined transport
- **Security boundaries** — auth, least privilege, structured errors

Build an MCP server once; connect it to Claude Desktop, Claude Code, Cursor, Agent SDK, and other MCP hosts.

### Architecture at a glance

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   MCP Host  │────▶│ MCP Client  │────▶│ MCP Server  │
│ (Claude Code│     │ (1:1 per    │     │ (your API,  │
│  Desktop…)  │     │  server)    │     │  DB, files) │
└─────────────┘     └─────────────┘     └─────────────┘
       │                   │                   │
       └─────── LLM reasons about tool calls ───┘
```

- **Host** — user-facing app (Claude Code, IDE, custom agent)
- **Client** — connector inside host; one client per server connection
- **Server** — exposes capabilities; enforces auth and validation

### Three primitives

| Primitive | Control | Purpose | REST analogy |
| --- | --- | --- | --- |
| **Tools** | Model-driven | Actions with side effects | POST |
| **Resources** | App-driven | Read-only context | GET |
| **Prompts** | User-driven | Reusable templates | Workflow presets |

Also in the spec: **sampling** (server requests model completion from client) — advanced; know it exists for completeness.

### MCP in the Claude ecosystem

| Integration | Role |
| --- | --- |
| Claude Code | Host + clients; `.mcp.json` / settings register servers |
| Agent SDK | `mcpServers` in options; in-process servers supported |
| Messages API | Your app implements client loop; MCP feeds tool definitions |

MCP does **not** replace the Messages API — it **feeds** tools and context into the agent loop.

### MCP vs built-in Claude Code tools

Built-in tools (Read, Bash, …) ship with the harness. **MCP extends** capability to your systems — Jira, Postgres, internal APIs — without forking Claude Code.

## Exam Notes

- **Wrong tool picked** → often **tool description** problem, not “bigger model.”
- **Read-only doc lookup** → **resource**, not tool (unless mutation needed).
- **Shared workflow template** → **prompt** primitive — still doesn’t *enforce* behavior alone.
- **Team server config** → project-level registration, not user-only config.
- MCP **tools** perform actions; confusing tools with resources is a common distractor.

## Production Notes

- Start with **few, sharp tools**; add resources for heavy read-only data.
- Version server capabilities; breaking schema changes break agents silently.
- Run MCP servers with **least-privilege** credentials scoped per environment.
- Monitor tool latency — slow tools dominate agent turn time.
- Document server ownership and on-call — MCP is production infrastructure.

## Common Mistakes

- One mega-tool that “does everything” — hurts model routing
- Using tools for pure reads that should be **resources**
- Expecting MCP prompts alone to **guarantee** compliance (use hooks + validation)
- Committing API keys in MCP config instead of env vars
- Assuming all hosts support identical transport/auth features without testing

## See Also

- [MCP Architecture](./architecture.md)
- [MCP Tools](./tools.md)
- [MCP Resources](./resources.md)
- [MCP Prompts](./prompts.md)
- [MCP Transport](./transport.md)
- [Skills vs MCP](../07-skills/skills-vs-mcp.md)
- [Tool Use](../04-agent-engineering/tool-use.md)
