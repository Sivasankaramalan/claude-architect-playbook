# Anti-Patterns

**Anti-patterns** are recurring bad designs that look reasonable in demos but fail under production load, adversarial input, or exam scrutiny. Each entry includes why it fails and what to do instead.

## Learning Objectives

After this page, you should be able to:

- Name common LLM application anti-patterns and their fixes
- Map anti-patterns to exam distractors
- Refactor designs toward simpler, enforceable architectures

## Key Ideas

### 1. Prompt-only security

**Symptom**: "You must never reveal secrets" in system prompt; no hooks or tool scoping.

**Why it fails**: Prompt injection and jailbreaks bypass instructions.

**Instead**: Least-privilege tools, secrets out of context, hooks, human approval for sensitive actions.

---

### 2. God tool

**Symptom**: Single `execute_command` or `run_sql` tool with broad powers.

**Why it fails**: Model mis-invocation becomes full breach; descriptions can't constrain semantics.

**Instead**: Narrow tools with validated args; read/write split; role-based allowlists.

---

### 3. Context landfill

**Symptom**: Every turn resends full logs, HTML dumps, and 200K tokens "just in case."

**Why it fails**: Cost, latency, silent truncation, worse answers.

**Instead**: Prune, summarize, retrieve top-k, cache static prefix.

---

### 4. Multi-agent by default

**Symptom**: Supervisor + 5 workers for a FAQ bot.

**Why it fails**: Coordination bugs, 5× cost, exam wrong on simplicity stems.

**Instead**: Router + single agent; add workers only when parallel domains prove needed.

---

### 5. Prose orchestration

**Symptom**: Model says "I will now call the database" without `tool_use` block.

**Why it fails**: No guaranteed execution; tests can't assert behavior.

**Instead**: Proper agent loop with tools and `stop_reason` handling.

---

### 6. JSON in free text

**Symptom**: "Respond with valid JSON only" without schema enforcement.

**Why it fails**: Markdown fences, trailing commas, partial JSON break pipelines.

**Instead**: Tool use / structured output + schema validation + capped retry.

---

### 7. Cache-hostile prompts

**Symptom**: Dynamic dates/user IDs in system prompt before tools block.

**Why it fails**: Zero cache hits; pay full input every turn.

**Instead**: Static prefix + cache breakpoint; dynamic suffix only.

---

### 8. Eval-free prompt tuning

**Symptom**: Tweaking prompts in prod based on one chat thread.

**Why it fails**: Regressions undetected; overfit to anecdote.

**Instead**: Golden eval suite; deploy with canary and rollback.

---

### 9. Orphan tool results

**Symptom**: Custom history format dropping `tool_use_id` pairing.

**Why it fails**: API errors or model confusion next turn.

**Instead**: Use SDK/helpers; contract test history builder.

---

### 10. Monitoring HTTP 200 only

**Symptom**: Dashboard green while wrong JSON and failed tasks soar.

**Why it fails**: Semantic failures invisible.

**Instead**: Task success metrics, guardrail triggers, eval trends.

---

### 11. Subagent memory myth

**Symptom**: Worker "should know" what user said 10 turns ago without injection.

**Why it fails**: Isolated context by design.

**Instead**: Supervisor passes explicit brief + constraints.

---

### 12. Reflection without grounding

**Symptom**: Three "check your work" passes with no tools or rubric.

**Why it fails**: Polished wrong answers; 3× latency.

**Instead**: Tool verification, schema, single critic with checklist.

## Exam Notes

- Distractors often **are** these anti-patterns stated confidently
- Simplest production fix usually reverses one anti-pattern, not full rewrite
- Pair recognition with [Exam Strategy](../00-getting-started/exam-strategy.md) deterministic rule

## Production Notes

Run architecture reviews with anti-pattern checklist; block launch on **god tool** and **prompt-only security** for tier-2+ systems.

## Common Mistakes

- Fixing anti-pattern with another anti-pattern (e.g., Opus everywhere instead of pruning)
- Copying demo repos that embed anti-patterns for brevity

## See Also

- [Do's](./dos.md)
- [Don'ts](./donts.md)
- [Production Patterns](../10-design-patterns/production-patterns.md)
- [Guardrails](../08-production/guardrails.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
