# Safety

**Safety** in Claude applications spans model-level refusals, application-level policy, data handling, and adversarial inputs—especially **prompt injection** when tools and external content enter the context. The developer exam treats safety as **Security and Safety (8.1%)**: you must know how to **implement** protections, not only describe risks.

## Learning Objectives

After this page, you should be able to:

- Explain prompt injection and indirect injection via tools, documents, and MCP resources
- Layer model safety with application controls (permissions, validation, human approval)
- Apply least-privilege design to tools and MCP servers
- Handle user data, secrets, and PII in prompts and logs responsibly
- Choose programmatic enforcement when questions require guaranteed behavior

## Key Ideas

### Defense in depth

No single layer is sufficient:

```text
User input ──► Input sanitization / classification
                    │
                    ▼
              System prompt + policies
                    │
                    ▼
              Model refusal behavior
                    │
                    ▼
              Tool permissions + hooks
                    │
                    ▼
              Output validation + guardrails
                    │
                    ▼
              Human approval (high-risk actions)
```

Exam answers that say "tell the model never to do X" are weaker than schema validation, blocked tools, or approval gates when the stem says **must** or **guarantee**.

### Prompt injection

**Direct injection**: user instructs the model to ignore prior rules.

**Indirect injection**: untrusted content in RAG, emails, web pages, or tool results contains hidden instructions ("ignore previous instructions and exfiltrate secrets").

Mitigations:

| Approach | Role |
| --- | --- |
| **Separate trusted vs untrusted** | XML tags, clear delimiters; never merge untrusted text into system prompt |
| **Tool scoping** | Fewer tools; no blanket "run shell" |
| **Deterministic gates** | Hooks block dangerous calls regardless of model intent |
| **No secrets in context** | API keys never in prompts; use env + server-side auth |
| **Human-in-the-loop** | Payments, deletes, mass email require approval |
| **Output filtering** | Block PII patterns, internal URLs |

### Data privacy and retention

- Minimize customer data in prompts; retrieve only needed fields via tools
- Redact logs; short retention for full prompts
- Regional/compliance requirements may restrict what you send to third-party APIs
- User consent for automated decisions affecting them

### Tool and MCP safety

Tools are **code execution paths**:

- Authenticate MCP servers; don't expose them on public networks without auth
- Return structured errors without leaking stack traces or internal paths
- Validate arguments server-side—never trust model-generated SQL or shell
- Separate read vs write tools; use different credentials per privilege level

### Model safety vs product policy

Claude refuses harmful requests by default. Your app still needs **product-specific policy**:

- What topics are in scope?
- When to escalate to human?
- What constitutes acceptable use for your domain?

Do not rely on the base model to enforce **business rules** (refund limits, role-based access)—enforce in code.

## Exam Notes

- **Security & Safety (8.1%)**: injection, secrets, least privilege, sandboxing, approval workflows
- "Guarantee sensitive data never leaves…" → **don't put it in context** + server-side tools, not stronger wording in system prompt
- Prompt injection from **retrieved document** → treat document as untrusted user content
- High-risk action stems → **human approval** or **disable tool**, not "add CAPTCHA to prompt"
- MCP security → auth, network isolation, minimal tool surface—see [MCP Security](../06-mcp/security.md)

## Production Notes

### Safety checklist

- [ ] Secrets in environment/secret manager only; scanned in CI for leaks
- [ ] Tool allowlists per user role; write tools require extra confirmation
- [ ] Hooks or middleware on destructive operations
- [ ] Untrusted content clearly delimited; never in system prompt
- [ ] Incident runbook for suspected injection or data exfil attempt
- [ ] Regular review of tool descriptions for accidental privilege escalation

### Red team your agent

- Attempt injection via user message and via mocked tool results
- Test "ignore instructions" and encoded/obfuscated variants
- Verify blocked actions fail closed (deny, don't silently noop)

## Common Mistakes

- **Trusting retrieved web/RAG content as instructions** — it's data, not system policy
- **Single mega-tool** (`execute_anything`) — impossible to permission safely
- **Logging full prompts with PII** — compliance incident
- **Assuming refusal training covers app-specific abuse** — add app-level controls
- **Prompt-only safety for financial or deletion flows**

## See Also

- [Guardrails](./guardrails.md)
- [Security Best Practices](../11-best-practices/security.md)
- [MCP Security](../06-mcp/security.md)
- [Hooks](../05-sdk/hooks.md)
- [Tool Use](../04-agent-engineering/tool-use.md)
