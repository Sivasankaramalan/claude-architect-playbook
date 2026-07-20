# Pruning

**Pruning** removes or compresses conversation history and tool outputs so agent loops stay within context limits, reduce cost, and improve signal-to-noise. It is a core **Model Selection and Optimization (16.8%)** and **Agents and Workflows (14.7%)** technique.

## Learning Objectives

After this page, you should be able to:

- Identify what to prune vs what must be preserved for correctness
- Apply turn-based, token-based, and summarization pruning strategies
- Prune tool results safely without breaking `tool_use_id` pairing
- Combine pruning with external memory and retrieval
- Avoid exam traps that confuse pruning with losing business state

## Key Ideas

### Why prune

Each agent turn resends history. Without pruning:

- Input tokens grow linearly (or faster with fat tool results)
- Latency increases
- Model attention dilutes across obsolete debug logs
- Context window errors appear late in sessions

Pruning is ** intentional forgetting** of low-value tokens.

### What you must not break

| Keep | Reason |
| --- | --- |
| Recent user goals and constraints | Task continuity |
| Latest tool results needed for next step | Correct actions |
| Valid `tool_use` / `tool_result` pairs | API contract |
| Structured facts not yet persisted externally | Avoid re-hallucination |

You may drop **intermediate** tool noise after extracting structured conclusions.

### Pruning strategies

**1. Sliding window** — keep last N user/assistant turns; summarize older.

**2. Token budget** — when history > threshold, summarize or drop lowest-scored messages.

**3. Tool result compression** — replace 50KB JSON with `{ "status": "ok", "order_id": "8821" }` before next turn.

**4. Observation masking** — remove stack traces and raw HTML after extracting fields.

**5. Phase-based** — at workflow stage change, archive Phase 1 transcript; inject summary bullet list.

```text
Turn 1–8: research (verbose tool logs)
        │
        ▼ summarize
Turn 9+: "Findings: …" + only tools needed for execution phase
```

### Summarization for pruning

Use a dedicated summarization call (often Haiku-class for cost) with a strict template:

- Decisions made
- Open questions
- Entity IDs and dates
- Explicit "do not forget" list

Store summary in session state; optionally in database for resume.

### External memory vs pruning

Pruning deletes from **prompt**. External memory persists facts **outside** the model:

- User preferences → DB
- Long research → vector store + retrieval tool
- Session summary → object store keyed by `session_id`

Pruning without external memory loses recoverable facts.

### Subagents as pruning

Delegate subtasks to an isolated context; return only a **compact result** to the coordinator. This is structural pruning—see [Decomposition](../10-design-patterns/decomposition.md).

## Exam Notes

- Long session degrades → **summarize/prune history**, not switch to Opus first
- Verbose MCP/tool logs fill context → **compress tool results** before next API call
- "Forgot" early requirement → pruning too aggressive or no **external persistence**
- Subagent completed research → coordinator gets **summary**, not full subagent log
- Pruning ≠ **prompt caching** — different mechanisms

## Production Notes

### Pruning policy checklist

- [ ] Max turns before forced summarization
- [ ] Max tool result bytes stored in history (truncate with link to full blob)
- [ ] Preserve tool pairs; never orphan `tool_result`
- [ ] Log what was pruned (metadata) for debugging
- [ ] Eval long-session cases after policy changes
- [ ] User-visible "memory" backed by DB, not hope

### Safe compression example

Before: full API JSON in history  
After: assistant note + structured snippet: "Inventory: 12 units SKU-44"

The model still sees actionable state; tokens drop dramatically.

## Common Mistakes

- **Deleting tool_use without matching tool_result** — API errors
- **Pruning user safety constraints** — policy violations
- **Summaries that invent facts** — validate summary against stored records
- **Pruning only user messages** — tool results are usually the bulk
- **No regression tests for 20+ turn sessions**

## See Also

- [Context Window](./context-window.md)
- [Caching](./caching.md)
- [Cost Control](./cost-control.md)
- [Memory](../05-sdk/memory.md)
- [Sessions](../05-sdk/sessions.md)
- [Supervisor Pattern](../10-design-patterns/supervisor.md)
