# MCP Prompts

**Prompts** in MCP are **reusable templates** registered by a server — parameterized workflow starters the **user or host** selects before inference. They standardize how to combine tools and resources; they do **not** programmatically enforce outcomes.

Public reference: MCP `prompts/list` and `prompts/get` in the [architecture docs](https://modelcontextprotocol.io/docs/learn/architecture).

## Learning Objectives

- Define MCP prompts vs system prompts vs Skills
- Explain user-controlled vs model-controlled primitives
- Use `prompts/list` and `prompts/get` in workflow design
- Recognize prompts as guidance, not guarantees
- Pair prompts with tools/resources for complete workflows

## Key Ideas

### Control model (third primitive)

| Primitive | Initiator | Enforces behavior? |
| --- | --- | --- |
| Tools | Model | No — executes if called |
| Resources | Application | No — context only |
| **Prompts** | **User / host UI** | **No** — template only |

Prompts help users start consistent tasks: “Summarize this ticket,” “Triage new bugs,” “Draft release notes from changelog resource.”

### What a prompt contains

A prompt definition typically includes:

- **Name** — stable id for UI / CLI
- **Description** — when to offer this workflow
- **Arguments** — user-supplied parameters (ticket id, date range)
- **Messages** — rendered template fragments (user/assistant roles)

Host calls `prompts/get` with arguments → injects resulting messages into the session.

### Prompts vs Claude system prompt

| | MCP prompt | API / agent `systemPrompt` |
| --- | --- | --- |
| Source | External MCP server | Your app / harness |
| Discovery | `prompts/list` | Static config |
| Portability | Across MCP hosts | App-specific |
| Enforcement | Soft | Soft (unless + hooks) |

### Prompts vs MCP tools

- **Prompt** — shapes *how* the conversation starts
- **Tool** — performs *actions* during the agent loop

Exam trap: “Ensure deploy script never runs on prod” → **hook / server validation**, not MCP prompt.

### Prompts vs Skills (Claude Code)

| | MCP prompts | Claude Code Skills |
| --- | --- | --- |
| Ecosystem | Any MCP host | Claude Code / Agent SDK |
| Content | Message templates | SKILL.md + scripts + assets |
| Activation | User picks prompt | Model matches skill description |

Skills overlap conceptually but live in Claude’s harness; MCP prompts are protocol-standard.

### Combining primitives

Strong workflow design:

1. User selects MCP **prompt** “Investigate incident”
2. Host attaches **resources** (runbook, recent logs read-only)
3. Model uses **tools** (query metrics, create ticket) during loop
4. **Hooks** block dangerous tools regardless of prompt text

## Exam Notes

- Prompts are **user-controlled** — know the three-way split (tools/model, resources/app, prompts/user).
- **Reusable template** without side effects → prompt candidate.
- Prompts **do not** replace PreToolUse hooks for guarantees.
- `prompts/get` supplies messages — not the same as `tools/call`.
- Don’t pick MCP prompts when question asks for **Claude Code Skill** in repo workflows.

## Production Notes

- Version prompt templates; breaking argument names confuse UIs.
- Localize descriptions if prompts surface in end-user menus.
- Keep prompts focused — link to resources for long reference text.
- Test rendered messages for token size before shipping.
- Document which tools each prompt expects to be available.

## Common Mistakes

- Expecting MCP prompts to enforce security policy alone
- Duplicating the same template as prompt and giant system prompt
- Prompts with hidden instructions to bypass hooks (anti-pattern; blocked in mature setups)
- Using prompts where a **tool** is needed to fetch dynamic required data
- Confusing MCP prompts with **prompt caching** breakpoints

## See Also

- [MCP Tools](./tools.md)
- [MCP Resources](./resources.md)
- [Skills Overview](../07-skills/overview.md)
- [Skills vs MCP](../07-skills/skills-vs-mcp.md)
- [Prompting](../02-prompt-engineering/prompting.md)
