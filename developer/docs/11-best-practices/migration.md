# Migration

**Migration** covers moving Claude applications safely across models, prompts, SDK versions, MCP contracts, and deployment topologies—without silent regressions. Relevant to **Applications and Integration (33.1%)**, **Model Selection (16.8%)**, and operational maturity.

## Learning Objectives

After this page, you should be able to:

- Plan model upgrades (e.g., Sonnet version bumps) with eval gates
- Migrate prompts, tools, and backends in compatible order
- Move from prototype (chat-only) to production agent architecture incrementally
- Handle breaking MCP or API changes with dual-run and rollback
- Answer exam scenarios about session resume vs fresh start after migration

## Key Ideas

### Model migration

When Anthropic releases new model snapshots:

1. **Baseline eval score** on current model
2. Run same suite on candidate model in staging
3. Compare pass rate, cost, latency, guardrail triggers
4. Canary in production (5–10% traffic)
5. Full cutover with rollback flag

Watch for: tool calling behavior changes, JSON strictness, refusal patterns.

Prompt caching may reset—expect temporary write costs.

### Prompt migration

- Version prompts (`prompt_v3.yaml`) alongside git tags
- Diff token count and cache breakpoint placement
- Never change prod prompt without eval delta review
- Keep **rollback prompt** one flag away

### Tool and MCP migration

```text
Phase 1: Add v2 tool alongside v1 (adapter)
Phase 2: Route agent to v2 via feature flag
Phase 3: Deprecate v1 after eval stable
Phase 4: Remove v1 from manifest
```

Breaking JSON shape changes need **adapter layer** so model isn't confused mid-migration.

### Prototype → production path

| Stage | Characteristics | Migration action |
| --- | --- | --- |
| Playground | Manual chat, no tools | Add tools + validation |
| Script | Single API calls | Add retries, logging |
| Agent v1 | Loop without caps | Max turns, evals, guardrails |
| Agent v2 | + MCP + hooks | Security review, contract tests |
| Scale | + cache, routing, batch | Cost controls, monitoring |

Don't jump from playground to supervisor multi-agent.

### Session and context migration

When changing prompt or tool manifest mid-session:

- **Resume** if change is backward compatible and context still valid
- **Fresh session + summary** if tool definitions changed materially or poisoned context
- Inform user when prior context discarded

Exam: stale tool results after deploy → **fresh session with injected summary**, not silent resume.

### SDK / Claude Code migration

- Pin SDK versions in CI
- Read changelog for hook/session behavior changes
- Migrate `CLAUDE.md` rules incrementally; test headless CI paths
- Update team docs when config hierarchy adds new layer

### Data and eval migration

When moving eval datasets:

- Tag cases with `prompt_version` compatibility
- Retire cases that no longer apply; add new failure-derived cases
- Keep historical scores for trend—not comparable across incompatible rubrics

## Exam Notes

- New model in prod safely → **staging eval + canary**, not immediate 100% switch
- Tool schema change broke agent → **version tools / fresh session**, not "prompt harder"
- Bulk processing migration → consider **Message Batches** when leaving sync API
- Cache metrics dropped after deploy → **expected** until reads stabilize; verify prefix unchanged

## Production Notes

### Migration runbook template

- [ ] Scope: model / prompt / tool / infra
- [ ] Rollback trigger metrics defined
- [ ] Eval pass threshold for go/no-go
- [ ] Communication plan for session resets
- [ ] Post-migration 24h enhanced monitoring
- [ ] Document lessons in eval case backlog

## Common Mistakes

- **Big bang model swap** on Black Friday
- **Changing tool + prompt + model simultaneously** — can't attribute regressions
- **No adapter for MCP breaking changes**
- **Assuming resume works after tool manifest overhaul**
- **Deleting old prompt version** before rollback window ends

## See Also

- [Deployment](../08-production/deployment.md)
- [Evaluations](../08-production/evaluations.md)
- [Testing](../08-production/testing.md)
- [Optimization](../09-performance/optimization.md)
- [Checklist](./checklist.md)
