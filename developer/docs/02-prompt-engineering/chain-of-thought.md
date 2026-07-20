# Chain of Thought

When and how to elicit step-by-step reasoning from Claude—and when **not** to use chain-of-thought (CoT) in production or on the exam.

> **Verify on official docs:** Extended thinking and CoT-related features vary by model. See [extended thinking](https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking) and prompting guides for current support.

**CCDV F domains:** Prompt and Context Engineering · Model Selection and Optimization

## Learning Objectives

After this page, you should be able to:

- Define chain-of-thought prompting and how it differs from **extended thinking**
- Identify tasks that benefit from visible reasoning steps vs those that don't
- Apply CoT safely without leaking sensitive reasoning to end users
- Combine CoT with [prompt chaining](./prompt-chaining.md) and [structured outputs](./structured-prompts.md)
- Choose the right exam answer when CoT is a distractor

## Key Ideas

### What Chain of Thought Is

**Chain-of-thought prompting** asks Claude to show intermediate reasoning before the final answer:

```
Q: If a ticket costs $15 and buy 3 get 1 free, how much for 10 tickets?

A: Let me work step by step.
- Each group of 4 tickets costs 3 × $15 = $45
- 10 tickets = 2 groups of 4 (8 tickets) + 2 extra
- Cost = 2 × $45 + 2 × $15 = $90 + $30 = $120
Final answer: $120
```

CoT improves accuracy on multi-step logic, math word problems, and complex conditional policies—when the model might skip steps silently.

### CoT vs Extended Thinking

| Aspect | Prompted CoT (visible text) | Extended thinking (API feature) |
| --- | --- | --- |
| **Where reasoning lives** | In output text you prompt for | In dedicated thinking blocks/tokens |
| **User visibility** | Unless you strip it, users see it | Often hidden; configurable |
| **Cost** | Billed as output tokens | Thinking tokens billed separately |
| **Best for** | Quick prompt tweak, moderate tasks | Hard reasoning/coding on supported models |
| **Exam cue** | "Ask model to explain steps" | "Enable extended thinking for complex proof" |

Do not conflate them—exam options may list both for different scenarios.

### When CoT Helps

| Task type | CoT benefit |
| --- | --- |
| Multi-step arithmetic / logic | High |
| Policy application with exceptions | High |
| Debugging code paths | High |
| Simple entity extraction | Low (adds tokens, little gain) |
| Latency-sensitive FAQ | Low |
| Already using forced JSON tool | Often unnecessary in same step |

### Prompt Patterns

**Basic explicit CoT:**

```xml
<instructions>
Solve the problem. Think step by step in a <reasoning> section,
then give the final answer in <answer>.
</instructions>
```

**Structured CoT + parse:**

```xml
<output_format>
<reasoning>...</reasoning>
<answer>...</answer>
</output_format>
```

**Zero-shot CoT trigger:** appending "Let's think step by step" helps on some tasks—less control than structured tags.

### Hiding Reasoning from Users

Production chatbots often should **not** show internal reasoning.

Patterns:

1. **Two-step chain:** Step 1 CoT (logged internally); Step 2 polish answer for user
2. **Extended thinking:** reasoning off main channel (model-dependent)
3. **Strip tags server-side:** parse `<answer>` only before returning JSON to client

Never expose chain-of-thought that includes **secrets**, **PII**, or **tool credentials** echoed from context.

### CoT and Tools

Agents doing math, date logic, or database queries should often **use tools** instead of pure CoT:

| Approach | Prefer when |
| --- | --- |
| CoT | Small logical puzzles in provided text |
| Calculator/code tool | Precise arithmetic, large datasets |
| Retrieval tool | Facts not in context |

Exams: **"Guarantee correct balance calculation on 10k rows"** → tool/code—not CoT prose.

### CoT + Structured Output

For hard extraction with rules:

```
Step 1: Reason about which clause applies (CoT, internal)
Step 2: Emit JSON via tool/schema (public)
```

Single-message "think then JSON" works for moderate complexity; chaining reduces JSON syntax errors after long reasoning.

## Exam Notes

- CoT improves **reasoning visibility**, not **factual grounding**—still need retrieval/tools for live data.
- **Latency-sensitive** scenarios disfavor long visible CoT—prefer smaller model + tool or extended thinking selectively.
- **Security questions** may reject CoT that echoes untrusted instructions—fix injection, don't add more reasoning.
- If only accuracy matters and model supports it, **extended thinking** may beat hand-written CoT prompts.
- Simple classification with clear labels → **no CoT** (waste of tokens).

## Production Notes

- Log reasoning blocks internally for support/debug; redact before analytics export.
- Monitor output token growth from CoT—cost scales with visible steps.
- A/B test: CoT vs tools on golden sets; many "reasoning" tasks are really **data access** tasks.
- Set `max_tokens` high enough for reasoning + answer—truncated CoT yields wrong finals (`stop_reason: max_tokens`).
- For regulated domains, check whether showing reasoning to users creates compliance issues.

## Common Mistakes

- Using CoT for every API call by default
- Showing raw CoT with sensitive context to end users
- Expecting CoT to replace calculator/DB tools on numeric workloads
- Truncated answers because reasoning consumed all `max_tokens`
- Parsing free-text CoT with fragile regex instead of structured tags or separate steps
- Choosing CoT when exam scenario demands **JSON Schema guarantee**

## See Also

- [Prompting](./prompting.md)
- [Prompt Chaining](./prompt-chaining.md)
- [Structured Prompts](./structured-prompts.md)
- [Claude Models](../01-foundations/claude-models.md)
- [Tool Use](../04-agent-engineering/tool-use.md)
