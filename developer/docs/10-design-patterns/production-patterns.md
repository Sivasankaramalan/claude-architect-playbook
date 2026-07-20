# Production Patterns

**Production patterns** combine API usage, agents, guardrails, observability, and deployment into repeatable recipes for reliable Claude applications. Synthesis page for **Applications and Integration (33.1%)**, **Agents and Workflows (14.7%)**, and **Security and Safety (8.1%)**.

## Learning Objectives

After this page, you should be able to:

- Apply reference patterns for coding agents, support bots, and RAG assistants
- Integrate retries, streaming, caching, evals, and monitoring into each pattern
- Choose patterns appropriate to risk tier and latency requirements
- Identify anti-patterns that fail exam "production-safe" stems
- Extend patterns with MCP, hooks, and human approval where needed

## Key Ideas

### Pattern 1: Tool-using support agent

```text
Router (Haiku) → Sonnet agent + read tools + ticket API
                 → guardrails on PII output
                 → human approval for refunds
```

**Production essentials**: structured tool errors, max turns, eval golden tickets, cache static policy prefix.

### Pattern 2: Coding agent (Claude Code / SDK)

```text
Agent loop + file tools + MCP (CI, docs)
→ hooks block prod deploy
→ session resume for long tasks
→ subagent for test writing
```

**Production essentials**: sandbox, permission tiers, `CLAUDE.md` in repo, headless CI with scoped flags.

### Pattern 3: RAG Q&A

```text
Retrieve top-k → cite sources in prompt suffix
→ Sonnet answer with "only from context" rule
→ guardrail: require citation IDs
```

**Production essentials**: injection-aware delimiters, chunk size tuning, eval for hallucination rate, cache large corpus prefix.

Not: dump entire wiki every turn.

### Pattern 4: Batch document processing

```text
Message Batches → extract structured fields per doc
→ validator → warehouse
```

**Production essentials**: idempotent processing, batch error reporting, no streaming requirement.

### Pattern 5: Multi-stage research + deliverable

```text
Supervisor → parallel research workers
          → synthesis → reflection + fact tools
          → final report
```

**Production essentials**: worker summaries, cost caps, trace fan-out monitoring.

### Cross-cutting production stack

Every pattern should include:

| Layer | Pattern element |
| --- | --- |
| API | Retries, rate limits, `request_id` logging |
| Agent | `stop_reason` loop, max turns |
| Safety | Least privilege tools, hooks |
| Quality | Eval suite + schema validation |
| Performance | Model routing, caching, pruning |
| Ops | Dashboards, deploy versioning, rollback |

### Risk tiering

| Tier | Requirements |
| --- | --- |
| Low (internal draft) | Basic logging |
| Medium (customer chat) | Guardrails + evals |
| High (money, privacy) | Approval + audit + strict tools |

Match pattern complexity to tier—not every chatbot needs supervisor + reflection.

## Exam Notes

- "Production-safe refund" → **approval + validated tool**, not prompt honesty
- High QPS FAQ → **router + Haiku + cache**, not Opus agent
- Bulk overnight → **Message Batches**
- Agent loops forever → **max turns + stop_reason**, monitoring
- Injection via KB → **untrusted delimiters + tool scoping**

Always ask: **simplest pattern that satisfies stem constraints?**

## Production Notes

### Pattern selection checklist

- [ ] Risk tier documented
- [ ] Latency SLO drives streaming vs batch
- [ ] Tool write surface minimized
- [ ] Eval cases for pattern-specific failures
- [ ] Runbook linked from monitors
- [ ] Rollback tested for prompt+tool combo

### Maturity ladder

1. Single sync call + schema validation  
2. Agent loop + tools  
3. Routing + caching  
4. Subagents/supervisor  
5. Full observability + continuous eval  

Don't skip steps 1–2's guardrails when jumping to 4.

## Common Mistakes

- **Production pattern without observability**
- **RAG without injection defenses**
- **Coding agent without sandbox/hooks**
- **Supervisor for linear 2-step flows**
- **Prompt-only "production" answers on exam**

## See Also

- [Architecture](./architecture.md)
- [Deployment](../08-production/deployment.md)
- [Safety](../08-production/safety.md)
- [Optimization](../09-performance/optimization.md)
- [Checklist](../11-best-practices/checklist.md)
- [Anti-Patterns](../11-best-practices/anti-patterns.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
