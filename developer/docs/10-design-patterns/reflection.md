# Reflection

The **reflection** pattern improves outputs by having the model (or a second pass) critique, verify, or refine an initial result before delivery. Common in **Agents and Workflows (14.7%)** and quality-focused production designs—balanced against latency and cost.

## Learning Objectives

After this page, you should be able to:

- Distinguish reflection from simple multi-turn chat
- Apply self-critique, critic model, and tool-verified reflection patterns
- Know when reflection helps vs adds cost without gain
- Combine reflection with evals and guardrails
- Pick exam options that use verification appropriately

## Key Ideas

### Reflection loop (conceptual)

```text
Draft → Critique → Revise → (optional) verify with tools → Final
```

Reflection adds **at least one extra model call**—use when quality or safety justifies cost.

### Pattern variants

**1. Self-reflection (same model)**

Prompt: "Review your answer for errors against criteria X; then output corrected version."

Cheap but model may repeat same mistake—limited independence.

**2. Critic model (second role/model)**

Generator (Sonnet) → Critic (Haiku or Sonnet with rubric) → Merge.

Critic focuses on checklist; better for code and policy compliance.

**3. Tool-grounded verification**

After draft, **deterministic tools** validate:

- Run tests / linter
- Query database for fact check
- Schema validate JSON

Strongest when facts are checkable—reflection prose alone can't verify SQL results.

**4. Human reflection**

Model draft → human edit → learn via eval cases—not automated reflection but production common.

### When reflection helps

- High-stakes summaries (medical, legal disclaimers—plus human)
- Code generation before merge
- Complex planning with explicit rubric
- Structured output that failed validation once

### When to skip

- Simple FAQ with retrieval citation already required
- Latency-sensitive chat with high baseline quality
- Tasks with strong tool-verified single pass

Exam: **simplest fix** — don't add critic loop if schema validation + retry suffices.

### Reflection vs agent loop

Agent loop runs until task done with tools. Reflection is a **quality pass**—may not need new tools, just revised text.

Can combine: agent completes → reflection pass → guardrails.

## Exam Notes

- Code wrong but plausible → **run tests tool** or **critic + fix**, not only "try again"
- JSON invalid once → **validation error feedback retry** before full reflection chain
- Cost-sensitive high volume → **skip reflection**; use routing + cache
- Factual answer → **tool verify**, not model self-critique alone
- Reflection without criteria → vague; rubric-based stronger

## Production Notes

### Reflection checklist

- [ ] Explicit rubric (what to check)
- [ ] Max one–two revise iterations (cap cost)
- [ ] Log draft vs final for eval
- [ ] Prefer tool verification when available
- [ ] Measure latency impact on P95

## Common Mistakes

- **Infinite refine loops** — no improvement, 3× cost
- **Reflection without grounding** — polished hallucinations
- **Same prompt twice** — no independent critic
- **Replacing guardrails with reflection** — still need schema enforcement
- **Reflection on every message** — unnecessary latency

## See Also

- [Guardrails](../08-production/guardrails.md)
- [Evaluations](../08-production/evaluations.md)
- [Chain of Thought](../02-prompt-engineering/chain-of-thought.md)
- [Testing](../08-production/testing.md)
- [Production Patterns](./production-patterns.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
