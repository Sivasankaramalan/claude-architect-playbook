# Skills Overview

**Claude Code Skills** are folders of instructions, scripts, and supporting files that teach the agent **how to perform repeatable workflows** — deploy steps, code review checklists, domain-specific procedures. Skills load **on demand** when the user’s task matches the skill’s description, unlike always-on project memory in `CLAUDE.md`.

Public docs: Claude Code Skills (see [code.claude.com](https://code.claude.com/docs) documentation index).

## Learning Objectives

- Define Skills and how they differ from CLAUDE.md, rules, and MCP
- Explain on-demand loading vs always-loaded memory
- Place Skills in the Claude Code / Agent SDK configuration ecosystem
- Map Skills to CCDV F **Claude Code (3.1%)** and related agent topics
- Know when Skills are the right abstraction vs hooks or MCP tools

## Key Ideas

### What a Skill is

A Skill is typically a directory containing:

- **`SKILL.md`** — frontmatter (name, description) + procedural instructions
- Optional **scripts**, templates, reference docs, assets
- Optional **hooks** in frontmatter (scoped to skill lifecycle)

The harness discovers skills from configured paths (e.g. `.claude/skills/`, user skills directory — verify current docs). At session start, the model sees **short descriptions**; full skill content loads when relevant.

### Skills vs other config mechanisms

| Mechanism | Activation | Best for |
| --- | --- | --- |
| **CLAUDE.md / rules** | Loaded as project memory (eager/lazy) | Standing conventions, architecture |
| **Skills** | Matched by task description | Multi-step workflows, scripts |
| **Hooks** | Deterministic lifecycle events | Block/allow/transform tool calls |
| **MCP tools** | Model `tool_use` | Live APIs, databases, external systems |
| **Slash commands** | User invokes explicitly | Shortcuts, parameterized prompts |

### Why Skills exist

- Keep **CLAUDE.md lean** — avoid 500-line monoliths
- Package **repeatable playbooks** with scripts the agent can run
- Share workflows across repos via skill folders or plugins
- Combine instructions + assets in one discoverable unit

### Skills in the Agent SDK

The Agent SDK harness includes a **Skill** built-in tool — agents can invoke packaged skills similarly to Claude Code. Skills complement MCP: procedures vs external integrations.

### Skills do not enforce alone

Skills guide the model — they are **soft** like CLAUDE.md. For guarantees (*must not deploy*, *must run tests*):

- **PreToolUse hooks**
- **CI gates**
- **MCP server validation**
- **`tool_choice` / allowlists**

Exam stems with *guarantee* → not “add a Skill” alone.

### Ecosystem placement

```
User request
    → Harness matches Skill description?
        → Load SKILL.md + assets
    → Agent loop (tools, MCP, hooks)
    → stop_reason ends turn
```

## Exam Notes

- **Repeatable workflow with scripts** → Skill (Claude Code domain).
- **Live Jira API integration** → MCP tool, not Skill alone.
- **Team coding style always applied** → CLAUDE.md / **rules**, not Skill.
- **Block dangerous Bash** → **PreToolUse hook**, not Skill.
- Skills load **on demand** — not the same as always-loaded root CLAUDE.md.

## Production Notes

- Write skill **descriptions** for accurate routing — same discipline as MCP tool descriptions.
- Keep scripts idempotent and safe; skills can trigger Bash.
- Version skills in git; review in PRs like application code.
- Avoid secrets in skill folders — use env vars.
- Test skills on representative tasks before team rollout.

## Common Mistakes

- Duplicating entire CLAUDE.md content inside every Skill
- Using Skills instead of MCP for systems that need live data
- Expecting Skills to enforce security without hooks
- Vague skill description → skill never loads when needed
- Confusing **Cursor Agent Skills** path conventions with Claude Code paths without checking docs

## See Also

- [Skills Structure](./structure.md)
- [When to Use Skills](./when-to-use.md)
- [Skills vs MCP](./skills-vs-mcp.md)
- [Skills Packaging](./packaging.md)
- [Memory (CLAUDE.md)](../05-sdk/memory.md)
- [Hooks](../05-sdk/hooks.md)
