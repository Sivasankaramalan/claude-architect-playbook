# Skills Packaging

**Packaging** Skills means organizing, sharing, and versioning skill directories so teams and the Claude harness can discover them reliably — in a single repo, across monorepos, via plugins, or personal skill libraries.

## Learning Objectives

- Place skills in standard project and user paths
- Package skills for team sharing (git, plugins)
- Version and document skill dependencies and env requirements
- Avoid path and naming collisions across skill catalogs
- Relate packaging to config hierarchy (project vs user)

## Key Ideas

### Where skills live (typical layouts)

| Location | Scope | Shareable |
| --- | --- | --- |
| `.claude/skills/<skill-name>/` | Project | Yes — commit to repo |
| `~/.claude/skills/` | User-global | Personal machine |
| Plugin bundle | When plugin enabled | Via plugin distribution |
| Agent SDK `settingSources` | Loaded with project settings | Same as project |

Exact paths evolve — check current Claude Code docs. **Team skills belong in the repository.**

### Package contents checklist

Each skill directory should include:

- [ ] **`SKILL.md`** with `name` + rich `description`
- [ ] **README** (optional) for humans — install, env vars, owners
- [ ] **Scripts** with executable bit or explicit interpreter line
- [ ] **References/** for long docs not inlined in SKILL.md
- [ ] **Tests or sample inputs** where feasible (golden outputs)
- [ ] **No secrets** — env var names documented only

### Naming conventions

- Directory name: lowercase, hyphenated (`release-notes`, `security-review`)
- `name` in frontmatter: align with directory for predictable logs
- Avoid generic names (`helper`, `utils`) — weak routing

### Monorepo patterns

```
repo/
├── CLAUDE.md
├── .claude/
│   ├── settings.json
│   ├── rules/
│   └── skills/
│       ├── api-migration/
│       └── service-deploy/
├── services/
│   └── billing/
│       └── CLAUDE.md          # nested memory, not skill
```

- **Repo-wide skills** at `.claude/skills/`
- **Package-specific memory** in nested `CLAUDE.md` — not always a separate skill
- Split skills when workflows differ materially between services

### Plugins as packaging

Plugins can bundle:

- Skills (folders + SKILL.md)
- Hooks (`hooks/hooks.json`)
- MCP server definitions

Use plugins when distributing **across many repos** — internal marketplace or git submodule.

### Versioning and change management

- Semver in skill README or frontmatter `version:` field (if supported)
- Breaking script interface → update description + changelog
- PR review checklist: description updated?, scripts safe?, hooks scoped?

### Dependencies

Document:

- Required CLI tools (`gh`, `kubectl`, `terraform`)
- Env vars (`JIRA_TOKEN` — loaded via host env, not skill file)
- MCP servers the skill expects (`issue-tracker` server must be configured)

Fail fast in scripts when dependencies missing — clear error messages for the agent.

### Config hierarchy interaction

Project skills + project `.claude/settings.json` travel together in git.

**Do not** rely on user-only skill copies for team mandatory workflows — new hires won’t have them.

Managed policy may restrict which skills or hooks run in enterprise hosts.

## Exam Notes

- **Team workflow in repo** → package under **project** `.claude/skills/` — not user home only.
- **Plugin** distractor when stem says “share across all company repos.”
- Skill packaging ≠ MCP server packaging — different distribution models.
- Colliding skill `name`s → unpredictable routing — fix naming.
- Secrets in committed skill scripts → always wrong answer.

## Production Notes

- CI: scan skill scripts for secret patterns.
- Sign or pin plugin versions for supply-chain safety.
- Telemetry: which skills run in production agent jobs.
- Deprecation: prefix description `DEPRECATED: use xyz-skill`.
- Keep skill folders small — large assets via LFS or external docs + MCP resources.

## Common Mistakes

- Skills only on one developer’s `~/.claude/skills/` for team process
- Duplicate skills in plugin **and** repo with diverging scripts
- SKILL.md references absolute paths (`/Users/me/...`)
- No dependency docs — agent runs failing scripts repeatedly
- Packaging coding standards as skills instead of **rules** — causes redundant loads

## See Also

- [Skills Structure](./structure.md)
- [Skills Overview](./overview.md)
- [When to Use Skills](./when-to-use.md)
- [Memory](../05-sdk/memory.md)
- [Hooks](../05-sdk/hooks.md)
- [MCP Servers](../06-mcp/servers.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
