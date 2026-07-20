# Guardrails

**Guardrails** are programmatic checks that constrain what enters or leaves your Claude application—schemas, validators, hooks, classifiers, and approval workflows. They turn soft prompt preferences into **enforceable** boundaries, which is exactly what production-focused exam questions reward.

Maps to **Security and Safety (8.1%)** and **Applications and Integration (33.1%)** (structured output, `tool_choice`).

## Learning Objectives

After this page, you should be able to:

- Distinguish guardrails from prompts and from model refusals
- Apply input, output, and tool-call guardrails in an agent pipeline
- Use JSON Schema, hooks, and `tool_choice` as deterministic controls
- Design fail-closed behavior when guardrails trigger
- Combine guardrails with evals for measurable compliance

## Key Ideas

### Three guardrail zones

| Zone | Examples | Enforcement |
| --- | --- | --- |
| **Input** | PII detection, prompt injection heuristics, max length | Block or sanitize before API call |
| **Output** | JSON schema, regex, forbidden topics, citation required | Reject, retry with fix prompt, or escalate |
| **Tool calls** | Hooks, allowlists, arg validation, approval gates | Prevent execution or require human OK |

Prompts *ask*; guardrails *enforce*.

### Structured output as a guardrail

When downstream code parses model output:

- Prefer **tools with strict schemas** or **structured outputs** over "respond in JSON" in free text
- Validate with JSON Schema; reject and retry with error feedback (limited retries)
- On repeated failure, fall back to safe default or human handoff

```text
Model output → schema validate
        │
        ├─ pass → downstream
        └─ fail → retry w/ validation errors OR abort
```

### Hooks (Claude Code / Agent SDK)

**Hooks** run your code at lifecycle points—before tool execution, after model response, on session start:

- Block `rm -rf`, production DB writes, or non-allowlisted MCP tools
- Log audit events with user and args hash
- Inject reminders only when needed (don't duplicate entire system prompt)

Hooks are exam-relevant **programmatic** controls for Claude Code scenarios.

### `tool_choice` and tool surface

- `auto` — model decides (default)
- `any` — must use a tool
- `tool` — force specific tool when routing is deterministic
- Reduce tool count — fewer ways to violate policy

Narrow tools beat vague mega-tools for both safety and accuracy.

### Human-in-the-loop guardrails

For irreversible or high-impact actions:

1. Model proposes action with structured payload
2. UI shows diff / amount / target
3. User approves or rejects
4. Only then execute tool

Never auto-run payment, delete, or permission-change tools on model intent alone.

### Guardrails + evals

Each guardrail should map to **test cases**:

- Input rejected correctly?
- Invalid JSON caught?
- Blocked tool never executes even under injection prompt?

See [Evaluations](./evaluations.md) and [Testing](./testing.md).

## Exam Notes

- Stems with **guarantee**, **must prevent**, **production-safe** → guardrails (schema, hooks, validation), not "stronger system prompt"
- Structured business data → **tool use / JSON Schema**, not prose "format as JSON"
- Force specific behavior → `tool_choice`, not repeated ALL CAPS instructions
- Guardrails are **fail-closed**: deny by default when validation fails
- Distinguish **model refusal** (harmful content) from **app guardrail** (business rule)

## Production Notes

### Guardrail implementation checklist

- [ ] Input max tokens and attachment size limits
- [ ] Output schema validation with capped retry loop
- [ ] Tool allowlist per tenant/role
- [ ] Pre-tool hook validates args against server rules
- [ ] Post-tool hook sanitizes results before re-entering context
- [ ] Audit log for blocked actions
- [ ] Metrics: guardrail trigger rate by type

### Retry policy after guardrail failure

- 1–2 repair attempts with explicit validation errors to the model
- Then escalate—avoid infinite "please fix JSON" loops (cost + injection surface)

## Common Mistakes

- **Prompt-only guardrails** for security-critical rules
- **Retry forever** on schema failure — burns tokens; attacker can probe
- **Validating only final message** — tool calls execute before final answer
- **Hooks that only log** when stem requires **prevent**
- **Overlapping tools** bypass routing guardrails

## See Also

- [Safety](./safety.md)
- [Testing](./testing.md)
- [Structured Prompts](../02-prompt-engineering/structured-prompts.md)
- [Hooks](../05-sdk/hooks.md)
- [Tool Use](../04-agent-engineering/tool-use.md)
- [Error Handling](../03-api/error-handling.md)
