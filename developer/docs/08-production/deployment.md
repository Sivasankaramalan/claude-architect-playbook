# Deployment

**Deployment** for Claude applications means shipping prompts, tool definitions, MCP servers, orchestration code, and configuration changes safely—with versioning, rollback, and environment separation. Developer exam scenarios often ask for the **most reliable production** approach.

Maps to **Applications and Integration (33.1%)** and touches **Security and Safety (8.1%)**.

## Learning Objectives

After this page, you should be able to:

- Structure dev/staging/prod environments for API keys, tools, and MCP endpoints
- Version prompts, tool manifests, and agent configs alongside application code
- Plan rollout strategies: blue/green, canary, feature flags
- Handle secrets, config hierarchy, and Claude Code deployment patterns
- Define rollback triggers using monitoring and eval signals

## Key Ideas

### What gets deployed

A Claude app release is more than binary code:

| Artifact | Version with |
| --- | --- |
| Application code | Git tag / container image |
| System prompts & templates | Hash in config or repo |
| Tool definitions | Same release as code consuming them |
| MCP servers | Independent service version + contract |
| Model default | Config per environment |
| Eval golden set | Prompt version compatibility |

Drift between prompt v2 and tool schema v1 causes production incidents.

### Environment separation

```text
Development  → dev API keys, mock tools, verbose logging
Staging      → prod-like tools, synthetic data, full eval suite
Production   → least privilege keys, redacted logs, canary
```

- Separate Anthropic **workspaces/keys** per environment where possible
- Staging must hit **same model classes** as prod for faithful behavior
- Never point staging at production databases with real PII

### Configuration hierarchy (Claude Code alignment)

For Claude Code and agent projects, config layers stack:

1. Enterprise / org policy (if applicable)
2. Project `CLAUDE.md` and `.claude/rules`
3. User-local overrides
4. CLI flags per invocation

Document which layer owns production-critical rules (often repo-checked `.claude/rules`, not local-only files).

### Rollout strategies

| Strategy | Use when |
| --- | --- |
| **Big bang** | Low-risk prompt typo fix |
| **Canary** | New agent workflow; route 5% traffic |
| **Blue/green** | Orchestrator service swap |
| **Feature flag** | Prompt/model/tool experiments |

Pair canary with [Monitoring](./monitoring.md): task success, latency, cost, guardrail rate.

### Message Batches for bulk offline work

Deploying batch pipelines differs from online API:

- Submit jobs asynchronously; poll completion
- Idempotent result processing (retries duplicate items)
- Separate SLA from interactive path

See [Messages API](../03-api/messages-api.md) batch documentation patterns.

### Secrets and CI/CD

- API keys in secret manager; injected at runtime
- CI uses scoped keys with spend limits
- Scan repos for leaked keys; rotate on exposure
- MCP server credentials rotated independently

### Rollback plan

Trigger rollback when:

- Error rate or guardrail spikes beyond threshold
- Eval canary subset fails
- P95 latency doubles with no traffic increase

Rollback **prompt + code + tools** together if they were released as a unit—partial rollback can desync schemas.

## Exam Notes

- Production reliability → **retries, idempotency, staging validation**, not skipping tests for speed
- Config change in team repo → **version-controlled `CLAUDE.md` / rules**, not oral tradition
- Bulk deferred processing → **Message Batches**, not synchronous Messages API at scale
- Safe deploy of agent with new tools → **staging eval + canary**, not immediate 100% traffic
- Secrets in exam → **environment variables / secret manager**, never committed prompts

## Production Notes

### Deployment checklist

- [ ] Prompt and tool versions tagged in release notes
- [ ] Staging eval pass required for merge/release gate
- [ ] Database migrations independent of prompt deploy order documented
- [ ] MCP server backward compatible or dual-run during migration
- [ ] Feature flags for model and prompt switches
- [ ] Automated rollback or one-click revert tested
- [ ] Post-deploy synthetic smoke + 15-minute enhanced monitoring

### Headless / CI agents

Claude Code headless (`-p`) in CI needs:

- Explicit permission flags for allowed tools
- Non-interactive approval for scoped operations
- Timeout and max-turn limits

Treat CI agents like production code execution—with sandboxing.

## Common Mistakes

- **Editing production prompts without version control**
- **Deploying tool schema change before backend supports it**
- **Same API key everywhere** — blast radius on leak or rate limit
- **No canary for agent workflow changes** — loops or costs explode silently
- **Rollback code but not prompt** — behavior unchanged or broken

## See Also

- [Monitoring](./monitoring.md)
- [Testing](./testing.md)
- [Evaluations](./evaluations.md)
- [Safety](./safety.md)
- [Production Patterns](../10-design-patterns/production-patterns.md)
- [Migration](../11-best-practices/migration.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
