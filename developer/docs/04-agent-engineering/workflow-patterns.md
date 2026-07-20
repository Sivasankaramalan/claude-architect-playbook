# Workflow Patterns

**Workflow patterns** are reusable structures for combining LLM steps, tools, human gates, and control flow. CCDV F expects you to pick the right pattern — agent loop, pipeline, routing, parallelization, reflection — for a given implementation scenario.

**Domain focus:** Agents and Workflows (15%) + Applications and Integration (33%).

## Learning Objectives

After this page you should be able to:

- Name common workflow patterns used with Claude (prompt chain, router, parallel, evaluator–optimizer).
- Match patterns to requirements: latency, reliability, cost, auditability.
- Implement patterns using Messages API primitives (`stop_reason`, tools, history).
- Distinguish workflows from autonomous agents.
- Select patterns on exam scenario questions.

## Key Ideas

### Pattern catalog

| Pattern | Shape | When to use |
| --- | --- | --- |
| **Single call** | One Messages API request | Simple Q&A, classification |
| **Prompt chain** | Output of step A → input of step B (deterministic) | Fixed multi-step transforms |
| **Agent loop** | `tool_use` until `end_turn` | Dynamic tool selection — [agent-loop.md](./agent-loop.md) |
| **Router** | Classify → branch to specialist prompt/model | Intent-based support bots |
| **Parallelization** | Same task on shards concurrently | Map-reduce, multi-doc — [orchestrator-workers.md](./orchestrator-workers.md) |
| **Orchestrator–workers** | Lead delegates to workers | Independent subtasks |
| **Manager / supervisor** | Manager routes + reviews subagents | QA-critical pipelines — [manager-pattern.md](./manager-pattern.md) |
| **Graph** | Explicit nodes and edges | Compliance ordering — [graph-pattern.md](./graph-pattern.md) |
| **Evaluator–optimizer** | Generate → critique → revise loop | Quality-sensitive content |

### Prompt chain (deterministic workflow)

```
Step 1: extract entities (structured tool, tool_choice forced)
Step 2: validate entities (code, no LLM)
Step 3: generate email (LLM, tool_choice none)
```

Each step is a **separate API call** with curated input — not necessarily an inner tool loop.

**Exam:** "Steps always in same order" → chain/graph, not free agent.

### Router pattern

```
User message → Router (Haiku, cheap) → label
  → if billing: billing system prompt + tools
  → if tech: tech system prompt + tools
```

Router output feeds **conditional** next call — implement via code branching, not model memory.

### Agent loop as a pattern

The agent loop is a **workflow where the model chooses the path**:

- Termination: `stop_reason: end_turn`
- Branching: which tools to call
- Requires: full history append, `tool_result` pairing

Use when **next step is data-dependent** and can't be hard-coded.

### Parallelization pattern

1. Split input (documents, URLs, test cases)
2. Run worker calls in parallel (respect rate limits)
3. Reduce/aggregate (orchestrator call or code merge)

Use **parallel tool calls** when one model turn can request multiple independent tool executions.

### Evaluator–optimizer (reflection)

```
Draft (LLM) → Evaluate (LLM or rules) → if fail: revise (LLM) → repeat
```

Stop after N iterations or when evaluator passes. Can be graph nodes or while-loop in code.

Exam: **"Improve answer quality before sending"** → reflection loop, not single shot.

### Combining patterns

Production apps stack patterns:

```
Router → Agent (tools) → Evaluator → Human approval → Send
```

Each segment uses appropriate **`tool_choice`** and **`stop_reason`** handling.

### Workflows vs Message Batches

- **Workflows** — logic for how requests relate
- **Message Batches** — async bulk transport for many independent requests — [../03-api/messages-api.md](../03-api/messages-api.md)

Don't confuse batch API with workflow orchestration.

## Exam Notes

- **"Always validate before execute"** → graph/chain, not prompt-only agent.
- **"Model picks tools based on live data"** → agent loop.
- **"Classify then specialize"** → router.
- **"10 PDFs same question"** → parallel workers + aggregate.
- **"Guarantee tool invocation for step 1"** → `tool_choice` in that step.
- Simplest pattern that meets requirements wins — don't over-architect.

## Production Notes

- Document which pattern each feature uses — onboarding and debugging.
- Instrument each step with latency and token tags.
- Fail closed on validation steps — don't proceed pipeline on `is_error` tool results without branch.
- Use Haiku for router/evaluator when quality sufficient.
- Keep workflow state in your store for resume — API won't rebuild pipeline for you.
- Test deterministic steps with unit tests; LLM steps with evals.

## Common Mistakes

- Using **free agent** when regulatory scenario requires **fixed graph order**.
- Using **complex graph** for simple single-shot task — unnecessary RPM/cost.
- Parallelizing **dependent** steps because pattern name sounds faster.
- Skipping **aggregation** after parallel map phase.
- Mixing workflow state with **incomplete message history** in agent segments.

## See Also

- [Agent Loop](./agent-loop.md)
- [Graph Pattern](./graph-pattern.md)
- [Orchestrator–Workers](./orchestrator-workers.md)
- [Manager Pattern](./manager-pattern.md)
- [Orchestration](../10-design-patterns/orchestration.md)
- [Reflection](../10-design-patterns/reflection.md)
- [Routing](../10-design-patterns/routing.md)
