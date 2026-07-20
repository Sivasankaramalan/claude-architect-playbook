# MCP Security

MCP connects LLMs to **real systems** — security is server-side enforcement, least-privilege credentials, safe errors, and host-level gates (hooks, permissions). CCDV F overlaps **Tools and MCPs** with **Security and Safety (8.1%)**.

## Learning Objectives

- Apply **least privilege** to MCP tools and credentials
- Keep secrets out of version control and tool outputs
- Validate all inputs on the server regardless of JSON Schema
- Return user-safe structured errors without leaking internals
- Combine MCP security with Claude Code hooks and permission modes

## Key Ideas

### Trust boundaries

```
Untrusted: model output + user prompts
Semi-trusted: MCP host / client configuration
Trusted: your MCP server + backing services
```

Never trust the model to “only call safe tools” — enforce at server and hooks.

### Authentication patterns

| Transport | Pattern |
| --- | --- |
| stdio | Env vars (`API_KEY`, `OAUTH_TOKEN`); OS user permissions |
| Streamable HTTP | OAuth 2.x, bearer tokens, API keys in headers; rotate regularly |

- Reference secrets as **`${ENV_VAR}`** in committed config — not literal tokens.
- **Personal** debug credentials → user-level MCP config.
- **Team/production** integrations → project config + secret manager injection in CI/CD.

### Least privilege

- Separate tools by risk: `search_tickets` vs `close_ticket`.
- Map OAuth scopes to smallest set per tool group.
- Read-only DB roles for query tools; write role only on mutation tools.
- Deny cross-tenant IDs in server validation — models hallucinate ids.

### Input validation

Two layers:

1. **JSON Schema** — types, enums, bounds (helps model + blocks garbage)
2. **Business rules** — ownership, state machine, prod vs staging flags

Schema alone does not stop semantic abuse (`delete_user` with valid id).

### Output safety

- Redact tokens, passwords, PII from tool **results** before model sees them.
- **PostToolUse hooks** can strip patterns from MCP output.
- Logs may contain more detail than model-facing text — separate channels.

### Structured errors without leakage

Good error to model:

> `AUTH expired — re-run authenticate tool (not retryable)`

Bad error:

> `psycopg2.OperationalError: connection to 10.0.0.5:5432 refused`

Include internal trace id for operators, not stack traces for the model.

### Human approval

High-risk tools (deploy, send email, spend money) should combine:

- Narrow tool existence (don’t expose in low-trust agents)
- PreToolUse **deny** or **ask** permission mode
- Optional human-in-the-loop UI in host

Prompts cannot replace this stack.

### Prompt injection via MCP

Malicious content in **resources** or tool results can instruct the model to misuse other tools. Mitigations:

- Treat external content as untrusted data
- Hook guards on dangerous tool matchers
- Output filtering and tool allowlists per task type

## Exam Notes

- **Secrets in repo** → always wrong; use env vars / secret manager.
- **Guarantee no prod writes** → hooks + server-side env checks — not CLAUDE.md.
- **Least privilege** beats “one admin token for all tools.”
- **Structured errors** — user-safe message + category; not raw stack.
- **Team MCP config** in project scope — not only `~/.claude/`.

## Production Notes

- Audit log every mutating `tools/call` with actor, tenant, inputs (redacted).
- Pen-test MCP HTTP endpoints like any API.
- Separate dev/staging/prod servers and credentials — never share tokens.
- Automate secret rotation; avoid long-lived PATs in developer laptops for prod data.
- Rate limit and alert on anomalous tool volume (agent loops).
- Review third-party MCP servers before enabling in enterprise hosts.

## Common Mistakes

- Admin database credential backing all tools
- Committing `.env` with MCP launch config
- Trusting model-generated SQL without parameterized queries / allowlists
- Returning full upstream JSON with embedded secrets to the model
- Skipping validation because “the schema looked fine”

## See Also

- [MCP Best Practices](./best-practices.md)
- [MCP Tools](./tools.md)
- [MCP Transport](./transport.md)
- [Hooks](../05-sdk/hooks.md)
- [Production Safety](../08-production/safety.md)
- [Security Best Practices](../11-best-practices/security.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
