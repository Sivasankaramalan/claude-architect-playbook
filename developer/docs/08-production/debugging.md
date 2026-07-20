# Debugging

**Debugging** Claude applications means isolating whether failures come from prompts, context limits, tool/MCP contracts, orchestration logic, model choice, or infrastructure (rate limits, timeouts). Developer exam scenarios often describe a symptom and ask for the **most direct diagnostic or fix**.

Primary domain: **Eval, Testing, and Debugging (2.6%)**; cross-cuts **Applications and Integration (33.1%)** and **Agents and Workflows (14.7%)**.

## Learning Objectives

After this page, you should be able to:

- Apply a structured triage order for agent and API failures
- Use `stop_reason`, usage stats, and traces to narrow root cause
- Reproduce failures with minimized fixtures and eval cases
- Distinguish transient errors (429, 529) from logic bugs
- Debug streaming, tool-use, and context-window issues systematically

## Key Ideas

### The DEBUG triage order

When something goes wrong in production, investigate in this order:

1. **D**eterministic layer — schemas, validation, `tool_choice`, hooks, permissions
2. **E**rrors & infra — HTTP status, rate limits, timeouts, MCP connectivity
3. **B**ehavior & prompt — instructions, examples, conflicting system messages
4. **U**sage & context — token count, truncation, stale history, cache breakpoints
5. **G**eneration settings — model, temperature, max tokens, thinking mode

Skipping straight to "rewrite the system prompt" wastes time when the bug is a missing `tool_result` or 429 retry policy.

### Symptom → likely cause map

| Symptom | Check first |
| --- | --- |
| Agent stops early | `stop_reason` = `end_turn` vs `max_tokens`; prompt says "stop when done" |
| Agent loops forever | Missing termination condition; tool always returns same error; no max turns |
| Wrong tool chosen | Overlapping tool descriptions; too many tools; weak descriptions |
| Tool "succeeds" but task fails | Model hallucinates args; no server-side validation |
| JSON parse errors | Structured output schema; ask model to use tool instead of free text |
| Slow responses | Tool latency vs model latency (trace spans); oversized context |
| Inconsistent behavior | Non-deterministic temperature; changing prompts; cache prefix drift |
| "Forgot" earlier facts | Context pruning; subagent without injected summary; session not resumed |

### Minimal reproduction

Shrink the failure until it fits one API call or one agent turn:

1. Export conversation history (redacted)
2. Remove unrelated tools and messages
3. Fix random seeds where applicable; use low temperature for repro
4. Save as eval case (see [Evaluations](./evaluations.md))

If the bug disappears when context shrinks, suspect **context window** or **attention** issues—see [Context Window](../09-performance/context-window.md).

### API-level debugging tools

| Signal | Use |
| --- | --- |
| `request_id` | Support correlation, compare retries |
| `stop_reason` | Loop control diagnosis |
| `usage` | Unexpected token spikes |
| Streaming events | Partial output, mid-stream errors |
| Error type (`rate_limit_error`, etc.) | Retry vs fix client |

Log **request and response shapes**, not necessarily full text, in staging.

### Agent loop debugging

Walk the loop on paper:

```text
messages in → API call → stop_reason?
    │
    ├─ tool_use → execute tools → append tool_result → loop
    └─ end_turn → validate output → done or retry policy
```

Common bug: **tool_result not appended** or appended with wrong `tool_use_id`.

Another: continuing the loop after `end_turn` because the client ignores `stop_reason`.

### Claude Code / SDK debugging

- **Hooks** — log or block tool calls pre-execution
- **Session resume** — stale state vs fresh session with summary
- **Permissions** — tool blocked silently vs explicit error
- Compare CLI flags and `CLAUDE.md` hierarchy when behavior differs local vs CI

## Exam Notes

- Stems with "agent keeps calling the same tool" → check **tool error handling**, **descriptions**, or **max turn** limits—not immediately "switch to Opus"
- "Production debugging" → **structured logs + request_id + traces**, not "ask the user to try again"
- Context "forgetting" → **prune/summarize**, **inject facts**, or **subagent isolation**—not unlimited history
- Rate limit errors → **exponential backoff**, concurrency limits—see [Retries](../03-api/retries.md)
- Prefer **simplest fix** that addresses root cause (exam golden rule)

## Production Notes

### Debug playbook checklist

- [ ] Capture `request_id`, model, prompt version hash, tool manifest version
- [ ] Replay in staging with same model class (not always Opus—match prod)
- [ ] Compare token usage before/after regression
- [ ] Verify tool/MCP contract unchanged (schema drift breaks silently)
- [ ] Add regression eval before closing incident

### Safe production debugging

- Never paste production secrets into public playgrounds
- Use synthetic data in shared threads
- Toggle verbose logging via feature flag, not permanent INFO dumps

## Common Mistakes

- **Changing model first** — masks prompt/tool bugs; expensive
- **Ignoring `stop_reason`** — misdiagnosing loop issues
- **Debugging without version pins** — prompt and tool definitions drift
- **Assuming tool use implies correct data** — validate server-side
- **Infinite retries on 400-class errors** — client bug, not transient

## See Also

- [Observability](./observability.md)
- [Tracing](./tracing.md)
- [Testing](./testing.md)
- [Evaluations](./evaluations.md)
- [Prompt Debugging](../02-prompt-engineering/prompt-debugging.md)
- [Error Handling](../03-api/error-handling.md)
- [Tool Results](../04-agent-engineering/tool-results.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
