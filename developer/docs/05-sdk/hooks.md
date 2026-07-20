# Hooks

**Hooks** are deterministic lifecycle callbacks that run at defined points in the agent loop — especially **before** and **after** tool execution. They are the primary mechanism when an exam stem says behavior must be **guaranteed**, not merely suggested in a prompt.

Hooks exist in **Claude Code** (settings JSON + shell commands) and the **Agent SDK** (callback hooks + optional loading of settings files).

Public docs: [Hooks guide](https://code.claude.com/docs/en/hooks-guide), [Agent SDK hooks](https://code.claude.com/docs/en/agent-sdk/hooks).

## Learning Objectives

- Distinguish **PreToolUse** vs **PostToolUse** and when each applies
- Configure hooks in `.claude/settings.json` with matchers and command handlers
- Explain settings **precedence** (managed → local → project → user → plugin → skill frontmatter)
- Use SDK callback hooks vs shell command hooks
- Relate hooks to MCP tool normalization and policy enforcement

## Key Ideas

### Why hooks exist

| Mechanism | Enforcement | Exam weight |
| --- | --- | --- |
| System prompt / CLAUDE.md | Soft guidance | Weak for “must / guarantee” |
| `tool_choice`, schemas | Structured model output | Strong for tool *selection* |
| **Hooks** | Hard gates around tool *execution* | Strong for “block / allow / transform” |

Hooks run **outside** the model — they do not depend on Claude “remembering” a rule.

### Primary hook events (tool loop)

| Event | When | Can block? | Typical use |
| --- | --- | --- | --- |
| **PreToolUse** | Before tool executes | **Yes** | Block `rm -rf`, prod paths, unapproved MCP tools |
| **PostToolUse** | After tool succeeds | No (already ran) | Auto-format, lint, audit log, normalize MCP JSON |
| PostToolUseFailure | After tool fails | No | Error metrics, user-safe messaging |
| UserPromptSubmit | User sends prompt | Yes | PII redaction, policy injection |
| Stop | Agent finishes turn | Yes | Require checklist before ending |

**PreToolUse** can return `permissionDecision`: `allow`, `deny`, `ask`, or `defer` (SDK callback hooks). Shell hooks use **exit code 2** to block.

### Example: PostToolUse auto-format

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$CLAUDE_FILE_PATH\""
          }
        ]
      }
    ]
  }
}
```

### Example: PreToolUse guard (conceptual)

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/validate-bash-command.sh"
          }
        ]
      }
    ]
  }
}
```

Matchers support exact names (`Write`), regex (`Edit|Write`), or `*` / empty for all tools.

### Configuration hierarchy (highest wins)

1. **Managed policy settings** (enterprise)
2. `.claude/settings.local.json` (machine-local, gitignored)
3. `.claude/settings.json` (project — **commit team hooks here**)
4. `~/.claude/settings.json` (user-global)
5. Plugin `hooks/hooks.json`
6. Skill / agent **frontmatter** hooks (active only while component runs)

**Exam rule:** Team-critical MCP config and enforcement hooks belong in **project** settings — not only user-level `~/.claude/`.

### Hooks vs MCP

- **MCP server** — implements tool behavior and structured errors.
- **Hooks** — intercept *any* tool call (built-in or MCP) for policy, logging, or output normalization.

Use hooks when the same rule must apply across **multiple** tools or servers.

### Agent SDK hooks

Pass `hooks` in `query()` options for TypeScript/Python callbacks, and enable `settingSources` / `setting_sources` to load shell hooks from `.claude/settings.json`.

SDK PreToolUse can modify `updatedInput` or deny before execution — mirror Claude Code semantics.

### InstructionsLoaded

Fires when `CLAUDE.md` or `.claude/rules/*.md` loads — observability only, not blocking. Useful for auditing which instructions entered context.

## Exam Notes

- **“Guarantee no deploy to prod”** → PreToolUse hook or permission gate — not CLAUDE.md alone.
- **“Auto-run formatter after edits”** → PostToolUse with Edit|Write matcher.
- **PreToolUse blocks**; **PostToolUse cannot undo** a successful tool — design gates before side effects.
- **Project vs user config**: shared team policies → `.claude/settings.json`; personal experiments → local/user.
- MCP normalization (strip secrets from tool output) → **PostToolUse** hook, not vaguer tool description.

## Production Notes

- Keep hook scripts **fast** — they run on every matched tool call; set timeouts.
- Make PreToolUse scripts **idempotent** and safe on partial input.
- Log hook decisions with tool name, session id, and decision reason.
- Test hooks in CI with mock tool payloads before rolling out deny rules.
- Prefer structured deny messages the model can relay to users.
- Document hook contracts in the repo README so teammates know why builds fail.

## Common Mistakes

- Using PostToolUse to “prevent” destructive Bash (tool already ran)
- Storing **team** MCP servers only in `~/.claude/settings.json`
- Relying on CLAUDE.md for security boundaries hooks should enforce
- Over-broad matchers (`*`) without testing latency impact
- Forgetting SDK apps need **`settingSources`** to pick up project hooks

## See Also

- [Agent SDK](./agent-sdk.md)
- [Skills Overview](../07-skills/overview.md)
- [Memory](./memory.md)
- [MCP Security](../06-mcp/security.md)
- [MCP Best Practices](../06-mcp/best-practices.md)
- [Production Guardrails](../08-production/guardrails.md)
