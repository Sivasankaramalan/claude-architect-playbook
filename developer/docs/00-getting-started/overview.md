# Overview

Unofficial community overview of **Claude Certified Developer – Foundations (CCDV F)**.

This guide prepares you to **build** with Claude — Messages API, tool use, MCP, Claude Code, agents, and production practices — not only to pass the exam.

## What CCDV F Is

| Item | Detail |
| --- | --- |
| Full name | Claude Certified Developer – Foundations |
| Short name | CCDV F |
| Role focus | Hands-on developers who ship Claude apps and workflows |
| Question style | Multiple choice and multiple response |
| Passing score | **720** (scaled 100–1000) |
| Sibling cert | [`Claude Certified Architect – Foundations (CCAF)`](../../../architect/README.md) — design / architecture emphasis |

Typical public prep specs (community-reported; verify on official Skilljar / Pearson pages before booking): ~53 questions, ~120 minutes, online-proctored or test center, English, ~12-month validity.

## Mental Model: Developer vs Architect

CCDV F shares a large conceptual overlap with CCAF (often cited around **60–70%**), but the **question framing shifts**:

| CCAF (Architect) | CCDV F (Developer) |
| --- | --- |
| Which architecture / pattern is best? | How would you **implement** this with the API or Claude Code? |
| System design and orchestration tradeoffs | Request lifecycle, SDK patterns, CLI/config, retries, streaming |
| Defend a topology | Choose the right API parameter, tool schema, or Claude Code workflow |

If you already passed CCAF, lean into **application developer** thinking: `stop_reason`, `tool_choice`, Message Batches, prompt caching, Claude Code config hierarchy, MCP server implementation details.

## How This Guide Is Organized

1. **Learn** — concise concept pages under [`docs/`](../)
2. **Build** — runnable examples under [`examples/`](../../examples/)
3. **Architect / Ship** — patterns, production, performance, best practices
4. **Reference** — handbook value after the exam

Start with:

1. [`exam-blueprint.md`](./exam-blueprint.md) — domains and weights
2. [`certification-path.md`](./certification-path.md) — how CCDV F fits the cert map
3. [`learning-roadmap.md`](./learning-roadmap.md) — study order by priority
4. [`exam-strategy.md`](./exam-strategy.md) — question patterns and tactics
5. [`official-resources.md`](./official-resources.md) — Skilljar / public links

## Golden Rules (Developer Exam)

- Prefer **programmatic** enforcement (schemas, hooks, validation, `tool_choice`) over prompt-only answers when the question says *guarantee*, *must*, or *production-safe*.
- Agentic loop control comes from **`stop_reason`**: `tool_use` → continue with tool results; `end_turn` → stop.
- Tool results must be **appended** to conversation history before the next model call.
- Choose the **simplest** fix that addresses the root cause (API feature vs rewrite architecture).
- Identify the **domain** before reading distractors.

## See Also

- [Exam Blueprint](./exam-blueprint.md)
- [Learning Roadmap](./learning-roadmap.md)
- [Developer track README](../../README.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
