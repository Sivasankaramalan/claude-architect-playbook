# Production Checklist

Actionable **pre-launch and ongoing** checklists for Claude applications. Use before production deploy, after major prompt/tool/model changes, and during exam review for "production-safe" scenario questions.

## Learning Objectives

After this page, you should be able to:

- Run a structured readiness review for Claude API and agent apps
- Verify security, reliability, performance, and quality gates
- Map checklist sections to CCDV F domains
- Adopt a minimal vs full checklist based on risk tier

## Key Ideas

### Risk tiers

| Tier | Examples | Checklist depth |
| --- | --- | --- |
| **T1 — Low** | Internal drafts, no PII | Core API + logging |
| **T2 — Medium** | Customer support read-mostly | + guardrails, evals, cache |
| **T3 — High** | Money, health, prod writes | Full checklist + approval + audit |

---

## Pre-Launch Checklist

### API & integration

- [ ] Messages API client uses official SDK with pinned version
- [ ] Retries with exponential backoff for 429/529; max attempts configured
- [ ] Rate limit handling tested under load
- [ ] `request_id` captured and logged on every call
- [ ] Streaming enabled for user-facing interactive paths (where applicable)
- [ ] Message Batches used for bulk offline work (not blocking sync API)
- [ ] Error types handled distinctly (400 vs 429 vs 500)
- [ ] Timeouts configured on HTTP client and tool calls

### Agent loop & orchestration

- [ ] `stop_reason` handled: `tool_use` → execute; `end_turn` → stop
- [ ] Every `tool_use` has matching `tool_result` before next model call
- [ ] Max agent turns enforced in orchestrator (not prompt-only)
- [ ] Max tokens appropriate; `max_tokens` truncation handled
- [ ] Tool timeouts and structured error returns on failure
- [ ] Idempotency keys on side-effect tools where needed
- [ ] Session resume vs fresh-start policy documented

### Tools & MCP

- [ ] Tool descriptions clear, non-overlapping, scoped
- [ ] Input schemas validated server-side
- [ ] Read and write tools separated; least-privilege credentials
- [ ] MCP servers authenticated; not public without controls
- [ ] Tool/MCP contract tests in CI
- [ ] Structured errors (no raw stack traces to model)

### Prompts & context

- [ ] System prompt versioned in git
- [ ] Token count for system + tools within budget
- [ ] Prompt cache breakpoint on stable prefix (if high QPS)
- [ ] Pruning/summarization policy for long sessions
- [ ] Untrusted RAG/web content delimited from system instructions
- [ ] No secrets or raw PII in prompts

### Model selection & optimization

- [ ] Default model chosen for task tier (not Opus everywhere)
- [ ] Routing rules for simple vs complex paths (if mixed traffic)
- [ ] Extended thinking disabled on high-volume paths unless required
- [ ] Cost estimate per successful task documented
- [ ] Cache read/write metrics planned

### Safety & guardrails

- [ ] High-risk actions require human approval
- [ ] Hooks or middleware block forbidden tools/commands
- [ ] Output schema validation with capped retry
- [ ] Prompt injection red-team cases run (direct + indirect)
- [ ] Fail-closed on validation/auth failures
- [ ] Privacy: log redaction and retention policy defined

### Testing & evals

- [ ] Unit tests for validators and message history builder
- [ ] Integration tests with fixtures (minimal live API smoke)
- [ ] Golden eval suite (20+ cases for critical flows)
- [ ] Eval runs on staging before prod prompt/model deploy
- [ ] Regression threshold blocks release on significant drop
- [ ] Failure-mode cases (tool error, rate limit, invalid JSON)

### Observability & monitoring

- [ ] Logs: correlation IDs, model, prompt version, `stop_reason`
- [ ] Metrics: latency P95, error rate, token cost, task success rate
- [ ] Traces on agent turns and tool executions
- [ ] Dashboards for cache hit rate and guardrail triggers
- [ ] Alerts with runbook links (not alert fatigue)
- [ ] Post-deploy synthetic smoke test

### Deployment & operations

- [ ] Separate API keys per environment
- [ ] Secrets in secret manager (not repo)
- [ ] Prompt + code + tools deployed as versioned unit
- [ ] Canary or feature flag for major agent changes
- [ ] Rollback procedure tested (including prompt revert)
- [ ] On-call knows how to pull `request_id` trace

### Claude Code (if applicable)

- [ ] `CLAUDE.md` / `.claude/rules` in repo
- [ ] Headless CI permissions scoped and timeout-bound
- [ ] Hooks for destructive command prevention
- [ ] Team config hierarchy documented

---

## Ongoing Operations Checklist (Weekly / Monthly)

### Weekly

- [ ] Review top costly sessions and failed tasks
- [ ] Check eval score trend vs prior week
- [ ] Scan guardrail trigger spikes
- [ ] Verify cache read ratio stable after deploys

### Monthly

- [ ] Rotate/review API key access
- [ ] Audit tool manifest for creep (unused tools)
- [ ] Model version currency vs Anthropic releases
- [ ] Add eval cases from production incidents
- [ ] Red team injection spot check

---

## Exam Quick Checklist (Scenario Questions)

When a stem says **production**, **guarantee**, or **must**:

1. [ ] Is there a **programmatic** option (schema, hook, validation, approval)?
2. [ ] Is **`stop_reason` / tool loop** correct?
3. [ ] Is **context** managed (prune, cache, retrieve)—not infinite history?
4. [ ] Is **model tier** appropriate (Haiku route vs Opus)?
5. [ ] Is **simplest** sufficient architecture chosen?
6. [ ] Are **retries/rate limits** mentioned for reliability?
7. [ ] Are **secrets** kept out of prompts?

## Exam Notes

- Full production checklist maps to multiple domains—exam picks **one best next step**; use quick checklist to eliminate weak distractors
- Missing evals/guardrails fails "production-safe" even if API works
- Claude Code items low weight (3.1%) but free points if you use the product

## Production Notes

Print or copy the **Pre-Launch** section into your team's release template. Not every T1 app needs every box—document waived items with risk acceptance for T3.

## Common Mistakes

- Treating checklist as **post-incident only**
- Checking boxes without automated verification (tests/evals)
- Waiving security items on "internal" tools that touch prod data
- Launch without rollback tested

## See Also

- [Do's](./dos.md)
- [Don'ts](./donts.md)
- [Security](./security.md)
- [Deployment](../08-production/deployment.md)
- [Production Patterns](../10-design-patterns/production-patterns.md)
- [Exam Blueprint](../00-getting-started/exam-blueprint.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
