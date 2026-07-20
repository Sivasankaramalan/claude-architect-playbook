# MCP Best Practices

Production MCP integrations succeed when tools are **narrow**, descriptions are **routing-grade**, errors are **structured**, transports match deployment, and security is enforced on the **server** plus **hooks** — not in prompts alone.

## Learning Objectives

- Apply a checklist for designing MCP tool catalogs
- Diagnose common failure modes (wrong tool, retries, context bloat)
- Integrate MCP with agent loops, hooks, and evals
- Operate servers with versioning, monitoring, and least privilege
- Contrast anti-patterns called out in CCDV F materials

## Key Ideas

### Design checklist

1. **One job per tool** — verbs in names (`create_`, `search_`, `get_`)
2. **Non-overlapping descriptions** — explicit “use X instead when …”
3. **JSON Schema** with field-level descriptions
4. **Nullable optional** fields when source data may be missing
5. **Structured errors** — category, retryable, safe message, recovery hint
6. **Resources** for read-only corpora; **tools** for mutations and computed fetches
7. **4–5 tools per agent role** — not full catalog everywhere
8. **Project-scoped** server config for team integrations

### Tool description template (conceptual)

```
Search open support tickets by keyword. Use when the user asks about
existing issues. Do NOT use for creating tickets (use create_ticket).
Returns up to 20 summaries: id, title, status. Empty list if none.
```

### Error handling matrix

| Category | retryable | Agent behavior |
| --- | --- | --- |
| RATE_LIMIT | true | Backoff, retry |
| VALIDATION | false | Fix inputs, ask user |
| AUTH | false | Run auth flow first |
| UPSTREAM | maybe | Retry if idempotent |
| PERMISSION | false | Escalate human |

Do **not** retry permission or validation errors like transient network faults.

### Transport and ops

- Local dev: **stdio** + fast iteration
- Shared services: **Streamable HTTP** + OAuth + HA
- stderr for logs (stdio); health checks (HTTP)
- Version servers; pin in project config

### Host integration

- Merge MCP tools with built-in tools — hooks apply to **both**
- **PreToolUse** for policy; **PostToolUse** for normalize/audit
- SDK **`allowedTools`** for blast-radius reduction
- Trace MCP latency separately — slow tools dominate turns

### Testing and evals

- Golden tests per tool: happy path, auth failure, invalid id
- Offline eval tasks that require specific tool sequences
- Regression when descriptions change — routing shifts silently

### Relationship to Messages API best practices

Same loop invariants:

- Append **tool_result** before next model call
- Drive loop on **`stop_reason`**
- **`tool_choice`** when guarantee required
- Independent review in **fresh context**, not same polluted thread

## Exam Notes

- **Vague / overlapping tools** — root cause distractor for mis-routing.
- **Unstructured errors** — wrong for production stems.
- **Required optional data** — causes hallucinated field values.
- **tool_use shape ≠ truth** — validate business rules separately.
- **Store team MCP in user config only** — wrong for shared repos.

## Production Notes

- Run periodic **tool catalog reviews** — delete unused tools.
- Dashboard: success rate, p95 latency, error category breakdown per tool.
- Document ownership and SLAs for each MCP server.
- Chaos-test upstream failures — agents loop aggressively.
- Align MCP tool names with hook matchers and allowlists explicitly.

## Common Mistakes

| Mistake | Fix |
| --- | --- |
| 30 similar tools | Consolidate + clarify descriptions |
| Stack traces to model | Structured errors |
| One OAuth scope to rule them all | Split tools + credentials |
| Resources for writes | Mutation tools with validation |
| No hooks on MCP Bash equivalents | PreToolUse deny patterns |
| Prompt-only “never delete prod” | Server + hook enforcement |

## See Also

- [MCP Tools](./tools.md)
- [MCP Security](./security.md)
- [MCP Servers](./servers.md)
- [Hooks](../05-sdk/hooks.md)
- [Anti-Patterns](../11-best-practices/anti-patterns.md)
- [Evaluations](../08-production/evaluations.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
