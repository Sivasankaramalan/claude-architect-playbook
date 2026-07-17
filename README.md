# CCAF Public Community Study Guide

Public, community-friendly study guide for Claude Certified Architect - Foundations preparation.

This repository is organized for fast revision, deep domain study, exam pattern recognition, and mock-practice drilling. It is written as a public, community-friendly study guide using original summaries, architecture patterns, and practice prompts.

## Official Exam Blueprint

- Format: multiple choice, one correct answer and three distractors.
- Score: scaled 100-1000. Passing score: 720.
- Exam style: scenario-based production architecture questions.
- Scenarios: 4 of 6 are selected randomly. Study all 6.

| Official Domain | Weight | Focus |
| --- | ---: | --- |
| Domain 1: Agentic Architecture & Orchestration | 27% | Agentic loops, coordinators, subagents, hooks, session state |
| Domain 2: Tool Design & MCP Integration | 18% | Tool interfaces, structured errors, tool distribution, MCP config |
| Domain 3: Claude Code Configuration & Workflows | 20% | CLAUDE.md, rules, skills, plan mode, CI/CD |
| Domain 4: Prompt Engineering & Structured Output | 20% | Explicit criteria, few-shot, tool_use, JSON schema, validation |
| Domain 5: Context Management & Reliability | 15% | Context preservation, escalation, provenance, large codebases |

Note: this guide keeps a learning-oriented folder layout. The official exam calls MCP Domain 2 and Claude Code Domain 3.

## Six Production Scenarios

1. Customer Support Resolution Agent: support tools, escalation, first-contact resolution.
2. Code Generation with Claude Code: coding, refactoring, debugging, documentation.
3. Multi-Agent Research System: coordinator with search, document analysis, synthesis, reporting.
4. Developer Productivity with Claude: codebase exploration, boilerplate, built-in tools, MCP.
5. Claude Code for CI/CD: automated review, test generation, PR feedback, false-positive control.
6. Structured Data Extraction: JSON schemas, validation, edge cases, downstream integration.

## How To Use This Repository

1. Start with `00_Exam_Overview/Certification_Guide.md`, `10_Cheat_Sheets/CCA_F_Cheat_Sheet.md`, and `10_Cheat_Sheets/One_Page_Revision.md`.
2. Study one domain folder at a time. Domain 1 and Domain 4 have the highest immediate exam ROI.
3. Drill `09_Exam_Question_Bank/My_Mock_Questions.md` and any original community-contributed questions after each domain pass.
4. Use `11_Flashcards/` for active recall and `14_Revision/` for final-week planning.
5. Read `CONTENT_COVERAGE.md` to understand what this public guide includes and what it intentionally excludes.

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

## Public Content Policy

This repository is intended for public learning. Do not commit:

- Real exam questions or exam dumps.
- Private employer training material.
- Proprietary PDFs, slide decks, videos, or copied course content.
- Credentials, private links, emails, tokens, or internal system names.

Contributions should be original explanations, public-doc references, high-level summaries, and original mock questions.

## Completeness Note

This guide aims to cover the full public study path: exam overview, domains, scenarios, architecture patterns, APIs, cheat sheets, flashcards, lessons, revision plans, and original mock-question templates.

It intentionally does not include copied exam questions, private training materials, or raw source files. For production implementation details and current product behavior, always verify against official public Anthropic documentation.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
