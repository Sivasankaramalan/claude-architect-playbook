# Tracing

**Tracing** follows a single user or agent request across every hop: Messages API calls, streaming chunks, tool invocations, MCP round-trips, and validation steps. Where logs tell you *that* something failed, traces show you *where* time and errors accumulated.

This page focuses on distributed tracing patterns for Claude developer workflows. It supports **Eval, Testing, and Debugging (2.6%)** and **Agents and Workflows (14.7%)**.

## Learning Objectives

After this page, you should be able to:

- Explain parent/child span relationships in a multi-turn agent loop
- Instrument synchronous, streaming, and batch API calls consistently
- Attach Anthropic `request_id` to external trace IDs
- Identify bottlenecks: model latency vs tool latency vs orchestration overhead
- Decide when trace sampling is sufficient for production cost control

## Key Ideas

### Spans in an agent loop

Each iteration of the agent loop should produce one **parent span** (e.g., `agent.turn.3`) with children:

| Child span | Captures |
| --- | --- |
| `claude.messages.create` | Model, tokens, latency, `stop_reason`, cache stats |
| `tool.execute.{name}` | Args hash, duration, error type |
| `mcp.call.{server}.{tool}` | Transport, auth, payload size |
| `validate.output` | Schema check, guardrail result |

When `stop_reason` is `tool_use`, the trace continues after tool results are appended—not as a new unrelated trace.

### Correlation identifiers

```text
trace_id (your OTel trace)
    └── span: session.start
            └── span: turn.1
                    ├── request_id: req_abc  (from Anthropic)
                    └── span: tool.search
```

Always store Anthropic's **`request_id`** (response header) on the Messages API span. Support teams and error reports reference it.

### Streaming traces

Streaming complicates timing:

- **Time to first token (TTFT)** — span attribute from first `content_block_delta`
- **Total generation time** — span end when `message_stop` fires
- **Partial failures** — stream interrupted mid-generation; mark span status ERROR with last event type

Do not wait for full completion before starting tool spans—tools run only after the assistant message (with `tool_use` blocks) is assembled.

### OpenTelemetry-style attributes (recommended)

Use consistent attribute names across services:

- `gen_ai.system` = `anthropic`
- `gen_ai.request.model`
- `gen_ai.usage.input_tokens` / `output_tokens`
- `gen_ai.response.stop_reason`
- `claude.cache.read_tokens` / `write_tokens` (custom)
- `tool.name`, `tool.success`, `tool.duration_ms`

Semantic conventions evolve; consistency within your org matters more than exact key names.

### Batches and async work

Message Batches break the synchronous trace model:

- Parent span: `batch.submit` → child `batch.poll` → per-item spans `batch.result.{id}`
- Link batch result spans to downstream processing with `batch_request_id`

For background agents, use **async context propagation** so tool callbacks retain the same `trace_id`.

## Exam Notes

- Tracing questions often ask **where to attach instrumentation**—answer: at the SDK/client boundary and around each tool executor
- Prefer **correlation IDs across turns** over "start a new log file per message"
- Streaming stems: TTFT improvement is **streaming UX**, not a separate API—trace both phases
- If asked what links Anthropic support data to your app, cite **`request_id`**

## Production Notes

### Implementation checklist

- [ ] One trace per user-facing request (or per agent task), not per HTTP call to your backend only
- [ ] Propagate trace context into MCP server processes (W3C `traceparent` headers)
- [ ] Record `stop_reason` on every model span
- [ ] Sample traces in prod (e.g., 1–10%); keep 100% for errors
- [ ] Export traces to the same system as metrics for unified dashboards

### Debugging workflow with traces

1. User report → find `session_id` → open trace
2. Identify slowest span (model vs tool)
3. Compare `request_id` spans across retries—did retry change the outcome?
4. File regression eval from an anonymized trace export

## Common Mistakes

- **One span for the entire multi-turn conversation** — loses per-turn latency breakdown
- **Dropping trace context in tool subprocesses** — MCP servers appear as "orphan" latency
- **Ignoring cache fields in spans** — missing cost attribution when cache hits drop latency
- **Tracing only happy paths** — failed validations and guardrail blocks need spans too

## See Also

- [Observability](./observability.md)
- [Monitoring](./monitoring.md)
- [Debugging](./debugging.md)
- [SDK Tracing](../05-sdk/tracing.md)
- [Latency](../09-performance/latency.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
