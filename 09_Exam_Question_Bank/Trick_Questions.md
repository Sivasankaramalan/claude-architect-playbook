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
