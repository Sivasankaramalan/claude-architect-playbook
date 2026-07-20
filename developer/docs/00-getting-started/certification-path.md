# Certification Path

How **CCDV F** fits Anthropic’s partner certification map, and how it relates to Architect Foundations.

## Certification Map (Foundations Focus)

| Certification | Role emphasis | Typical prep focus |
| --- | --- | --- |
| Claude Certified Associate – Foundations | Product / customer-facing | Fluency, responsible use, when to hand off |
| **Claude Certified Developer – Foundations (CCDV F)** | Hands-on builders | API, Claude Code, MCP, agents, production |
| Claude Certified Architect – Foundations (CCAF) | System designers | Orchestration, patterns, scenario architecture |
| Claude Certified Architect – Professional | Enterprise portfolio | Governance, lifecycle, stakeholder depth |

Community-reported public specs (verify officially before booking):

| Cert | Approx. questions | Time | Passing score |
| --- | ---: | ---: | --- |
| Associate – Foundations | 60 | 120 min | 720 / 1000 |
| Developer – Foundations | 53 | 120 min | 720 / 1000 |
| Architect – Foundations | 60 | 120 min | 720 / 1000 |
| Architect – Professional | 63 | 120 min | 720 / 1000 |

## Overlap With CCAF (~60–70%)

Common ground often includes:

- Agentic loops and tool use
- MCP fundamentals
- Prompt and context engineering
- Structured outputs
- Claude API basics and error handling
- Hooks, skills vs rules vs `CLAUDE.md`
- Multi-step reasoning patterns

### Emphasized More on CCDV F

- Claude Code CLI and workflows (`-p`, resume/continue, plan/headless)
- Slash commands, skills, config hierarchy
- Hooks (`PreToolUse`, `PostToolUse`, etc.)
- MCP **implementation** (server config, tool descriptions)
- SDK / Messages API mechanics
- CI/CD-oriented Claude Code usage
- Message Batches, prompt caching, JSON output as **implementation** choices

### Emphasized More on CCAF

- Which architecture / orchestration pattern to pick
- Scenario banks (support agent, research system, CI review, extraction, etc.)
- System-level reliability and provenance decisions

## Recommended Paths

### Path A — Developer first

1. Skilljar developer path + API / MCP courses  
2. This guide’s Learning Path  
3. Sit **CCDV F**  
4. Optionally deepen orchestration → sit **CCAF**

### Path B — Architect first (your situation if CCAF is done)

1. You already own most agent / MCP / context concepts  
2. Shift study time to **Applications & Integration**, **model selection**, and **Claude Code implementation**  
3. Sit **CCDV F** with a short, API-heavy revision plan

## Prep Course Spine (Official Skilljar)

Work these in order when available on Anthropic Partners Skilljar:

1. Claude Platform / API foundations  
2. Claude with the Anthropic API  
3. Introduction to Model Context Protocol  
4. Claude Code in Action / Claude Code 101  
5. Agent skills / subagents courses as listed on the Developer Foundations path  

Full link list: [`official-resources.md`](./official-resources.md).

## See Also

- Sibling Architect guide: [`../../../architect/README.md`](../../../architect/README.md)
- [Exam Blueprint](./exam-blueprint.md)
- [Overview](./overview.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
