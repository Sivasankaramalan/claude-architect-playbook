# Observability

Production Claude applications are non-deterministic distributed systems: each request may invoke tools, MCP servers, retries, and multi-turn agent loops. **Observability** is the practice of instrumenting those paths so you can answer *what happened*, *why it failed*, and *what it cost*—without reproducing the exact user session by hand.

This page covers the signals, data model, and developer workflow for observable agent systems. It maps primarily to **Eval, Testing, and Debugging (2.6%)** and supports **Applications and Integration (33.1%)** production scenarios.

## Learning Objectives

After this page, you should be able to:

- Define the three pillars (logs, metrics, traces) in the context of Claude API and agent loops
- Identify which events to capture at each layer: API, SDK, tool, MCP, and orchestration
- Correlate a user-facing failure to `request_id`, session ID, and tool call spans
- Choose observability depth appropriate to dev, staging, and production
- Explain why prompt/response logging requires privacy and retention policies

## Key Ideas

### The three pillars for LLM apps

| Pillar | What to capture | Example questions |
| --- | --- | --- |
| **Logs** | Discrete events with context | Which tool failed? What was the error payload? |
| **Metrics** | Aggregated counters and histograms | P95 latency? Cache hit rate? Tokens per session? |
| **Traces** | End-to-end request spans | Where did the 12-second delay occur—model or DB tool? |

Unlike traditional CRUD APIs, LLM apps add **semantic events**: model id, input/output tokens, `stop_reason`, tool names, cache read/write tokens, and retry count.

### Minimum viable event schema

For each **turn** in a conversation or agent loop, log:

1. **Correlation IDs** — `request_id` (from Anthropic response headers), your `session_id`, `user_id` (hashed if needed)
2. **Request metadata** — model, `max_tokens`, streaming flag, cache breakpoints
3. **Response metadata** — `stop_reason`, usage block, latency, HTTP status
4. **Tool layer** — tool name, arguments (redacted), execution time, success/failure, structured error code
5. **Business outcome** — task completed? human escalation? fallback path?

Store **hashes or truncated previews** of prompts in production; full payloads belong in secure, short-TTL debug stores.

### Layered instrumentation

```text
User request
    │
    ▼
Orchestrator / agent loop  ← span: agent.run
    │
    ├── Messages API call   ← span: claude.messages.create
    │       └── streaming chunks (optional child spans)
    │
    ├── Tool execution      ← span: tool.{name}
    │       └── MCP / HTTP / DB
    │
    └── Post-processing     ← span: validate_output
```

Instrument **outside** the model: your code owns tool timing, validation, and routing decisions. The API gives you usage and `request_id`; you supply the rest.

### What Anthropic gives you vs what you build

| Source | Built-in signal |
| --- | --- |
| Messages API response | `usage`, `stop_reason`, `request_id` header |
| Streaming | Event types (`message_start`, `content_block_delta`, etc.) |
| Message Batches | Batch status, per-result errors |
| Your application | Tool spans, session state, eval scores, guardrail triggers |

Anthropic does not replace your APM—you wire OpenTelemetry, Datadog, Langfuse, or similar at the SDK boundary.

### Observability-driven development

1. **Design eval cases** from production failure logs (see [Evaluations](./evaluations.md))
2. **Alert on SLOs**, not on individual bad completions—e.g., tool error rate > 2%, P99 latency > 8s
3. **Sample traces** for human review; don't review 100% of traffic at scale

## Exam Notes

- **Eval/Testing domain (2.6%)**: expect questions on *what to log* or *how to diagnose* agent failures—not vendor-specific dashboard names
- When a stem asks how to **investigate production issues**, prefer: structured logs + correlation IDs + replayable test cases over "read the prompt again"
- Distinguish **monitoring** (health, SLOs) from **tracing** (single-request drill-down)—see [Monitoring](./monitoring.md) and [Tracing](./tracing.md)
- Privacy-aware logging beats verbose logging: redact secrets and PII before choosing "log everything"

## Production Notes

### Recommended production checklist

- [ ] Propagate `request_id` from API responses into all tool and MCP logs
- [ ] Track token usage and cache read/write per session for cost attribution
- [ ] Log `stop_reason` every turn—unexpected `end_turn` mid-task often indicates context or prompt issues
- [ ] Separate **debug** log level (full tool args) from **prod** (redacted)
- [ ] Define retention: 7–30 days for operational logs; longer only for aggregated metrics
- [ ] Connect observability to on-call runbooks (see [Debugging](./debugging.md))

### Anti-patterns in production

- Logging full system prompts containing secrets or customer data
- No correlation between API calls and tool executions in the same agent loop
- Measuring only HTTP 200 without tracking semantic failures (wrong JSON shape, hallucinated IDs)

## Common Mistakes

- **Treating the model response as the only artifact** — tool results and validation outcomes matter equally
- **Confusing metrics with evals** — high uptime does not mean correct answers; pair with [Evaluations](./evaluations.md)
- **Omitting `stop_reason` from dashboards** — loops that never reach `end_turn` indicate stuck or misconfigured agents
- **Over-instrumenting prompts in prod** — compliance risk; use sampling and redaction

## See Also

- [Tracing](./tracing.md)
- [Monitoring](./monitoring.md)
- [Debugging](./debugging.md)
- [Evaluations](./evaluations.md)
- [SDK Tracing](../05-sdk/tracing.md)
- [Cost Control](../09-performance/cost-control.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
