# Agent SDK

The **Claude Agent SDK** exposes Anthropic’s production agent harness as a library. You call **`query()`** with a task prompt and **options**; the SDK runs the agentic loop, executes tools, and streams results until `stop_reason` is `end_turn`.

Public docs: [Agent SDK overview](https://code.claude.com/docs/en/agent-sdk) (Python and TypeScript).

## Learning Objectives

- Use `query()` and interpret streamed agent messages
- Configure `allowedTools`, `permissionMode`, `systemPrompt`, and `mcpServers`
- Explain built-in tools and when to add MCP servers
- Describe subagent delegation and context isolation
- Connect SDK control flow to `stop_reason` and tool results

## Key Ideas

### Minimal pattern (conceptual)

**Python:**

```python
from claude_agent_sdk import query, ClaudeAgentOptions

async for message in query(
    prompt="Find and fix the failing test in tests/",
    options=ClaudeAgentOptions(
        allowed_tools=["Read", "Edit", "Bash", "Glob"],
        permission_mode="acceptEdits",
    ),
):
    print(message)
```

**TypeScript:**

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Find and fix the failing test in tests/",
  options: {
    allowedTools: ["Read", "Edit", "Bash", "Glob"],
    permissionMode: "acceptEdits",
  },
})) {
  console.log(message);
}
```

### Core options (exam-relevant)

| Option | Purpose |
| --- | --- |
| `allowedTools` / `allowed_tools` | Pre-approve tool subset — least privilege |
| `permissionMode` / `permission_mode` | How file/shell permissions are handled |
| `systemPrompt` / `system_prompt` | Agent-level instructions (not a substitute for hooks) |
| `mcpServers` / `mcp_servers` | Attach MCP tool/resource/prompt providers |
| `hooks` | SDK callback hooks (PreToolUse, PostToolUse, …) |
| `settingSources` / `setting_sources` | Load `.claude/settings.json`, hooks, MCP from project |
| `resume` / session id | Continue a prior run (see [sessions.md](./sessions.md)) |
| `model` | Claude model id for the agent |

### Built-in tools (representative)

| Tool | Role |
| --- | --- |
| Read / Write / Edit | File I/O and patches |
| Bash | Shell commands — high risk; gate with hooks |
| Glob / Grep | Codebase search |
| WebSearch / WebFetch | External information |
| Subagent tools | Spawn isolated workers for subtasks |

Built-in tools are **not** unlimited authority — combine allowlists, permission mode, and hooks for production.

### Agent loop (same as Messages API semantics)

1. SDK sends messages + tool definitions to Claude.
2. Model returns content and/or `tool_use` blocks.
3. If `stop_reason == tool_use`, SDK runs tools, appends **`tool_result`**, calls again.
4. If `stop_reason == end_turn`, loop ends.

You normally **do not** manually append tool results when using `query()` — the harness does it.

### Subagents

Subagents solve **context isolation** and **parallel specialization**:

- Coordinator delegates via subagent tool calls.
- Subagent gets **only** the context you pass — not full parent history.
- Each subagent should have a **scoped tool set** (often 4–5 tools, not everything).

Exam pattern: “Reduce context pollution for code review” → subagent with fresh context + review-specific tools.

### MCP in the SDK

Register servers in options so the agent discovers tools at runtime:

- **stdio** — local subprocess server (common in dev)
- **HTTP / Streamable HTTP** — remote shared services
- **In-process MCP** — SDK can host MCP without subprocess overhead (TypeScript)

See [MCP Servers](../06-mcp/servers.md) for server-side design.

### Headless alternative

For languages without the SDK, public docs note running **Claude Code CLI** with `-p` (prompt) and `--output-format json` — same harness, different embedding.

## Exam Notes

- **`query()`** is the SDK entry point — not raw `messages.create()` unless the question is Messages API domain.
- **`allowedTools`** = programmatic scoping; stronger than telling the model “only use Read.”
- **Subagents ≠ shared memory** — explicit context passing is the correct answer class.
- **`tool_choice`** lives at Messages API level; SDK wraps it — know both layers.
- For “autonomous fix bugs in repo” stems → Agent SDK or Claude Code, not prompt-only chat.

## Production Notes

- Start with **minimal tool allowlist**; expand only when tasks fail for lack of capability.
- Run **PreToolUse hooks** for destructive Bash, prod DB writes, or secret paths.
- Use **`permissionMode`** appropriate to environment (`plan` / ask in staging, stricter in prod).
- Stream messages to UX or logs — don’t wait for full completion on long runs.
- Pass **`cwd` / working directory** intentionally; agents operate relative to project root.
- Version-pin SDK packages; Anthropic ships frequent updates aligned with Claude Code.
- For long jobs, persist **session id** to resume after failure (see [sessions.md](./sessions.md)).

## Common Mistakes

- Giving every agent all built-in tools “to avoid routing complexity”
- Expecting subagents to see coordinator tool results automatically
- Using `systemPrompt` alone to block dangerous commands instead of PreToolUse hooks
- Forgetting to enable `settingSources` when relying on repo `.claude/settings.json`
- Mixing up **SDK `query()`** with **Messages API** manual loops in exam answers

## See Also

- [SDK Overview](./sdk-overview.md)
- [Hooks](./hooks.md)
- [Sessions](./sessions.md)
- [Tool Use](../04-agent-engineering/tool-use.md)
- [Multi-Agent](../04-agent-engineering/multi-agent.md)
- [MCP Introduction](../06-mcp/introduction.md)
