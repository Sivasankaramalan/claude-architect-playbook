# Skills Structure

A Claude Code **Skill** is organized as a directory centered on **`SKILL.md`** — YAML frontmatter plus markdown instructions — optionally bundled with scripts, examples, and reference material the agent reads or executes during a matched workflow.

## Learning Objectives

- Author valid `SKILL.md` frontmatter (`name`, `description`)
- Organize supporting files (scripts, templates, references)
- Add optional skill-scoped hooks in frontmatter
- Write descriptions optimized for skill discovery/routing
- Distinguish skill contents from CLAUDE.md rules

## Key Ideas

### Minimal skill layout

```
my-skill/
├── SKILL.md          # Required — metadata + instructions
├── scripts/          # Optional — executable helpers
│   └── deploy.sh
├── references/       # Optional — docs loaded when needed
│   └── checklist.md
└── assets/           # Optional — templates, samples
    └── report-template.md
```

### SKILL.md frontmatter

Frontmatter drives **discovery** — the harness exposes description to the model before full load:

```markdown
---
name: release-notes
description: Draft release notes from git history and update CHANGELOG.
  Use when the user asks for release notes, changelog updates, or version prep.
---

# Release Notes Skill

## Steps
1. Run `scripts/collect-commits.sh` for the target range.
2. Group changes by type (feature, fix, breaking).
3. Update CHANGELOG.md using assets/report-template.md.
...
```

| Field | Role |
| --- | --- |
| `name` | Short identifier; may map to slash command patterns |
| `description` | **Critical** — when the skill should activate |

Descriptions should include **trigger phrases** and **negative scope** (“Do not use for hotfix deploys — use deploy-hotfix skill”).

### Body content guidelines

- **Procedural steps** — numbered, checkable
- **Command examples** with expected outputs
- **Edge cases** — empty diff, missing tag, failed tests
- **Links** to repo-specific paths sparingly — prefer relative references

Keep SKILL.md focused; move long reference material to `references/` files the agent reads with built-in Read tool.

### Optional hooks in skill frontmatter

Skills may declare hooks active **while the skill runs** — e.g. PostToolUse format only during a formatting skill. Scope is limited to skill lifecycle; global policy still belongs in `.claude/settings.json`.

Supported events (subset): PreToolUse, PostToolUse, Stop — verify current docs for full list.

### Skills vs `.claude/rules/`

| | Skill | Rule file |
| --- | --- | --- |
| File | `SKILL.md` in skill dir | `.claude/rules/*.md` |
| Trigger | Task match | Path/convention loading |
| Scripts | Common | Rare |
| Scope | Workflow | Ongoing standards |

### Skills vs slash commands

Slash commands are **user-invoked** shortcuts. Skills are **model-selected** (or invoked via Skill tool) based on description match. Some teams expose both for the same workflow.

## Exam Notes

- **`description` in frontmatter** — primary routing field (parallel to MCP tool descriptions).
- **Workflow + scripts** → Skill structure, not bare CLAUDE.md bullet.
- Skill hooks → **scoped** enforcement during workflow — not replacement for global PreToolUse on Bash.
- **`name`** should be stable — used in tooling and logs.
- Monolithic instructions belong split across SKILL.md + references/, not one unreadable file.

## Production Notes

- Lint skill folders in CI — required frontmatter fields present.
- Scripts: use `set -euo pipefail` in bash; avoid destructive defaults.
- Document required env vars at top of SKILL.md.
- Keep scripts cross-platform or gate with platform checks.
- Review skill descriptions when users report “skill didn’t activate.”

## Common Mistakes

- Missing or generic `description` (“helps with stuff”)
- 2,000-line SKILL.md — exhausts context when loaded
- Secrets committed in `scripts/` or templates
- Putting standing code style in Skills instead of **rules**
- Skill hooks that duplicate global hooks without clear scope — confusing debugging

## See Also

- [Skills Overview](./overview.md)
- [When to Use Skills](./when-to-use.md)
- [Skills Packaging](./packaging.md)
- [Memory (CLAUDE.md)](../05-sdk/memory.md)
- [Hooks](../05-sdk/hooks.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
