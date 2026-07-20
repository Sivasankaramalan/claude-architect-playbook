# Exam Strategy

Tactics for **CCDV F** multiple-choice / multiple-response questions.

## Before You Look at Options

1. **Name the domain** — Applications? Agents? MCP? Security?  
2. Ask: **What Anthropic principle is this testing?** (stop reason loop, deterministic enforcement, tool scoping, context isolation, etc.)  
3. Decide whether the fix is **API parameter**, **Claude Code config**, **MCP design**, or **prompt/context** — then eliminate the other buckets.

## Prefer Deterministic Over Prompt-Only

When the stem says *guarantee*, *must*, *prevent*, or *production*:

| Weaker distractor | Stronger answer class |
| --- | --- |
| “Add more instructions to the system prompt” | Schema / `tool_choice` / hooks / validation / permissions |
| “Tell the model to always…” | Programmatic gate or structured contract |
| “Hope the model remembers…” | Persist facts / pass explicit context / resume correctly |

Same philosophy as CCAF — CCDV F just asks you to **name the implementation**.

## High-Frequency Decision Axes

### `stop_reason`

- `tool_use` → run tool(s), append `tool_result`, call again  
- `end_turn` → stop the agent loop  
- Do not invent a parallel control channel when stop reason already decides the next step  

### Streaming vs Synchronous vs Batches

- Interactive UX / TTFB → streaming  
- Simple request/response → synchronous Messages API  
- Offline / deferrable / cost-sensitive bulk → Message Batches  

### Resume vs Fresh Session

- Prior context still valid → resume / continue  
- Stale tool results or poisoned context → fresh session + injected summary  

### Model Choice

- Router / cheap / fast steps → Haiku-class  
- Balanced production default → Sonnet-class  
- Hard reasoning / high-stakes synthesis → Opus-class  
- Always weigh **latency, cost, quality** against the task  

### MCP Tool Design

- Clear, non-overlapping tool descriptions  
- Structured errors (not raw stack traces)  
- Least-privilege tools and auth boundaries  

## Multiple-Response Tips

- Select **all** options that are necessary and correct — not “the single best essay.”  
- Watch for pairs: one correct API feature + one correct operational practice.  
- Discard options that restate the problem without fixing the root cause.

## Common Traps

- Treating subagents as if they inherit coordinator history automatically  
- Assuming tool use guarantees **semantic** correctness (it guarantees shape/flow, not business truth)  
- Choosing a complex multi-agent design when a single tool-using loop suffices  
- Ignoring rate limits / retries / idempotency in “production” stems  
- Over-weighting Claude Code for questions that are really Messages API  

## Timed Practice Habit

For each practice scenario:

1. Domain tag (5 seconds)  
2. Root cause in one sentence  
3. Eliminate prompt-only if a deterministic control exists  
4. Pick the simplest remaining correct option  

## See Also

- [Exam Blueprint](./exam-blueprint.md)
- [Learning Roadmap](./learning-roadmap.md)
- [Overview](./overview.md)
