# Domain 1 PPT Summary

Focus: agentic loop, coordinator-subagent orchestration, context passing, hooks, task decomposition, session state, and deterministic enforcement.

## What To Master

Domain 1 is the highest-weighted part of the exam. It tests whether you can design autonomous task execution safely rather than simply asking Claude to "try harder."

The core unit is the agentic loop:

1. Send the conversation, tools, and current task to Claude.
2. Inspect `stop_reason`.
3. If `stop_reason` is `tool_use`, execute the tool and append the result.
4. If `stop_reason` is `end_turn`, return the final answer.

Do not infer completion from natural-language text. A response can contain text and still require tool execution.

## Coordinator And Subagents

Use hub-and-spoke architecture when work has specialist roles. The coordinator owns routing, context selection, retries, error handling, and final synthesis. Subagents should not call each other directly.

Important implication: subagents do not automatically inherit coordinator memory. If a synthesis agent needs web URLs, document names, page numbers, confidence scores, or previous findings, the coordinator must pass them explicitly.

## Decomposition Patterns

- Use fixed pipelines for predictable workflows such as extraction, code review, and report generation.
- Use dynamic decomposition for open-ended investigation, unfamiliar codebases, and research tasks.
- Use multi-pass architecture when one huge pass causes attention dilution.
- Use broad decomposition first when a topic may contain many categories.

## Enforcement

Prompts influence behavior. Hooks and prerequisite gates enforce behavior. If the action involves money, security, compliance, permissions, or irreversible state change, prefer deterministic enforcement.

## Common Exam Traps

- Stopping the loop because Claude returned text.
- Giving all tools to one agent.
- Assuming subagents share memory.
- Fixing a missed compliance step with stronger prompt wording.
- Blaming a downstream agent when the coordinator decomposed the work too narrowly.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. All rights reserved.
