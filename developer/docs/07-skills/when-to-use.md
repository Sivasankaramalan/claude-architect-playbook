# When to Use Skills

Choose **Skills** when you have a **repeatable, multi-step workflow** that benefits from bundled instructions and scripts — and the workflow should activate **only when relevant**, not on every session. Choose other mechanisms when the requirement is always-on guidance, live integrations, or hard enforcement.

## Learning Objectives

- Select Skills vs CLAUDE.md / rules / hooks / MCP for scenario stems
- Identify exam triggers mentioning workflows, scripts, and on-demand loading
- Avoid Skills for guarantees, live API data, or universal conventions
- Combine Skills with hooks and MCP in layered designs
- Recognize Claude Code domain question patterns (small but high ROI)

## Key Ideas

### Decision flowchart

```
Need live external API/data?
  YES → MCP tools (+ resources)
  NO ↓
Must behavior be guaranteed (block/allow)?
  YES → Hooks (+ server validation)
  NO ↓
Applies to every edit in repo?
  YES → CLAUDE.md / .claude/rules/
  NO ↓
Repeatable multi-step workflow with scripts/templates?
  YES → Skill
  NO ↓
Simple user shortcut?
  YES → Slash command
```

### Use Skills when

| Scenario | Why Skill |
| --- | --- |
| Release preparation checklist | Steps + scripts, occasional use |
| Security review playbook | Large procedure, load on demand |
| Migration runbook (framework X → Y) | References + ordered steps |
| Domain-specific report generation | Templates in `assets/` |
| Onboarding task “set up local env” | Commands + verification |

### Do not use Skills alone when

| Scenario | Better choice |
| --- | --- |
| Block prod database writes | PreToolUse hook + MCP validation |
| Always use 2-space indent in TS | `.claude/rules/typescript.md` |
| Query live tickets from Jira | MCP tool |
| Ensure JSON output shape | JSON Schema / structured output |
| Fetch current stock price | MCP tool or WebFetch with policy |

### Skills + MCP together

Common production pattern:

- **Skill** — procedure: “How we investigate incidents”
- **MCP tools** — `query_logs`, `create_incident`, `page_oncall`
- **Hooks** — deny `Bash` matching `kubectl delete`
- **Rules** — repo test conventions

The Skill tells the agent *how to orchestrate*; MCP performs *integrations*.

### Skills + hooks together

- Skill describes “run tests before commit”
- **Hook** or CI **guarantees** tests actually ran — Skill alone is weak for exam *guarantee* stems

### Claude Code exam scenarios (representative)

| Stem hint | Likely answer |
| --- | --- |
| “Repeatable deploy workflow in repo” | Skill |
| “Shared team conventions always” | Project CLAUDE.md / rules |
| “Prevent force-push to main” | Hook |
| “Connect to company CRM” | MCP server |
| “Resume long refactor session” | `--continue` / session (see [sessions.md](../05-sdk/sessions.md)) |
| “Headless CI code review” | Claude Code `-p` or Agent SDK |

### Config hierarchy reminder

Team artifacts → **project** files (committed). Personal experiments → user/local. Exam trap: team skill only on one developer’s laptop.

Precedence for settings/hooks: managed → local → project → user → plugin → skill frontmatter.

## Exam Notes

- **On demand** vs **always loaded** — key Skills discriminator.
- **Guarantee** → hooks/schemas — Skills are distractors.
- **External system** → MCP — not Skill-only.
- **Modular conventions by file type** → rules with `paths:` — not Skills.
- Small **Claude Code weight (3.1%)** — quick points if you know the table above.

## Production Notes

- Maintain a team index of official Skills — avoid duplicate overlapping skills.
- Deprecate skills explicitly in description when replaced.
- Measure skill activation via logs / InstructionsLoaded / tracing.
- Pair new Skills with eval tasks (“can agent complete release prep?”).
- Document in README which Skills are mandatory vs optional helpers.

## Common Mistakes

- Skill for every tiny task — discovery noise, wrong skill loaded
- Replacing MCP with a Skill that curl’s APIs manually in Bash — fragile
- Standing architecture docs copied into Skills — belongs in CLAUDE.md
- No hook/CI backup for destructive steps documented only in Skill
- Overlapping skill descriptions — same failure mode as overlapping MCP tools

## See Also

- [Skills Overview](./overview.md)
- [Skills vs MCP](./skills-vs-mcp.md)
- [Skills Structure](./structure.md)
- [Hooks](../05-sdk/hooks.md)
- [Memory](../05-sdk/memory.md)
- [Exam Blueprint](../00-getting-started/exam-blueprint.md)
