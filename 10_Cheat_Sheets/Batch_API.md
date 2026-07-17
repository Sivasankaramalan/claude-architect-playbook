# Batch API Cheat Sheet


## Golden Exam Rules

- If the question says guarantee, do not choose a system-prompt-only answer. Use programmatic enforcement, schemas, hooks, gates, or tool_choice.
- Agentic loop control comes from `stop_reason`: `tool_use` means continue, `end_turn` means stop.
- Tool results must be appended back into conversation history before the next model call.
- Subagents do not inherit coordinator history. Pass all needed context explicitly.
- Agents should not all receive every tool. Scope tools by responsibility, usually 4-5 per agent.
- Vague or overlapping tool descriptions are a root cause of wrong tool selection.
- Use structured errors with category, retryability, user-safe message, and recovery guidance.
- Required fields force hallucination when source data may be absent. Use nullable optional fields.
- Tool use guarantees JSON shape, not semantic truth. Validate business rules separately.
- Independent review in fresh context is stronger than self-review in the same context.
- Preserve critical facts outside lossy summaries in a persistent case-facts block.
- Never bury critical instructions or findings in the middle of huge context.


## Exam Traps


## Common Distractor Patterns

- Add stronger prompt instructions for behavior that must be guaranteed.
- Parse natural-language completion phrases instead of checking `stop_reason`.
- Give one agent every tool to avoid routing complexity.
- Assume subagents share memory or inherit coordinator context.
- Retry business-rule or permission errors as if they were transient.
- Use `tool_choice: auto` while expecting a guaranteed tool call.
- Make optional data required, causing `N/A`, `UNKNOWN`, or fabricated values.
- Ask the same model to self-review its own output in the same context.
- Use larger context windows as the first fix for attention or summarization problems.
- Store team-critical MCP config only in user-level configuration.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. All rights reserved.
