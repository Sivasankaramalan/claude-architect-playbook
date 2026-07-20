# Memory

In Claude Code and the Agent SDK harness, **memory** means **project instructions and context files** loaded into the agent — not a separate vector database unless you add one via MCP. The exam tests **CLAUDE.md**, **rules**, and how they differ from API system prompts and Skills.

## Learning Objectives

- Describe the CLAUDE.md / rules configuration hierarchy
- Contrast persistent instructions (memory files) vs per-turn API `system` prompts
- Explain lazy loading of nested CLAUDE.md and path-scoped rules
- Choose between CLAUDE.md, rules, Skills, and hooks for a given requirement
- Avoid treating “memory” as unlimited recall across sessions without explicit persistence

## Key Ideas

### What “memory” is in Claude Code

| Source | Scope | Loaded when |
| --- | --- | --- |
| `~/.claude/CLAUDE.md` | All projects (user) | Session start (user global) |
| `CLAUDE.md` or `.claude/CLAUDE.md` | Project root | Session start |
| Nested `CLAUDE.md` | Subdirectory | **Lazy** — when agent accesses that path |
| `.claude/rules/*.md` | Modular topics / paths | Eager or conditional via `paths:` frontmatter |

**Team instructions** belong in **repository** scope — committed `CLAUDE.md` / `.claude/rules/` — not only `~/.claude/`.

### Hierarchy (general precedence)

More specific / closer to the file being edited usually wins over broad global guidance:

1. Path-specific **rules** matching current files
2. **Nested CLAUDE.md** in working directory tree
3. **Project** CLAUDE.md
4. **User** `~/.claude/CLAUDE.md`

Exact merge behavior is harness-defined — treat hierarchy questions as **“most specific + project for team norms.”**

### CLAUDE.md vs `.claude/rules/`

| CLAUDE.md | Rules directory |
| --- | --- |
| Single or few markdown files | Many small focused files |
| Good for project overview | Good for conventions by topic or glob |
| Can grow monolithic | Reduces context bloat via modularity |

**InstructionsLoaded** hook fires when these files enter context — useful for auditing, not enforcement.

### Memory vs Messages API system prompt

| | CLAUDE.md / rules | API `system` parameter |
| --- | --- | --- |
| Audience | Claude Code / SDK with settings | Any Messages API client |
| Version control | Committed with repo | App config / DB |
| Enforcement | Soft (guidance) | Soft unless paired with tools/hooks |
| Exam association | Claude Code domain | Applications & Integration |

For **guaranteed** behavior, pair memory with **hooks** or **tool constraints** — memory alone is a weak exam answer.

### Memory vs Skills

| | CLAUDE.md / rules | Skills |
| --- | --- | --- |
| Activation | Always eligible when loaded | **On demand** when task matches skill description |
| Content | Conventions, architecture | Workflow scripts, templates, procedures |
| Size | Keep lean | Can include larger procedural bundles |

See [Skills Overview](../07-skills/overview.md).

### Memory vs sessions

- **Memory files** shape *defaults* every session.
- **Sessions** store *this run’s* turns and tool results.
- Changing CLAUDE.md mid-session may not reload until path triggers **InstructionsLoaded** / new session.

### External memory (MCP)

Long-term factual memory beyond files → usually **MCP resources/tools** or your app database, not CLAUDE.md alone.

## Exam Notes

- **“Team coding standards in repo”** → project `CLAUDE.md` or `.claude/rules/` — not user home only.
- **“Repeatable deploy workflow with scripts”** → **Skill**, not bloated CLAUDE.md.
- **“Block merges without tests”** → **hook** or CI — not memory file.
- Nested CLAUDE.md loads when agent **works in that subtree** — lazy loading distractor vs “always loaded.”
- Memory does **not** replace passing explicit facts to subagents.

## Production Notes

- Keep root CLAUDE.md **short** — link to rules or docs for depth.
- Split rules by concern: `testing.md`, `api-style.md`, `security.md` with optional `paths:` globs.
- Review memory files in PRs like code — they steer every agent run.
- Avoid secrets in CLAUDE.md; reference env var **names** only.
- Periodically prune outdated instructions — stale memory causes silent wrong behavior.
- For monorepos, use **nested CLAUDE.md** in packages instead of one giant root file.

## Common Mistakes

- Putting **enforceable security policy** only in CLAUDE.md
- Storing **team** conventions in `~/.claude/CLAUDE.md` only
- Confusing **Skills** (task workflows) with **rules** (standing conventions)
- Assuming Claude “remembers” prior **sessions** without resume/continue or explicit summary
- Duplicating the same guidance in CLAUDE.md, system prompt, and Skills — wastes context

## See Also

- [Sessions](./sessions.md)
- [Hooks](./hooks.md)
- [Skills Overview](../07-skills/overview.md)
- [Skills Structure](../07-skills/structure.md)
- [Context Window](../09-performance/context-window.md)
- [Pruning](../09-performance/pruning.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
