# Learning Roadmap

Study order for **CCDV F**, aligned with official domain weights and this guide’s docs folders.

## Learning Path (Handbook Order)

```text
Getting Started
        │
        ▼
Claude Fundamentals
        │
        ▼
Prompt Engineering
        │
        ▼
Claude API
        │
        ▼
Agent Development
        │
        ▼
Agent SDK
        │
        ▼
MCP
        │
        ▼
Skills
        │
        ▼
Production Engineering
        │
        ▼
Optimization
        │
        ▼
Design Patterns
        │
        ▼
Best Practices
```

Use this order for long-term engineering skill. For a **time-boxed exam push**, weight study by P0 → P1 → P2 below.

## Priority-Weighted Study Plan

| Priority | Domains | ~Weight | Docs focus |
| --- | --- | ---: | --- |
| **P0** | Applications & Integration; Model Selection & Optimization; Agents & Workflows | **64.6%** | `03-api`, `01-foundations`, `09-performance`, `04-agent-engineering`, `05-sdk` |
| **P1** | Prompt & Context; Tools & MCP | **21.6%** | `02-prompt-engineering`, `06-mcp` |
| **P2** | Security; Claude Code; Eval/Testing/Debug | **13.8%** | `08-production`, `07-skills`, `11-best-practices` |

### How To Use Each Step

| Step | Folder | Exam domains covered |
| --- | --- | --- |
| Getting Started | [`./`](./) | Blueprint, strategy, resources |
| Claude Fundamentals | [`../01-foundations/`](../01-foundations/) | Model selection, tokens, API overview |
| Prompt Engineering | [`../02-prompt-engineering/`](../02-prompt-engineering/) | Prompt & context (P1) |
| Claude API | [`../03-api/`](../03-api/) | Applications & Integration (P0) |
| Agent Development | [`../04-agent-engineering/`](../04-agent-engineering/) | Agents & Workflows (P0) |
| Agent SDK | [`../05-sdk/`](../05-sdk/) | Apps + Agents (P0) |
| MCP | [`../06-mcp/`](../06-mcp/) | Tools & MCP (P1) |
| Skills | [`../07-skills/`](../07-skills/) | Agents + Claude Code (P0/P2) |
| Production Engineering | [`../08-production/`](../08-production/) | Security + Eval (P2) |
| Optimization | [`../09-performance/`](../09-performance/) | Model selection & optimization (P0) |
| Design Patterns | [`../10-design-patterns/`](../10-design-patterns/) | Agents & workflows (P0) |
| Best Practices | [`../11-best-practices/`](../11-best-practices/) | Cross-cutting + security |

At each step: **Learn** the docs → **Build** matching [`examples/`](../../examples/) → skim [`diagrams/`](../../diagrams/).

## 2-Hour Revision Plan (Exam Eve)

### 45 min — Claude API (Applications & Integration)

- Messages API, streaming, tool use, JSON Schema  
- `stop_reason`, `tool_choice`, Message Batches, prompt caching  

### 30 min — Claude Code

- `CLAUDE.md`, hooks, skills, slash commands  
- Headless mode, subagents, resume/continue, config hierarchy  

### 25 min — MCP

- Tools, resources, prompts  
- Server lifecycle, auth, tool descriptions  

### 20 min — Prompt Engineering

- XML, context, examples, structured output, long context  

## Last 30–45 Minutes (If Coming From CCAF)

You are likely already strong on MCP, orchestration, context, hooks, multi-agent, and structured outputs. Bias remaining time to:

1. **API implementation** — `tool_use` / `tool_result` / `stop_reason` / `tool_choice` / JSON Schema / Batches / caching  
2. **Claude Code CLI** — `-p`, `--resume`, `--continue`, plan/headless, rules/commands/skills  
3. **MCP implementation details** — not just “what MCP is,” but how you configure and secure it  

## Coming From CCAF — Mindset Shift

Think like an **application developer**, not an architect:

- Prefer answers that name a concrete API feature or Claude Code workflow  
- Ask: “How do I implement this?” not only “Which pattern is elegant?”  
- Expect fewer pure architecture picks and more production wiring questions  

## See Also

- [Exam Blueprint](./exam-blueprint.md)
- [Exam Strategy](./exam-strategy.md)
- [Developer README Learning Path](../../README.md#learning-path)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
