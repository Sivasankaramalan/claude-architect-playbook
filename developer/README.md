# Claude Certified Developer Guide

Unofficial, community-friendly study guide for **Claude Certified Developer – Foundations (CCDV F)**.

This is the **Developer** track inside the [Claude Certification Playbook](../README.md). Sibling track: [`../architect/`](../architect/).

**Status:** Getting Started + all topic docs filled from public Anthropic / MCP documentation. Verify live model IDs, pricing, and API fields on official docs before exam day.

## Official Exam Snapshot

| Item | Detail |
| --- | --- |
| Exam | Claude Certified Developer – Foundations (CCDV F) |
| Format | Multiple choice and multiple response |
| Passing score | **720** (scaled 100–1000) |

| Domain | Weight | Priority |
| --- | ---: | --- |
| Applications and Integration | 33.1% | P0 |
| Model Selection and Optimization | 16.8% | P0 |
| Agents and Workflows | 14.7% | P0 |
| Prompt and Context Engineering | 11.0% | P1 |
| Tools and MCPs | 10.6% | P1 |
| Security and Safety | 8.1% | P2 |
| Claude Code | 3.1% | P2 |
| Eval, Testing, and Debugging | 2.6% | P2 |

Full topic checklists: [`docs/00-getting-started/exam-blueprint.md`](./docs/00-getting-started/exam-blueprint.md).

## What Makes This Repository Stand Out

Instead of being another “exam answers” dump, this track is positioned as a **production-ready Claude engineering handbook**:

| Pillar | What you get |
| --- | --- |
| **Learn** | Concise explanations of every core concept |
| **Build** | Runnable code examples for each topic |
| **Architect** | Design patterns, workflows, and production guidance |
| **Ship** | Deployment, observability, testing, evaluations, and optimization |
| **Reference** | A practical handbook to return to long after certification |

That lasting value matters as much as the exam: the same material should help AI engineers build real-world applications with Claude.

## Learning Path

Follow this sequence for the strongest foundation → production progression:

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

| Step | Docs section |
| --- | --- |
| Getting Started | [`docs/00-getting-started/`](./docs/00-getting-started/) |
| Claude Fundamentals | [`docs/01-foundations/`](./docs/01-foundations/) |
| Prompt Engineering | [`docs/02-prompt-engineering/`](./docs/02-prompt-engineering/) |
| Claude API | [`docs/03-api/`](./docs/03-api/) |
| Agent Development | [`docs/04-agent-engineering/`](./docs/04-agent-engineering/) |
| Agent SDK | [`docs/05-sdk/`](./docs/05-sdk/) |
| MCP | [`docs/06-mcp/`](./docs/06-mcp/) |
| Skills | [`docs/07-skills/`](./docs/07-skills/) |
| Production Engineering | [`docs/08-production/`](./docs/08-production/) |
| Optimization | [`docs/09-performance/`](./docs/09-performance/) |
| Design Patterns | [`docs/10-design-patterns/`](./docs/10-design-patterns/) |
| Best Practices | [`docs/11-best-practices/`](./docs/11-best-practices/) |

## Quick Start

1. Read [`docs/00-getting-started/overview.md`](./docs/00-getting-started/overview.md)
2. Study [`docs/00-getting-started/exam-blueprint.md`](./docs/00-getting-started/exam-blueprint.md) (P0 first)
3. Follow the Learning Path / [`learning-roadmap.md`](./docs/00-getting-started/learning-roadmap.md)
4. Drill tactics in [`exam-strategy.md`](./docs/00-getting-started/exam-strategy.md)
5. Use official Skilljar links in [`official-resources.md`](./docs/00-getting-started/official-resources.md)
6. Build along with [`examples/`](./examples/) and revise with [`diagrams/`](./diagrams/)

## Docs Map

| Section | Focus |
| --- | --- |
| [`00-getting-started`](./docs/00-getting-started/) | Overview, blueprint, path, roadmap, strategy, official links |
| [`01-foundations`](./docs/01-foundations/) | LLM basics, Claude models, API overview, tokens |
| [`02-prompt-engineering`](./docs/02-prompt-engineering/) | Prompting patterns, XML, CoT, caching, debugging |
| [`03-api`](./docs/03-api/) | Messages API, streaming, vision, files, errors |
| [`04-agent-engineering`](./docs/04-agent-engineering/) | Agent loops, tools, multi-agent patterns |
| [`05-sdk`](./docs/05-sdk/) | Agent SDK, hooks, sessions, memory, tracing |
| [`06-mcp`](./docs/06-mcp/) | MCP architecture, servers, tools, security |
| [`07-skills`](./docs/07-skills/) | Skills structure, when to use, vs MCP |
| [`08-production`](./docs/08-production/) | Observability, evals, safety, deployment |
| [`09-performance`](./docs/09-performance/) | Context, caching, latency, cost control |
| [`10-design-patterns`](./docs/10-design-patterns/) | Architecture and orchestration patterns |
| [`11-best-practices`](./docs/11-best-practices/) | Dos, don'ts, anti-patterns, checklist |
| [`glossary.md`](./docs/glossary.md) | Shared terms |

## Repo Layout

```text
developer/
├── docs/          # Study material (Learn / Architect / Ship)
├── examples/      # Runnable examples (Build)
├── diagrams/      # Architecture and pattern diagrams
├── assets/        # Images, icons, banners
└── website/       # Optional docs site (later)
```

## Public Content Policy

Original explanations and public-doc references only. No exam dumps, private course copies, or secrets. See [`../PUBLICATION_CHECKLIST.md`](../PUBLICATION_CHECKLIST.md).

## Contributing

See [`CONTRIBUTING.md`](./CONTRIBUTING.md), [`CODE_OF_CONDUCT.md`](./CODE_OF_CONDUCT.md), and [`ROADMAP.md`](./ROADMAP.md).

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
