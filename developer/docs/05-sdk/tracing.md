# Tracing

**Tracing** and **observability** for Claude agents mean capturing model turns, tool calls, latencies, errors, and session metadata so you can debug failures and measure production quality. The Agent SDK and Claude Code emit structured events; your app adds correlation ids and export to your stack.

## Learning Objectives

- Identify what to log in an agent loop (prompts, tools, stop_reason, latency, session id)
- Distinguish SDK/stream tracing from Messages API request logging
- Connect tracing to hooks (PostToolUse audit) and MCP structured errors
- Relate tracing to eval and regression testing workflows
- Choose metrics that matter for agent systems (tool success rate, turns to completion)

## Key Ideas

### What to trace in agent systems

| Signal | Why it matters |
| --- | --- |
| **session_id** | Resume, support, cross-service correlation |
| **turn index / step** | Find where loop diverged |
| **`stop_reason`** | Expected tool loop vs premature end |
| **Tool name + inputs** (redacted) | Reproduce bad tool selection |
| **Tool results / errors** | MCP vs built-in failure modes |
| **Model id + token usage** | Cost and latency optimization |
| **Latency per turn** | SLA and streaming UX |
| **Hook decisions** | Policy denials vs model choices |

### Agent SDK tracing patterns

`query()` returns an **async stream** of messages — treat each message as a trace event:

```python
async for message in query(prompt=task, options=options):
    log_agent_event(message)  # type, tool_use, result, session metadata
```

Production apps typically:

1. Wrap `query()` in a span (OpenTelemetry or vendor APM).
2. Emit **structured JSON** logs (not printf debugging).
3. Attach **user / tenant / job id** separate from session id.

Public SDK docs cover streaming message shapes — log the fields your SRE team needs, not full raw prompts if PII-heavy.

### Hooks as tracing hooks

**PostToolUse** is ideal for **audit trails** — append-only log of file edits, MCP calls, or denials.

**InstructionsLoaded** audits which CLAUDE.md / rules entered context — useful when behavior “changed mysteriously.”

Tracing complements hooks: hooks enforce or normalize; tracing **records**.

### MCP and tracing

Well-designed MCP tools return **structured errors** (category, retryable, user message). Log:

- Server name + tool name
- Error category (auth, validation, upstream)
- Duration

Avoid logging full stack traces to users; include trace id in user message for support.

See [MCP Tools](../06-mcp/tools.md) and [Production Tracing](../08-production/tracing.md).

### Messages API apps

If you own the loop manually, trace each `messages.create()`:

- Request id from Anthropic headers
- Input/output token counts
- Tool use blocks and your executor’s results

Same conceptual model as SDK — different instrumentation point.

### Tracing vs evaluations

| Tracing | Evaluations |
| --- | --- |
| Production / staging telemetry | Offline scoring of quality |
| Debug single failed run | Regression across golden tasks |
| Continuous | CI or scheduled batches |

Trace logs often **feed** eval datasets (anonymized failure cases).

## Exam Notes

- **“Debug why agent stopped early”** → check **`stop_reason`**, not natural-language “done” phrases.
- **“Audit all file changes”** → PostToolUse hook or equivalent logging — not system prompt.
- **“MCP call failed intermittently”** → structured error category + retry policy — not raw stack to model.
- Tracing does **not** fix bad tool descriptions — it reveals wrong tool *selection*.
- Independent review → fresh context session — tracing helps find which turn to replay.

## Production Notes

- **Redact** secrets and PII in tool inputs/outputs before log storage.
- Sample high-volume traces; always retain **error** spans at 100%.
- Alert on: spike in tool failures, deny hooks, `max_tokens` stops, rate limits.
- Propagate **W3C trace context** from HTTP gateway into agent worker.
- Store traces with **retention policy** aligned with compliance (GDPR, SOC2).
- Build dashboards: median turns-to-success, cost per successful task, top failing tools.

## Common Mistakes

- Logging entire repo file contents on every Read tool call
- Parsing assistant prose for completion instead of **`stop_reason`**
- No session id — cannot correlate multi-step production failures
- Treating tracing as a substitute for **evals** on quality regressions
- Exposing internal MCP stack traces to end users

## See Also

- [Agent SDK](./agent-sdk.md)
- [Hooks](./hooks.md)
- [Sessions](./sessions.md)
- [Production Tracing](../08-production/tracing.md)
- [Observability](../08-production/observability.md)
- [Evaluations](../08-production/evaluations.md)
- [Debugging](../08-production/debugging.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
