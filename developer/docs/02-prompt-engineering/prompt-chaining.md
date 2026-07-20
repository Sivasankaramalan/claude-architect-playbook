# Prompt Chaining

Breaking complex workflows into **sequential LLM steps** (or LLM + code steps), passing outputs forward—instead of one monolithic prompt.

> **Verify on official docs:** Workflow patterns appear across Anthropic agent and prompt guides. See also [building effective agents](https://docs.anthropic.com/en/docs/agents-and-tools/building-effective-agents).

**CCDV F domains:** Prompt and Context Engineering · Agents and Workflows · Applications and Integration

## Learning Objectives

After this page, you should be able to:

- Explain when to split work across multiple Messages API calls
- Design chains with clear contracts between steps
- Combine LLM steps with deterministic code, tools, and validation
- Manage cost, latency, and failure handling in multi-step pipelines
- Distinguish prompt chaining from agent loops driven by `stop_reason: tool_use`

## Key Ideas

### What Prompt Chaining Is

**Prompt chaining** runs a pipeline:

```
Step A (classify) → Step B (transform) → Step C (format) → User
         ↑                  ↑
    optional code      tool calls / DB
```

Each step is usually a **separate API call** with its own focused prompt—though a step may also invoke tools in an agent loop before the next step.

Contrast with **single mega-prompt:** one call asking "classify, summarize, translate, and email."

### Why Chain?

| Benefit | Explanation |
| --- | --- |
| **Higher accuracy** | Each step has one job |
| **Easier debugging** | Inspect intermediate outputs |
| **Cheaper routing** | Haiku classify → Sonnet only when needed |
| **Cleaner structure** | Step 3 emits JSON without corrupting step 1 reasoning |
| **Safer validation** | Code gates between steps |

### Chain vs Agent Loop

| Pattern | Control flow | Typical trigger |
| --- | --- | --- |
| **Prompt chain** | Your orchestrator decides next step | Fixed DAG or router output |
| **Agent loop** | Model requests tools until `end_turn` | `stop_reason: tool_use` |

They compose: a chain step may *be* an agent loop (research sub-agent), then pass summary to next step.

### Common Chain Topologies

**Linear:**

```
Extract entities → Query DB (code) → Draft email (Claude)
```

**Router:**

```
Intent classifier → {FAQ path | Ticket path | Sales path}
```

**Fan-out / map-reduce:**

```
Split document → Summarize chunks (parallel) → Merge summary
```

Map-reduce is chaining plus parallelism—not one prompt for a 200-page PDF.

### Designing Step Contracts

Each handoff should be **machine-readable** when possible:

```json
// Step 1 output (forced tool)
{ "intent": "billing", "confidence": 0.91, "needs_human": false }
```

Your orchestrator reads JSON, not prose, before step 2.

Use [structured prompts](./structured-prompts.md) per step—not paragraphs the next prompt must re-parse.

### What Belongs in Code vs Claude

| Step | Prefer |
| --- | --- |
| Regex validation, checksums | Code |
| HTTP calls to known APIs | Code (or tool wrapper) |
| Summarization, rewriting tone | Claude |
| Policy interpretation with nuance | Claude (+ citations) |
| Retry with alternate model | Orchestrator code |

Exams reward **simplest reliable split**—don't add LLM steps where code suffices.

### Context Management Across Steps

Later steps don't automatically see earlier **hidden** reasoning unless you pass it.

| Strategy | Use when |
| --- | --- |
| Pass only structured summary forward | Reduce tokens/noise |
| Pass full prior transcript | Need exact quotes |
| Fresh system prompt per step | Different policies/models |
| Shared thread history | Conversational UX continuity |

Avoid replaying 100k tokens into every step—summarize intermediate results.

### Error Handling

| Failure | Response |
| --- | --- |
| Step 1 low confidence | Route to human or clarifying question step |
| Step 2 schema invalid | Retry step 2 only with repair prompt (cap retries) |
| Step 3 tool error | Append `tool_result` is_error; retry or abort |
| Any step timeout | Degrade gracefully; log correlation ID |

Use consistent **`metadata`** or request IDs across steps for tracing.

### Cost and Latency

Each step = at least one API call:

```
3-step chain ≈ 3× network RTT + 3× input token prefixes
```

Mitigations:

- Route easy paths to Haiku
- Cache stable system prompts ([prompt caching](./prompt-caching.md))
- Parallelize independent steps (map phase)
- Collapse chain when evals show single prompt works

## Exam Notes

- **"First categorize, then generate response"** → explicit chain or router—not one vague prompt.
- **Hard reasoning then JSON** → two-step chain beats single-step format errors.
- **Parallel summarize chapters** → map-reduce chain, not sequential full re-read.
- Don't confuse with **Message Batches** (async bulk)—chains can be sync online flows.
- If scenario already uses **`tool_use` loop**, adding prompt chain on top may be overkill—pick one orchestration story.

## Production Notes

- Implement chains as explicit state machines (Temporal, Step Functions, or in-app DAG)—not copy-paste scripts.
- Persist intermediate artifacts for audit (support tickets, finance).
- Unit-test orchestrator branches with mocked Claude responses.
- Set per-step timeouts and token ceilings.
- Feature-flag chain topology changes independently from prompt text tweaks.

## Common Mistakes

- Chaining without structured handoffs—parsing prose with regex between steps
- Re-sending entire original document in every step
- No retry boundary (re-running step 1 when step 3 fails)
- Unbounded agent loops disguised as "chains"
- Using Opus for trivial routing step 1
- Hiding failures by passing empty string to next step

## See Also

- [Chain of Thought](./chain-of-thought.md)
- [Structured Prompts](./structured-prompts.md)
- [Prompting](./prompting.md)
- [Agent Loop](../04-agent-engineering/agent-loop.md)
- [Workflow Patterns](../04-agent-engineering/workflow-patterns.md)
- [Orchestrator Workers](../04-agent-engineering/orchestrator-workers.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
