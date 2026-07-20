# Exam Blueprint

Official **CCDV F (Claude Certified Developer – Foundations)** domain weights, format, and topic map for this community guide.

> Always re-check weights and format on Anthropic’s official certification / Skilljar pages before exam day — blueprints can change.

## Format

| Item | Value |
| --- | --- |
| Question types | Multiple choice and multiple response |
| Passing score | **720** (scaled 100–1000) |
| Exam style | Scenario / implementation-oriented developer questions |

## Official Domains and Weights

| Domain | Weight | Prep priority |
| --- | ---: | --- |
| Applications and Integration | **33.1%** | P0 |
| Model Selection and Optimization | **16.8%** | P0 |
| Agents and Workflows | **14.7%** | P0 |
| Prompt and Context Engineering | **11.0%** | P1 |
| Tools and MCPs | **10.6%** | P1 |
| Security and Safety | **8.1%** | P2 |
| Claude Code | **3.1%** | P2 |
| Eval, Testing, and Debugging | **2.6%** | P2 |

P0 ≈ **64.6%** of the exam. Master Applications, Model Selection, and Agents before deep-diving P2.

## Domain → Docs Mapping

| Official domain | Primary folders in this guide |
| --- | --- |
| Applications and Integration | [`03-api/`](../03-api/), [`05-sdk/`](../05-sdk/), [`01-foundations/`](../01-foundations/) |
| Model Selection and Optimization | [`01-foundations/`](../01-foundations/), [`09-performance/`](../09-performance/) |
| Agents and Workflows | [`04-agent-engineering/`](../04-agent-engineering/), [`05-sdk/`](../05-sdk/), [`10-design-patterns/`](../10-design-patterns/) |
| Prompt and Context Engineering | [`02-prompt-engineering/`](../02-prompt-engineering/), [`09-performance/`](../09-performance/) |
| Tools and MCPs | [`06-mcp/`](../06-mcp/), [`04-agent-engineering/tool-use.md`](../04-agent-engineering/tool-use.md) |
| Security and Safety | [`08-production/safety.md`](../08-production/safety.md), [`11-best-practices/security.md`](../11-best-practices/security.md) |
| Claude Code | [`07-skills/`](../07-skills/), [`05-sdk/hooks.md`](../05-sdk/hooks.md) *(CLI/config pages to expand)* |
| Eval, Testing, and Debugging | [`08-production/evaluations.md`](../08-production/evaluations.md), [`08-production/testing.md`](../08-production/testing.md), [`08-production/debugging.md`](../08-production/debugging.md) |

## P0 Topic Checklist

### 1. Applications and Integration (33.1%)

Expect “which API feature?” and production reliability questions.

- Messages API request/response lifecycle
- Streaming vs synchronous
- Message Batches
- Tool use + tool results in history
- Structured output / JSON Schema
- System prompts and conversation history
- `stop_reason`, `tool_choice`
- Multimodal requests
- SDK usage patterns
- Rate limits, retries, error handling
- Context window, token counting
- Prompt caching

### 2. Model Selection and Optimization (16.8%)

- When to choose Opus / Sonnet / Haiku
- Latency vs cost vs reasoning vs coding tradeoffs
- Large context workloads
- Prompt caching and token optimization
- Extended thinking
- Streaming as a latency UX choice

### 3. Agents and Workflows (14.7%)

Large overlap with Architect — still testable with developer framing.

- Agent loop
- Subagents / context isolation
- Planning
- Human approval
- Hooks
- Memory and sessions
- Resume / continue
- Stop reasons driving control flow

## P1 Topic Checklist

### Prompt and Context Engineering (11.0%)

- XML tags, delimiters, role prompting
- Few-shot examples
- Chain of thought (when appropriate)
- Context engineering / long context
- Structured output contracts

### Tools and MCPs (10.6%)

- Resources, tools, prompts, sampling
- Transport and server registration
- Tool descriptions and permissions
- Authentication and hooks
- Common MCP design mistakes (vague tools, unstructured errors)

## P2 Topic Checklist

### Security and Safety (8.1%)

- Prompt injection
- Secrets / API keys
- Least privilege, tool permissions, user approval
- Sandboxing and data privacy

### Claude Code (3.1%)

Small weight, often free points if you use the product.

- `CLAUDE.md`, `.claude/rules`, slash commands
- Skills, hooks, subagents
- Headless / plan mode, permissions
- Resume / continue, configuration hierarchy
- CLI flags such as `-p`, `--resume`, `--continue`

### Eval, Testing, and Debugging (2.6%)

- Offline / regression / human / A/B evaluation
- Prompt evaluation
- Debugging agent failures

## High-Probability Question Shapes

- Which **API parameter** solves this?
- Which **Claude feature** should be used?
- Best **developer workflow** (Claude Code / SDK)?
- Most **reliable production** solution?
- Best **MCP** design?
- Context management / forgetting history?
- Tool selection / `tool_choice`?
- Error handling / retries?
- Session management (resume vs fresh)?
- Streaming vs synchronous?

## See Also

- [Overview](./overview.md)
- [Learning Roadmap](./learning-roadmap.md)
- [Exam Strategy](./exam-strategy.md)
- [Official Resources](./official-resources.md)
