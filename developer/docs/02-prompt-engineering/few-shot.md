# Few-Shot Examples

Using input–output demonstrations in prompts to teach Claude formats, tone, edge cases, and task boundaries—without fine-tuning.

> **Verify on official docs:** Example placement and long-context behavior are covered in Anthropic's [prompt engineering](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview) guides.

**CCDV F domains:** Prompt and Context Engineering

## Learning Objectives

After this page, you should be able to:

- Design effective few-shot examples that match your output contract
- Decide how many examples to include given token budget and task complexity
- Place examples correctly relative to instructions and user input
- Avoid contradictory or biased demonstrations
- Know when few-shot is insufficient vs [structured outputs](./structured-prompts.md) or tools

## Key Ideas

### What Few-Shot Means

**Few-shot prompting** includes one or more completed examples of the task before the real input:

```
Example 1: Input → Output
Example 2: Input → Output
Now handle: Real input → ?
```

Claude generalizes the pattern—useful when describing the format in prose alone is ambiguous.

### Zero-Shot vs One-Shot vs Few-Shot

| Style | Examples | When |
| --- | ---: | --- |
| **Zero-shot** | 0 | Simple, well-defined tasks |
| **One-shot** | 1 | Clear format demo needed |
| **Few-shot** | 2–5+ | Nuanced labels, tone, edge cases |

More examples ≠ always better—conflicting examples hurt more than zero examples.

### Anatomy of a Good Example

Each example should demonstrate:

| Element | Guideline |
| --- | --- |
| **Representative input** | Realistic length and noise |
| **Correct output** | Exactly what your parser expects |
| **Edge handling** | Optional: show null/empty/unknown case |
| **Consistent format** | Same keys, tags, or headings every time |

Wrap sets in [XML tags](./xml-tags.md):

```xml
<examples>
  <example>
    <input>Refund request within 14 days, unused item</input>
    <output>{"decision": "approve", "reason": "within policy window"}</output>
  </example>
  <example>
    <input>Refund after 90 days</input>
    <output>{"decision": "deny", "reason": "outside 30-day policy"}</output>
  </example>
</examples>

<current_input>
{{ticket_text}}
</current_input>
```

### Example Count vs Token Cost

Examples consume **input tokens** on every request. Balance:

- **High-volume Haiku routes:** 1 tight example
- **Complex classification:** 3–5 diverse examples covering borderline cases
- **Cached system prompts:** examples in stable prefix → amortize with [prompt caching](./prompt-caching.md)

Remove outdated examples when product policy changes—silent drift causes wrong approvals.

### Diversity and Coverage

Cover:

- Typical case (happy path)
- Borderline case (exam favorites)
- Negative / "cannot answer" case if policy requires abstention

Avoid **near-duplicate** examples that waste tokens without teaching new behavior.

### Common Few-Shot Use Cases

| Use case | What examples teach |
| --- | --- |
| JSON field naming | Exact keys and types |
| Support tone | Empathy + brevity |
| Severity scoring | Consistent rubric |
| Extraction from messy OCR | Ignore headers/footers |
| Tool selection | When to search vs answer directly |

### Few-Shot vs Fine-Tuning

| Approach | Pros | Cons |
| --- | --- | --- |
| **Few-shot in prompt** | Instant, reversible, no training pipeline | Token cost, context limits |
| **Fine-tuning** (if offered) | Cheaper inference at scale | Data prep, maintenance, eval burden |

CCDV F focuses on **prompt + API** patterns—not training clusters.

### When Few-Shot Is Not Enough

Upgrade to stronger enforcement when:

- Downstream requires **100% schema compliance**
- Examples would need dozens of labels (token explosion)
- Security requires deterministic allowlists

→ Use [structured prompts](./structured-prompts.md), forced tools, or structured outputs API.

## Exam Notes

- Scenario: **"Model returns wrong JSON keys"** → add **consistent few-shot** *or* schema enforcement—if "production guarantee," pick schema/tool.
- **Contradictory examples** in option list → fix examples before changing model tier.
- Place **instructions before examples** or after? Anthropic often recommends instructions first, then examples, then live input—know both orders if question tests structure.
- Few-shot does **not** give Claude new factual knowledge about your private API—still need docs/tools.
- Bias risk: examples skew decisions—include balanced approve/deny samples in classification tasks.

## Production Notes

- Store examples in version-controlled templates; review with legal/compliance for customer-facing decisions.
- Automate checks that example outputs still validate against current JSON Schema.
- Rotate examples when metrics drift (sudden label imbalance).
- For multilingual apps, examples in target language outperform translated instructions alone.
- Monitor token growth when PMs "just add one more example."

## Common Mistakes

- Examples that violate written rules above them
- Output examples with markdown fences when parser expects raw JSON
- Too many similar examples crowding out actual user content
- Stale examples after policy change (model follows demos over text policy)
- Using few-shot for numeric accuracy on large datasets—use tools instead
- Including PII from real tickets in permanent examples

## See Also

- [Prompting](./prompting.md)
- [XML Tags](./xml-tags.md)
- [Structured Prompts](./structured-prompts.md)
- [Prompt Chaining](./prompt-chaining.md)
- [Prompt Debugging](./prompt-debugging.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
