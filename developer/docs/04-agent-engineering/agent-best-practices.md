# Agent Best Practices

Production Claude agents combine API mechanics with engineering discipline: safe tools, bounded loops, observability, validation, and human gates. This page consolidates **developer exam favorites** across Agents and Applications domains.

**Domain focus:** Agents and Workflows (15%) + Applications and Integration (33%) + Security (8%).

## Learning Objectives

After this page you should be able to:

- Apply the core agent checklist: `stop_reason`, history, tool pairing, `is_error`, validation.
- Enforce safety with least privilege, approvals, and schema + semantic checks.
- Bound cost and reliability (iterations, timeouts, retries, rate limits).
- Choose programmatic controls (`tool_choice`, hooks) over prompt-only guarantees.
- Avoid high-frequency exam traps listed in Common Mistakes.

## Key Ideas

### The non-negotiable agent checklist

| # | Practice | Why |
| --- | --- | --- |
| 1 | Drive loop on **`stop_reason`** | Correct termination and tool rounds |
| 2 | **Append full history** every call | Stateless API |
| 3 | Pair **`tool_use_id`** on every `tool_result` | Valid requests, coherent model |
| 4 | Use **`is_error: true`** on tool failures | Model recovery vs hallucinated success |
| 5 | **`tool_choice`** when must invoke tools | Prompt alone doesn't guarantee |
| 6 | Validate beyond JSON Schema | **Schema ≠ semantic truth** |
| 7 | Cap **iterations** and **tokens** | Cost and RPM control |
| 8 | Least privilege **tools** per agent/node | Security |

### Programmatic vs prompt-only

Exam wording **"must", "guarantee", "production-safe"** → prefer:

- `tool_choice: { type: "tool", name: "..." }` or `{ type: "any" }`
- JSON Schema with `required` and `enum`
- Code validation gates between steps
- Hooks / approval for destructive tools
- Graph edges for mandatory ordering

Prompting alone is a weak distractor when enforcement is asked.

### Tool design

- **One clear purpose** per tool — vague mega-tools reduce accuracy
- Descriptions state **when to use** and **when not to**
- Input schema: minimal fields, strong types, examples in description
- Return **structured, truncated** results — don't flood context
- Separate **read** and **write** tools

### Error and retry discipline

- Tool failure → `tool_result` + `is_error`, continue loop — [tool-results.md](./tool-results.md)
- API 429/529 → backoff — [../03-api/retries.md](../03-api/retries.md)
- Idempotency on write tools before retries
- User-facing errors sanitized; details in logs

### Human in the loop

Require approval when tools:

- Send external communications
- Modify production data
- Spend money or change permissions

Pattern: detect `tool_use` → pause → show plan → execute on approve → `tool_result`.

Claude Code: permissions, hooks — [../05-sdk/hooks.md](../05-sdk/hooks.md).

### Observability

Log per iteration:

- `stop_reason`, model, token `usage`
- Tool names, latency, success/`is_error`
- `request-id`, session id, iteration count

Enables debugging "agent gave up" or "ran forever."

### Context management

- Summarize or prune old turns in long sessions
- Prompt cache stable **system + tools** blocks
- Multi-agent: don't merge full worker logs into parent — [multi-agent.md](./multi-agent.md)
- Large files: Files API + chunk tools — [../03-api/files.md](../03-api/files.md)

### Security

- Treat **`tool_use.input` as untrusted** — validate, parameterize queries
- Prompt injection via retrieved docs — separate instructions from data (XML tags, delimiters)
- Never expose secrets in prompts or tool results
- MCP: authenticate servers, scope tools — [../06-mcp/security.md](../06-mcp/security.md)

### Testing and evals

- Unit test history builder and tool pairing (pure code)
- Eval suites for task success rate, tool selection accuracy
- Regression when changing tool schemas or models

### Anti-patterns

| Anti-pattern | Fix |
| --- | --- |
| Infinite agent loop | Max iterations + duplicate tool detection |
| One agent, 40 tools | Split roles / multi-agent |
| Trust model "I queried DB" | Require actual `tool_use` |
| Retry entire session on 429 | Backoff + queue |
| Skip validation on structured tool output | Business rule checks + `is_error` |

## Exam Notes

- **"Production-safe agent posting refunds"** → approval + validation + write tool isolation.
- **"Ensure tool always called first step"** → `tool_choice`, not stronger prompt.
- **"Wrong JSON but valid schema"** → semantic validation distractor.
- **"Agent forgot prior tool output"** → history not appended.
- **"Simplest fix"** → often API parameter or loop fix, not new architecture.
- Identify **domain** first (Applications vs Agents vs Security) before reading answers.

## Production Notes

- Pin model versions; test upgrades in staging evals.
- Feature-flag new tools — gradual rollout.
- Run destructive tools in sandbox accounts by default.
- Document agent **SLAs**: max latency, max cost per task.
- Graceful degradation: if agent hits iteration cap, return partial + escalation path.
- Regularly audit tool permissions against actual usage.

## Common Mistakes

- Building agents without reading **`stop_reason`**.
- **`tool_choice: auto`** when exam requires guaranteed tool invocation.
- Assuming **parallel tool calls** mean you can skip any `tool_result`.
- No **`is_error`** on failures → model confabulates success.
- Believing **JSON Schema** replaces business validation.
- **Unbounded loops** — no max iterations.
- Giving agents **admin credentials** for convenience.
- Choosing multi-agent **orchestration** for a one-shot classification task.

## See Also

- [Agent Loop](./agent-loop.md)
- [Tool Use](./tool-use.md)
- [Tool Results](./tool-results.md)
- [Workflow Patterns](./workflow-patterns.md)
- [Messages API](../03-api/messages-api.md)
- [Error Handling](../03-api/error-handling.md)
- [Guardrails](../08-production/guardrails.md)
- [Security](../11-best-practices/security.md)
- [Anti-Patterns](../11-best-practices/anti-patterns.md)
