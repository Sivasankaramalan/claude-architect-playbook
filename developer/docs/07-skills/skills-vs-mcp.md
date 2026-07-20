# Skills vs MCP

**Skills** and **MCP** solve different problems in Claude agent systems. Skills package **how to work** (procedures, scripts, templates inside the repo). MCP connects agents to **external systems** (APIs, databases, enterprise services) through a standard protocol.

CCDV F tests both under related domains — choose the right layer, or combine them.

## Learning Objectives

- Contrast Skills (harness workflows) vs MCP (external integrations)
- Map control models: on-demand procedures vs model-invoked tools
- Explain when to implement MCP server vs Skill with Bash scripts
- Avoid anti-patterns (curl in Skill vs proper MCP tool)
- Design systems using Skills + MCP + hooks together

## Key Ideas

### Side-by-side comparison

| Dimension | Claude Code Skills | MCP |
| --- | --- | --- |
| **Purpose** | Repeatable workflows in codebase | External data & actions |
| **Protocol** | Claude harness convention | Open MCP standard (JSON-RPC) |
| **Discovery** | Skill `description` in SKILL.md | `tools/list`, `resources/list`, `prompts/list` |
| **Activation** | Task match / Skill tool | Model `tool_use` (tools) |
| **Portability** | Claude Code / Agent SDK | Any MCP host (Desktop, IDE, custom) |
| **Side effects** | Via built-in tools (Bash, Edit) | Via server-implemented tools |
| **Enforcement** | Soft (guidance) | Server validation + hooks |
| **Best content** | Steps, scripts, templates | API wrappers, DB queries, auth |

### Mental model

```
Skill  = "Playbook in the repo"
MCP    = "Hands into your systems"
Hooks  = "Hard rails on tool execution"
Rules  = "Always-on conventions"
```

### When MCP wins

- Needs **authentication** to SaaS (OAuth, API keys)
- **Structured errors** and validation at trust boundary
- **Reuse** same integration across Claude Desktop, Code, custom apps
- **Read-only corpora** via resources + mutations via tools
- **Central ops** — one HTTP MCP service for whole org

### When Skills win

- Workflow is **repo-specific** (branch naming, release steps)
- Procedures reference **local scripts** and templates
- No new network service justified for simple automation
- Team wants version-controlled **runbook** beside code
- Task is **infrequent** — on-demand load saves context

### Gray area: Skill with Bash curl

Anti-pattern for production:

- Skill script calls `curl` to internal API with pasted tokens
- No structured errors, no central auth, breaks across hosts

Prefer **MCP server** wrapping the API — one validation path, reusable, auditable.

Acceptable: Skill script runs **local** deterministic tools (`npm test`, `terraform plan`) with hooks guarding flags.

### Skills vs MCP prompts

Both offer “templates,” but:

- **MCP prompts** — protocol primitive; user/host selects before chat; any MCP client
- **Skills** — Claude harness; richer bundles (scripts, multi-file)

Exam: **Claude Code workflow in repo** → Skill; **cross-host reusable template from server** → MCP prompt.

### Skills vs MCP tools (orchestration)

| Need | Layer |
| --- | --- |
| “Call `create_ticket` with schema” | MCP tool |
| “Follow our 12-step incident process” | Skill |
| “Never call `create_ticket` on weekends” | Hook + server rule |

### Agent SDK unifies both

`query()` options accept **`mcpServers`** and harness **Skills** — design composable stacks, not either/or religion.

## Exam Notes

- **CRM / database / ticket system** → MCP, not Skill-only.
- **Release checklist with local scripts** → Skill.
- **Guarantee** behavior → hooks — neither Skill nor MCP prompt alone.
- **Wrong tool selected** — MCP description issue; Skills issue = bad `description` in SKILL.md.
- Don’t pick MCP when stem is purely **CLAUDE.md team conventions**.

## Production Notes

- One integration → one MCP server; many workflows → Skills referencing those tools.
- Document in each Skill which MCP tools it expects (`issue-tracker/create_ticket`).
- Integration tests for MCP; skill eval tasks for procedures.
- Avoid duplicating API client logic in Skill scripts **and** MCP server — DRY at MCP layer.

## Common Mistakes

| Mistake | Better approach |
| --- | --- |
| MCP server for markdown coding standards | `.claude/rules/` |
| Skill with hardcoded API tokens | MCP + env auth |
| Skill instead of MCP for shared org API | Central MCP service |
| MCP tool that runs 12-step human procedure text | Skill + small MCP tools |
| Assuming Skills work in non-Claude hosts | Use MCP for portability |

## See Also

- [Skills Overview](./overview.md)
- [When to Use Skills](./when-to-use.md)
- [MCP Introduction](../06-mcp/introduction.md)
- [MCP Tools](../06-mcp/tools.md)
- [Agent SDK](../05-sdk/agent-sdk.md)
