# Prompt Debugging

Systematic methods to diagnose bad Claude outputs: distinguish prompt bugs from integration bugs, instrument requests, and iterate without guesswork.

> **Verify on official docs:** Debugging production LLM apps also spans [Messages API errors](../03-api/error-handling.md) and observability guides on [docs.anthropic.com](https://docs.anthropic.com/).

**CCDV F domains:** Prompt and Context Engineering · Eval, Testing, and Debugging · Applications and Integration

## Learning Objectives

After this page, you should be able to:

- Classify failures: prompt, context, tool, model, or infrastructure
- Use API response fields (`stop_reason`, `usage`, content blocks) as debug signals
- Apply a structured iteration loop for prompt fixes
- Know when to change prompts vs API parameters vs architecture
- Recognize exam scenarios pointing to specific root causes

## Key Ideas

### Failure Taxonomy

| Symptom | Likely layer | First check |
| --- | --- | --- |
| Wrong JSON shape | Prompt / structure | Schema, tools, examples |
| Ignores instructions | Prompt / context | Contradictions, buried rules, injection |
| Stops mid-answer | API params | `stop_reason: max_tokens` |
| Empty or tool-only response | Agent integration | `tool_use` handling |
| Outdated facts | Grounding | Retrieval/tools, not prompt adjectives |
| Intermittent quality | Sampling / model | Temperature, model tier |
| 429 / 529 errors | Infrastructure | Retries, rate limits |
| Identical bug after "fix" | History | Are you resending old assistant turns? |

### The Debug Loop

```
1. Reproduce with frozen input (save messages JSON)
2. Read full response: stop_reason, usage, content blocks
3. Hypothesis: one variable category (prompt OR tools OR params)
4. Change one thing; re-run
5. Compare against golden expected output
6. Promote fix to eval suite
```

Avoid changing model, prompt, tools, and temperature simultaneously—you won't know what worked.

### API Signals to Inspect

**`stop_reason`**

| Value | Debug action |
| --- | --- |
| `max_tokens` | Increase `max_tokens`; shorten reasoning; chain steps |
| `end_turn` | Output complete—bug is content quality not truncation |
| `tool_use` | Expected in agents—ensure client executes and continues |
| `stop_sequence` | Verify intentional stop string not triggered early |

**`usage`**

- Sudden `input_tokens` spike → duplicate history, wrong template, missing cache
- High `output_tokens` with rambling → tighten prompt or lower temperature

**Content blocks**

- Unexpected `tool_use` → tool descriptions too broad or `tool_choice` mis-set
- Text + tool mix → parse both; don't drop tool blocks in UI layer

### Prompt-Specific Checks

| Check | Question |
| --- | --- |
| **Contradictions** | Do examples disagree with rules? |
| **Priority** | Does later user message override system policy? |
| **Specificity** | Is output format ambiguous? |
| **Placement** | Are instructions after long doc where model loses focus? |
| **Delimiters** | Is user data clearly separated ([XML tags](./xml-tags.md))? |
| **Few-shot drift** | Are examples still valid policy ([few-shot](./few-shot.md))? |

### Integration-Specific Checks

Common developer bugs misdiagnosed as "bad prompt":

```
❌ Tool result never appended before next API call
❌ Wrong tool_use_id in tool_result block
❌ Assistant message omitted from history
❌ System prompt only sent on first turn
❌ Streaming parser drops final tool block
❌ JSON parsed before stream completes
```

Golden rule from CCDV F: **`stop_reason: tool_use` → append tool_result → recall API**.

### Minimal Reproduction

Strip to smallest `messages` array that fails:

1. Remove old history—still fails?
2. Remove tools—still fails?
3. Swap user content for short synthetic input—still fails?

If failure disappears when history removed → **context/history bug**, not model regression.

### Comparison Testing

| Method | Purpose |
| --- | --- |
| **A/B prompts** | Which instruction variant wins on eval set |
| **Model swap** | Opus vs Sonnet—is task beyond tier? |
| **Fixed seed** | Not fully deterministic—use eval aggregates |
| **Regression suite** | 20–50 golden cases in CI |

See [evaluations](../08-production/evaluations.md) and [testing](../08-production/testing.md).

### Debugging Strategies by Task

**Classification wrong labels**

- Add [few-shot](./few-shot.md) borderline cases
- Force structured output tool
- Check class imbalance in examples

**Agent calls wrong tool**

- Narrow tool descriptions; remove unused tools
- Add negative examples in tool policy
- Adjust `tool_choice`

**Long doc Q&A misses answer**

- Move question after document
- Ask for quotes/citations
- Chunk + RAG instead of single paste

**Prompt injection**

- Separate untrusted content; system-level refusal rules
- Never execute instructions from user documents
- See [security](../11-best-practices/security.md)

### Logging for Postmortems

Log (with redaction):

- Request ID / `response.id`
- Model, `stop_reason`, token usage
- Prompt version hash
- Tool names invoked
- Latency per hop in [chains](./prompt-chaining.md)

Avoid logging full PII payloads to third-party analytics.

### When Not to Prompt-Debug

Switch approach when:

- Repeated `max_tokens` → parameter/architecture
- Needs live data → tools/retrieval
- Needs deterministic math → code execution
- Needs guaranteed schema → structured outputs
- Cost/latency SLA failed → model routing + caching

Exams often offer a **simple API fix** disguised among heavy rewrites.

## Exam Notes

- Truncated answer → **`max_tokens` / `stop_reason`**, not "use Opus" by default.
- Agent stops after first tool message → **missing tool_result in history**.
- Model "forgets" earlier turn → **history not resent**—not prompt wording.
- JSON invalid in production → **structured output/tool**, not "add please validate JSON."
- Intermittent wrong policy → contradictory **few-shot** examples.
- Debugging question with logs showing `cache_read_input_tokens: 0` always → prefix never matches (volatile prompt).

## Production Notes

- Build internal "replay request" admin tool from stored JSON.
- Alert on rising `stop_reason: max_tokens` rate.
- Track prompt version in `metadata` for correlation.
- Run weekly eval regression on release candidates.
- Document known model behavioral quirks per snapshot after upgrades.

## Common Mistakes

- Rewriting entire prompt before checking `stop_reason` and history
- Fixing streaming parser while tool blocks never arrive
- Adding "IMPORTANT:" caps lock instead of structural fix
- No golden tests—every fix re-breaks last month's case
- Debugging in production without sampling/redaction
- Assuming Skilljar-style trick prompts—CCDV F tests developer mechanics

## See Also

- [Prompting](./prompting.md)
- [Structured Prompts](./structured-prompts.md)
- [Prompt Chaining](./prompt-chaining.md)
- [Error Handling](../03-api/error-handling.md)
- [Evaluations](../08-production/evaluations.md)
- [Debugging](../08-production/debugging.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
