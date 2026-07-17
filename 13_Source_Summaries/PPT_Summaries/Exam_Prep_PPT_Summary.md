# Exam Prep PPT Summary

Focus: domain weights, six scenarios, preparation exercises, do/don't list, and final exam strategy.

## Exam Shape

The exam is scenario-based. You are usually given a production system, a failure symptom, and several plausible fixes. The correct answer is the one that most directly fixes the root cause with the right level of reliability.

## Study Priorities

1. Master the agentic loop and `stop_reason` handling.
2. Understand coordinator-subagent patterns and explicit context passing.
3. Practice writing clear tool descriptions and structured error responses.
4. Know Claude Code configuration concepts: memory, rules, skills, plan mode, and CI usage.
5. Design structured output with `tool_use`, JSON schema, validation, and retry feedback.
6. Learn context-management patterns: persistent facts, summarization risk, retrieval, and provenance.

## Recommended Practice Projects

- Build a customer-support agent with customer lookup, order lookup, refund handling, and escalation.
- Build a structured extraction pipeline with nullable fields, validation, and retry-with-feedback.
- Build a multi-agent research pipeline with search, analysis, synthesis, report generation, and provenance.
- Configure a sample Claude Code project with memory files, rules, skills, MCP config, and CI-style review output.

## Do

- Explain why wrong answers are wrong.
- Study all six scenarios.
- Use original mock questions to test root-cause reasoning.
- Practice deterministic enforcement patterns.
- Review official public documentation before implementing real systems.

## Do Not

- Memorize copied questions.
- Publish private training content.
- Treat prompt wording as a substitute for enforcement.
- Give one agent every tool.
- Wait until the last day to learn Claude Code configuration details.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. All rights reserved.
