# Certification Guide

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

Note: this repository keeps the folder layout requested by the owner. The official exam calls MCP Domain 2 and Claude Code Domain 3.


## Six Production Scenarios

1. Customer Support Resolution Agent: support tools, escalation, first-contact resolution.
2. Code Generation with Claude Code: coding, refactoring, debugging, documentation.
3. Multi-Agent Research System: coordinator with search, document analysis, synthesis, reporting.
4. Developer Productivity with Claude: codebase exploration, boilerplate, built-in tools, MCP.
5. Claude Code for CI/CD: automated review, test generation, PR feedback, false-positive control.
6. Structured Data Extraction: JSON schemas, validation, edge cases, downstream integration.

## Target Candidate

A solution architect who can design and implement production Claude applications using Claude Code, Claude APIs, Agent SDK-style agentic systems, MCP integrations, prompt engineering, structured output, context management, and CI/CD workflows.

## What The Exam Really Tests

The exam is not documentation recall. It tests whether you can select the safest and most proportionate architecture for a production failure mode. Most questions describe a symptom and ask for the best fix. The correct answer usually addresses the root cause directly, while distractors are plausible but incomplete.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
