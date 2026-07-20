# Monitoring

**Monitoring** tracks the health and SLOs of Claude applications in production: availability, latency, error rates, token spend, cache efficiency, and agent success rates. Unlike [Tracing](./tracing.md) (single-request forensics), monitoring answers *is the system healthy right now?*

Supports **Applications and Integration (33.1%)** and **Model Selection & Optimization (16.8%)**.

## Learning Objectives

After this page, you should be able to:

- Define SLIs and SLOs appropriate for LLM APIs and agent workflows
- Choose metrics that detect semantic degradation, not only HTTP errors
- Set alert thresholds that reduce noise while catching real incidents
- Monitor cost, cache, and rate-limit signals proactively
- Connect dashboards to runbooks and on-call response

## Key Ideas

### SLIs for Claude apps

| SLI | Definition | Why it matters |
| --- | --- | --- |
| **Availability** | % successful API calls (2xx after retries) | Infra and quota health |
| **Latency** | P50/P95 time to final response (or TTFT if streaming) | User experience |
| **Task success rate** | % sessions completing workflow without escalation | Semantic health |
| **Tool error rate** | % tool calls failing or timing out | Integration breakage |
| **Token cost per task** | Input + output tokens (and cache reads) | Budget control |
| **Guardrail trigger rate** | Blocks / validations failed | Safety and quality drift |

HTTP 200 with wrong JSON is a **logical failure**—track business-level success separately.

### Golden signals adapted for agents

Google's four golden signals still apply:

1. **Latency** — model + tools + orchestration (not API alone)
2. **Traffic** — requests, concurrent agent sessions
3. **Errors** — HTTP errors, tool exceptions, validation failures, max-turn exceeded
4. **Saturation** — rate limit proximity, queue depth, context size trends

Add **cost** as a fifth agent-specific signal.

### Dashboard layers

```text
Executive: cost/day, success rate, P95 latency
     │
Operational: errors by type, model, region, tool
     │
Debug: link to traces, recent deploys, prompt version
```

Tag metrics with **model id**, **prompt version**, **feature flag**—regressions often follow deploys.

### Rate limits and quotas

Monitor:

- 429 rate and retry success
- Tokens per minute vs tier limits
- Batch queue depth (Message Batches)

Alert **before** hard failures: sustained 429s or rising retry latency.

See [Rate Limits](../03-api/rate-limits.md) and [Retries](../03-api/retries.md).

### Cache monitoring

For prompt caching deployments:

- `cache_read_input_tokens` vs `cache_creation_input_tokens`
- Hit rate trend after prompt changes (breakpoint drift breaks cache)
- Cost savings estimate

Sudden drop in cache reads often means **prefix changed**—see [Caching](../09-performance/caching.md).

### Alerting best practices

- Page on **SLO burn** (error budget), not single failed request
- Use multi-window alerts (5 min spike vs 1 hr trend)
- Include **runbook link** in alert body
- Separate infra alerts (529 overloaded) from app alerts (tool schema mismatch)

## Exam Notes

- Production reliability → **retries, rate limit handling, monitoring**, not "switch to bigger model"
- Monitoring vs debugging: dashboards for **trends**; traces for **one session**
- Cost overrun scenario → **token metrics, caching, model routing**, not only "use shorter prompts"
- Semantic quality drop with green HTTP → **task success metric + evals**, not uptime alone

## Production Notes

### Monitoring checklist

- [ ] API latency and error rate by model and endpoint
- [ ] Tool/MCP latency and error rate by tool name
- [ ] Token usage and estimated cost per hour/day
- [ ] Cache hit/read/write metrics
- [ ] Agent turn count distribution (detect runaway loops)
- [ ] `stop_reason` breakdown (`tool_use` vs `end_turn` vs `max_tokens`)
- [ ] Deploy markers on charts for correlation

### SLO example (illustrative)

- 99.5% API availability (monthly)
- P95 end-to-end task latency < 15s for tier-1 workflow
- Task success rate > 92% on instrumented flows

Tune to your product; document error budget policy.

## Common Mistakes

- **Monitoring only Anthropic HTTP status** — missing tool layer failures
- **No cost alerts** — surprise invoice before quality issues
- **Alert fatigue** — threshold too sensitive on variable LLM latency
- **Ignoring cache metrics** — miss easy optimization wins
- **No version tags** — can't correlate prompt deploy with spike in failures

## See Also

- [Observability](./observability.md)
- [Tracing](./tracing.md)
- [Evaluations](./evaluations.md)
- [Cost Control](../09-performance/cost-control.md)
- [Rate Limits](../03-api/rate-limits.md)
