# Security

Developer-focused **security** for Claude apps: threats, controls, and exam-aligned implementations. Primary domain **Security and Safety (8.1%)**; supports **Tools and MCPs (10.6%)**.

## Learning Objectives

After this page, you should be able to:

- Threat-model prompt injection, tool abuse, and data exfiltration paths
- Implement least privilege, sandboxing, and approval workflows in code
- Secure MCP transport, auth, and secrets handling
- Choose exam answers with programmatic enforcement over prompt-only advice
- Align logging and retention with privacy requirements

## Key Ideas

### Threat model (STRIDE-lite for agents)

| Threat | Example | Control |
| --- | --- | --- |
| **Spoofing** | Fake MCP server | TLS, auth tokens, server allowlist |
| **Tampering** | User edits tool args in proxy | Server-side validation |
| **Repudiation** | "Agent didn't delete file" | Audit logs with `request_id` |
| **Information disclosure** | Secrets in prompts/logs | Secret manager, redaction |
| **Denial of service** | Agent loop bomb | Max turns, rate limits, quotas |
| **Elevation** | Injection triggers admin tool | Hooks, role-based tool sets |

### Prompt injection defenses

1. **Trust boundaries** — system vs untrusted user vs untrusted tool/RAG content (delimiters/XML)
2. **Instruction hierarchy** — product policy in code/hooks, not repeated in user-visible areas
3. **No secrets in context** — model never sees raw API keys; tools fetch creds server-side
4. **Minimal write surface** — fewer ways to cause harm
5. **Human approval** — high-impact actions

Indirect injection via email/web/RAG is **common in production**—treat retrieved text as hostile.

### Tool and MCP security

- **Authenticate** MCP (OAuth, mTLS, private network)
- **Authorize** per user/session which tools are mounted
- **Validate** all arguments against schema + business rules
- **Sandbox** code execution (containers, no root, network egress allowlist)
- **Structure errors** — no stack traces to model

See [MCP Security](../06-mcp/security.md).

### Secrets management

- Environment variables or cloud secret manager at runtime
- Rotate on leak; scan git history
- Separate keys per environment with spend limits
- Never commit `.env`; block in CI with secret scanning

### Hooks and guardrails as security controls

**Hooks** enforce policy even when model is manipulated:

- Block `DELETE`, production writes, credential tools
- Require approval token in args for sensitive ops
- Log forensic trail

See [Hooks](../05-sdk/hooks.md) and [Guardrails](../08-production/guardrails.md).

### Data privacy

- Data minimization in prompts
- Regional requirements for AI processing
- Retention limits on logs containing user content
- User deletion flows propagate to stored summaries/embeddings

### Supply chain

- Pin MCP server versions
- Review third-party tool servers like any dependency
- CI permissions for headless Claude Code agents

## Exam Notes

- **Guarantee** no secret leakage → secrets not in prompt + scoped tools
- Injection in uploaded PDF → untrusted content handling, not "ignore harmful instructions" alone
- MCP on public internet without auth → **wrong**; private + credentials
- Destructive CLI → **hooks/permissions**, not softer system prompt
- Least privilege → separate read/write tools and credentials

## Production Notes

### Security review checklist

- [ ] Threat model documented for top 5 abuse cases
- [ ] Red team injection attempts (direct + indirect) logged
- [ ] Tool allowlist per role
- [ ] Write tools require approval or second factor
- [ ] MCP servers authenticated and network-isolated
- [ ] Secrets out of prompts/logs; scanning enabled
- [ ] Incident response runbook for suspected exfil
- [ ] Audit retention meets compliance

### Fail closed

On validation or auth failure: **deny** action, return structured error—don't partially execute.

## Common Mistakes

- **Security = model refusal only**
- **Production DB credentials on research MCP server**
- **Logging full tool args with PII**
- **Prompt NDA instead of access controls**
- **Skipping security for "internal" agents that reach prod data**

## See Also

- [Safety](../08-production/safety.md)
- [Guardrails](../08-production/guardrails.md)
- [Do's](./dos.md)
- [Don'ts](./donts.md)
- [MCP Security](../06-mcp/security.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
