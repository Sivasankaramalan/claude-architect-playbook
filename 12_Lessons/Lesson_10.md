# Lesson 10: Final Mock Review And Revision

## Goal

Understand the concepts well enough to recognize scenario symptoms and choose the safest architecture answer.

## Study Tasks

- Read the related domain notes.
- Write three root-cause examples.
- Answer 10 question-bank items.
- Explain why each distractor is wrong.

## Must Remember


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
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. All rights reserved.
