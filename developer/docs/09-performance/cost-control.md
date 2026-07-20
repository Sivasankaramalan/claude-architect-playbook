# Cost Control

**Cost control** manages Anthropic API spend: input/output tokens, cache reads/writes, model tier, agent turn count, tool fan-out, and batch vs online pricing. **Model Selection and Optimization (16.8%)** heavily weights cost-aware decisions alongside quality and latency.

## Learning Objectives

After this page, you should be able to:

- Break down a Claude bill into model, tokens, cache, and agent-loop drivers
- Apply practical caps: budgets, rate limits, model routing, pruning, caching
- Estimate cost per successful task—not per API call alone
- Design alerts and quotas for multi-tenant or team usage
- Choose exam answers that reduce cost without breaking production requirements

## Key Ideas

### Token economics

Billing is primarily **token-based**:

- **Input tokens** — everything you send (system, tools, history, documents)
- **Output tokens** — model generation (including thinking tokens where applicable)
- **Cache write/read** — separate pricing; reads cheaper than full input reprocessing

Output tokens for long JSON prose are expensive—**structured tools** often reduce output size.

See [Token Pricing](../01-foundations/token-pricing.md) and [Pricing](../03-api/pricing.md).

### Top cost drivers in agent apps

| Driver | Mitigation |
| --- | --- |
| Large tool schemas every turn | Cache prefix; reduce tools |
| Verbose tool results in history | [Pruning](./pruning.md) |
| Opus on all steps | [Routing](../10-design-patterns/routing.md) to Haiku/Sonnet |
| Runaway agent loops | Max turns; clear stop conditions |
| Repeated failed schema retries | Guardrails + fix prompts once |
| Huge RAG chunks | Top-k retrieval |
| No cache on static docs | [Caching](./caching.md) |

### Cost per successful task

Executives care about **$/resolved ticket**, not $/request.

Measure:

```text
cost_per_success = total_session_tokens_cost / completed_tasks
```

Optimize loops that burn tokens without finishing.

### Budget controls

- **Per-user/tenant quotas** — daily token caps
- **Concurrency limits** — prevent parallel agent storms
- **Model allowlists** — block Opus in dev
- **Spend alerts** at 50/80/100% of monthly budget
- **Pre-call token estimate** — reject oversize attachments early

### Message Batches

Deferrable work at scale:

- Different latency SLA
- Often favorable for bulk offline generation
- Not for interactive chat

### Prompt caching ROI

Calculate:

- Write cost on first hit + deploys
- Read savings × expected repeated calls

Break-even quickly on high-QPS agents with stable system+tools prefix.

### Multi-tenant fairness

Noisy neighbor one tenant running huge agents → rate limit per API key or workspace; chargeback metrics by `tenant_id` tag.

## Exam Notes

- Highest spend agent with static system prompt → **prompt caching** + **prune history**
- Classification before heavy agent → **cheap model route** first
- Overnight 10k summaries → **Message Batches**, not synchronous loop
- Cost spike after prompt change → **cache invalidation** (writes) + longer input
- Cannot sacrifice correctness for cost when stem says **accuracy required** → optimize turns/tools, not skip validation

## Production Notes

### Cost control checklist

- [ ] Dashboard: daily spend by model, feature, tenant
- [ ] Token usage logged per session with outcome flag
- [ ] Max turns and max tokens per session enforced in code
- [ ] Cache read/write metrics
- [ ] Monthly budget alerts
- [ ] Eval gate prevents "cheap but wrong" deploys
- [ ] Review top 10 expensive sessions weekly

### Developer hygiene

- Don't embed large fixtures in prompts—fetch via tools
- Use streaming for UX, not extra calls
- Turn off extended thinking in high-volume paths
- Version and review prompt diffs for token impact (CI token count)

## Common Mistakes

- **Watching only output tokens** — input dominates agent apps
- **Haiku everywhere** — rework loops cost more than one Sonnet call
- **No max agent turns** — infinite tool/model cycles
- **Caching dynamic prefixes** — pay writes, no reads
- **Ignoring failed task cost** — retries double spend

## See Also

- [Optimization](./optimization.md)
- [Caching](./caching.md)
- [Latency](./latency.md)
- [Monitoring](../08-production/monitoring.md)
- [Rate Limits](../03-api/rate-limits.md)
