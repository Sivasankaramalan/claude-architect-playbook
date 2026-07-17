# Exam Guide Summary

## Official Exam Blueprint

- Format: multiple choice, one correct answer and three distractors.
- Score: scaled 100-1000. Passing score: 720.
- Exam style: scenario-based production architecture questions.
- Scenarios: 4 of 6 are selected randomly. Study all 6.

| Official Domain | Weight | Focus |
|---|---:|---|
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

## Domain Task Map

### Domain 1: Agentic Architecture & Orchestration

- Design agentic loops for autonomous task execution.
- Orchestrate coordinator-subagent systems.
- Configure subagent invocation and explicit context passing.
- Enforce multi-step workflows and handoffs.
- Apply hooks for tool interception and normalization.
- Decompose complex workflows.
- Manage session state, resumption, and forking.

### Domain 2: Tool Design & MCP Integration

- Design clear tool interfaces and boundaries.
- Implement structured MCP tool errors.
- Distribute tools across agents.
- Configure MCP servers for project and user workflows.
- Select built-in tools effectively.

### Domain 3: Claude Code Configuration & Workflows

- Configure memory and instruction hierarchy.
- Create slash commands and skills.
- Apply path-specific rules.
- Choose plan mode versus direct execution.
- Use iterative refinement.
- Integrate Claude Code into CI/CD.

### Domain 4: Prompt Engineering & Structured Output

- Write prompts with explicit criteria.
- Use few-shot examples for consistency.
- Enforce structured output with tool use and JSON schema.
- Implement validation, retry, and feedback loops.
- Use batch strategies for asynchronous workloads.
- Design multi-instance and multi-pass review.

### Domain 5: Context Management & Reliability

- Preserve critical information across long interactions.
- Resolve ambiguity and design escalation.
- Propagate errors across multi-agent systems.
- Explore large codebases without context overload.
- Design human review workflows.
- Preserve provenance and uncertainty in synthesis.

## How To Study This Summary

Use the blueprint to decide where to spend time. Domain 1 has the highest weight, but Domain 3 and Domain 4 together cover a large portion of the exam. The exam rarely asks isolated definitions; it usually asks what architecture decision best fixes a production failure.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. All rights reserved.
