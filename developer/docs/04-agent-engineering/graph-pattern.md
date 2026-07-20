# Graph Pattern

The **graph pattern** models agent workflows as **nodes** (steps/agents) and **edges** (transitions), with explicit state passed between nodes. Unlike purely prompt-driven managers, graphs make control flow **visible, testable, and deterministic** where needed.

**Domain focus:** Agents and Workflows (15%) — structured workflows vs free-form agents.

## Learning Objectives

After this page you should be able to:

- Define nodes, edges, and shared state in a graph-based agent workflow.
- Contrast graph workflows with open agent loops and manager routing.
- Map Claude API calls to graph node executions.
- Identify when exams prefer explicit graphs over autonomous agents.
- Combine graphs with tool use and conditional branching.

## Key Ideas

### Core concepts

| Concept | Meaning |
| --- | --- |
| **Node** | One step: LLM call, tool execution, human approval, transformer |
| **Edge** | Transition condition to next node |
| **State** | Shared object (JSON) mutated or passed between nodes |
| **Entry / exit** | Start node from user input; exit when terminal node completes |

Example mental model:

```
[ ingest ] → [ classify ] → { support? → [ support_agent ] : [ sales_agent ] } → [ summarize ] → END
```

### LLM nodes in the graph

Each LLM node typically:

1. Builds messages from **state** slice + node-specific system prompt
2. Calls Messages API (optional tools, `tool_choice` per node)
3. Runs inner agent loop if `stop_reason: tool_use` until node complete
4. Writes outputs back to **state** (structured fields)
5. Follows edge to next node

**Per-node `stop_reason` handling** — inner loops don't escape the graph; only terminal nodes return to user.

### Graph vs free agent loop

| Free agent loop | Graph pattern |
| --- | --- |
| Model picks next action | Developer defines allowed transitions |
| Flexible, harder to test | Predictable, easier compliance |
| Risk of loops/wander | Bounded paths |
| Best for exploratory tasks | Best for regulated pipelines |

Hybrid: graph skeleton with **agentic nodes** (LLM + tools inside one node).

### Conditional edges

Branch on:

- **Structured classifier output** (category enum in state)
- **Tool success/failure** (`is_error` flag from prior node)
- **Human approval** boolean in state
- **Deterministic rules** (amount > threshold → escalation node)

Exam: **"Must always run safety check before send"** → graph edge, not prompt hope.

### State management

State object might contain:

```json
{
  "user_query": "...",
  "classification": "billing",
  "retrieved_docs": [],
  "draft_reply": "",
  "approved": false
}
```

- **Prune** what each node sees — context engineering per node
- Persist state for **resume/continue** after failures
- Don't dump entire history into every node if summaries suffice

### Tool use within nodes

Tools available **per node** — not global:

- `retrieve` node: search tools only
- `action` node: write tools with approval
- Reduces prompt injection surface — least privilege

Parallel tool calls still apply **inside** a node's agent loop.

### Frameworks

LangGraph-style frameworks compile graphs to executable workflows. CCDV F tests **concepts** — nodes, state, conditional edges — whether or not you use a library.

### vs orchestrator–workers

- **Orchestrator–workers:** dynamic delegation, often LLM-planned fan-out
- **Graph:** predefined topology; LLM fills node content but doesn't invent new topology mid-flight (unless meta-node allowed)

## Exam Notes

- **"Mandatory ordering: fetch → validate → send"** → graph with fixed edges.
- **"Audit trail of steps"** → graph nodes with logged state transitions.
- **"Agent keeps skipping compliance step"** → replace free loop with graph gate.
- **"Branch on ticket type"** → conditional edge after classify node.
- Distractor: bigger single agent prompt when **explicit graph** is required for guarantees.

## Production Notes

- Persist **state + current node id** for crash recovery.
- Version graph definitions; feature-flag topology changes.
- Unit test edges deterministically; LLM-eval node outputs separately.
- Set per-node **max_tokens**, timeouts, and tool allowlists.
- Human-in-the-loop as explicit **approval node** — not informal chat pause.
- Visualize graph in ops dashboards for on-call.

## Common Mistakes

- Giant **global tool list** on every node — security and confusion risk.
- Storing unbounded raw history in **state** — token explosion on later nodes.
- No terminal **END** node — workflow hangs.
- Using graph for **fully exploratory research** where free agent is simpler and fine.
- Ignoring **`tool_result` / `tool_use_id` rules** inside node sub-loops.

## See Also

- [Workflow Patterns](./workflow-patterns.md)
- [Manager Pattern](./manager-pattern.md)
- [Orchestrator–Workers](./orchestrator-workers.md)
- [Routing](../10-design-patterns/routing.md)
- [Agent Loop](./agent-loop.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
