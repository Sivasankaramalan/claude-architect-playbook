# SDK Overview

The **Claude Agent SDK** (formerly Claude Code SDK) is Anthropic’s library for building production agents that use the same agent loop, built-in tools, and context management as **Claude Code**. For CCDV F, distinguish three integration surfaces: the **Messages API** (low-level HTTP), the **Agent SDK** (programmatic agent harness), and the **Claude Code CLI** (interactive or headless terminal workflow).

## Learning Objectives

- Compare Messages API, Agent SDK, and Claude Code CLI — when each is the right choice
- Name the Agent SDK packages and primary entry point (`query`)
- Explain how SDK agents relate to tool use, MCP, hooks, sessions, and subagents
- Map SDK topics to exam domains (Applications, Agents, Tools & MCP, Claude Code)

## Key Ideas

### Three integration layers

| Layer | What you control | Best for |
| --- | --- | --- |
| **Messages API** | Every request, tool schema, loop, retries | Custom backends, non-agent chat, strict orchestration |
| **Agent SDK** | Prompt + options; harness runs the loop | Autonomous agents in your app (Python / TypeScript) |
| **Claude Code CLI** | Project files, rules, skills, hooks, MCP config | Developer workflows, CI, headless `-p` runs |

The Agent SDK is **not** a replacement for the Messages API — it **wraps** the same model + tool-use primitives inside a maintained agent loop.

### Agent SDK packages

| Language | Package | Notes |
| --- | --- | --- |
| Python | `claude-agent-sdk` | Requires Python 3.10+; bundles Claude Code CLI binary |
| TypeScript | `@anthropic-ai/claude-agent-sdk` | Node 18+; optional bundled CLI binary |

Primary API: **`query(prompt, options)`** — returns an async stream of messages as the agent works.

### What the SDK gives you out of the box

- **Built-in tools** — Read, Write, Edit, Bash, Glob, Grep, WebSearch, WebFetch, and orchestration tools (subagents, etc.)
- **MCP integration** — register MCP servers in options (`mcpServers`); supports in-process servers
- **Permission modes** — e.g. `acceptEdits`, tool allowlists (`allowedTools` / `allowed_tools`)
- **Hooks** — SDK callbacks + shell hooks from settings files (see [hooks.md](./hooks.md))
- **Sessions** — resume prior agent runs with session IDs (see [sessions.md](./sessions.md))
- **Subagents** — delegate isolated subtasks with explicit context passing

### Mental model for the exam

```
Your app  →  Agent SDK (query + options)  →  Agent loop  →  Messages API  →  Claude
                    ↑                              ↓
              hooks, MCP, tools            stop_reason drives loop
```

When a question asks how to **ship an autonomous coding agent inside your product**, lean Agent SDK. When it asks about **raw API parameters** (`tool_choice`, streaming, batches), lean Messages API. When it asks about **team repo conventions or CLAUDE.md**, lean Claude Code config (often used *with* the SDK via `settingSources`).

## Exam Notes

- **Agent SDK vs Messages API**: SDK = pre-built loop + tools; Messages API = you implement the loop.
- **Agent SDK vs Claude Code CLI**: Same harness; SDK embeds it in code; CLI is terminal-first (also runnable headless with `-p`).
- **`stop_reason`** still governs the agent loop inside the SDK — `tool_use` continues, `end_turn` stops.
- Questions about **guaranteed** behavior → hooks, permissions, allowlists — not “stronger system prompt.”
- SDK agents can load **project settings** (hooks, MCP) when `settingSources` / `setting_sources` includes `project` or `local`.

## Production Notes

- Set **`ANTHROPIC_API_KEY`** (or platform auth) in the runtime environment — never commit keys.
- Use **`allowedTools`** / **`permissionMode`** to scope blast radius before production.
- Prefer **structured logging** around `query()` streams; correlate with session IDs for support.
- For CI/CD, combine headless CLI or SDK with **deterministic hooks** (lint, test, secret scan).
- Metering: agentic SDK usage may be billed separately from interactive Claude Code — verify current plan docs.
- Keep MCP servers **project-scoped** in `.mcp.json` / settings committed to the repo; personal servers in user config.

## Common Mistakes

- Using the Messages API and re-implementing the entire agent loop when the SDK already fits the use case
- Assuming SDK subagents **inherit** parent conversation history (they do not — pass context explicitly)
- Confusing **SDK memory** (CLAUDE.md / rules loaded by harness) with **API conversation history**
- Disabling hooks/settings sources in SDK options, then expecting Claude Code project config to apply
- Treating built-in Bash/Write tools as safe without permission modes or PreToolUse hooks

## See Also

- [Agent SDK](./agent-sdk.md)
- [Hooks](./hooks.md)
- [Sessions](./sessions.md)
- [Memory](./memory.md)
- [Tracing](./tracing.md)
- [MCP Introduction](../06-mcp/introduction.md)
- [Skills Overview](../07-skills/overview.md)
- [Messages API](../03-api/messages-api.md)
- [Agent Loop](../04-agent-engineering/agent-loop.md)
