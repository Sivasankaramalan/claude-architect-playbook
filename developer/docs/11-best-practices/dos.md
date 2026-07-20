# Do's

Practices that consistently improve Claude application **quality, safety, and operability**—aligned with CCDV F domains across Applications (33.1%), Agents (14.7%), Optimization (16.8%), and Security (8.1%).

## Learning Objectives

After this page, you should be able to:

- Apply Anthropic-aligned developer best practices across API, agents, and MCP
- Prefer programmatic enforcement where exams ask for guarantees
- Build maintainable prompts, tools, and orchestration code
- Tie practices to observability, evals, and cost control

## Key Ideas

### API and integration

- **Do** handle `stop_reason` explicitly in agent loops (`tool_use` → execute → continue; `end_turn` → stop).
- **Do** append `tool_result` messages with correct `tool_use_id` before the next API call.
- **Do** implement exponential backoff for 429/529 with jitter and max retries.
- **Do** use streaming for interactive user-facing latency.
- **Do** log `request_id` from response headers for support correlation.
- **Do** validate structured outputs with JSON Schema; retry with validation errors (capped).
- **Do** use Message Batches for large offline/deferrable workloads.

### Model and performance

- **Do** route simple steps to Haiku-class models; reserve Opus for genuinely hard reasoning.
- **Do** place prompt cache breakpoints after stable system prompts and tool definitions.
- **Do** prune or summarize long histories; persist durable facts outside the context window.
- **Do** count tokens before sending large documents or tool manifests.
- **Do** measure cost per **successful task**, not per call alone.

### Tools and MCP

- **Do** write clear, non-overlapping tool descriptions with focused schemas.
- **Do** return structured, actionable errors from tools (code, message, retryable flag).
- **Do** validate tool arguments server-side—never trust model-generated SQL/shell blindly.
- **Do** apply least privilege: separate read and write tools; scoped credentials.
- **Do** authenticate MCP servers and keep them off public networks without controls.

### Agents and workflows

- **Do** cap max agent turns and tool timeouts.
- **Do** pass explicit context when spawning subagents—never assume inheritance.
- **Do** use supervisor/worker patterns when domains parallelize; use single loop when sufficient.
- **Do** use hooks for audit and block high-risk tool calls deterministically.

### Safety and production

- **Do** keep secrets in environment/secret managers—out of prompts and logs.
- **Do** delimit untrusted content (RAG, web, email) separately from system instructions.
- **Do** require human approval for irreversible or financial actions.
- **Do** maintain regression eval suites tied to prompt and tool versions.
- **Do** version and deploy prompts, tools, and code together with rollback plans.

### Claude Code

- **Do** commit `CLAUDE.md` and `.claude/rules` to the repo for team consistency.
- **Do** use headless mode in CI with explicit permission scopes and timeouts.

## Exam Notes

- Stems asking what you **should** do in production → look for **deterministic**, **simplest correct** options
- "Guarantee" / "must" → schema, hooks, validation, approval—not stronger adjectives in prompts
- Reliability → retries, idempotency, structured errors
- Long sessions → prune/summarize + external memory

## Production Notes

These do's are intentionally overlapping with [Checklist](./checklist.md)—use the checklist before launch and this page for rationale during design reviews.

## Common Mistakes

- Treating this list as optional "nice to haves" on high-risk flows
- Implementing do's in prompts only without code enforcement
- Adopting every pattern at once instead of simplest sufficient subset

## See Also

- [Don'ts](./donts.md)
- [Anti-Patterns](./anti-patterns.md)
- [Checklist](./checklist.md)
- [Security](./security.md)
- [Exam Strategy](../00-getting-started/exam-strategy.md)
