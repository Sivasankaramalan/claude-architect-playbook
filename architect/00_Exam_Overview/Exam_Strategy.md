# Exam Strategy

## Strategy

- Study all six scenarios. The exam picks four randomly.
- Prioritize Domain 1 first because it is the highest weighted and appears across many scenarios.
- Use scenario thinking: identify the failure mode, classify the risk, then select the architecture control.
- Eliminate prompt-only answers when guarantees, financial risk, security, compliance, or auditability are required.
- Prefer the most direct root-cause fix over over-engineered classifiers, confidence scores, or generic retry loops.


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


## Answering Pattern

1. Identify the symptom.
2. Classify whether it is selection, context, validation, enforcement, escalation, or decomposition.
3. Decide whether the required behavior is probabilistic or deterministic.
4. Choose the answer that changes the system boundary most directly.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
