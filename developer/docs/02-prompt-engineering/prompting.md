# Prompting

Core prompt design principles for Claude: clarity, structure, role separation, and aligning instructions with how the Messages API delivers context.

> **Verify on official docs:** Prompting best practices and model-specific guidance are updated on [prompt engineering overview](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview) and related pages.

**CCDV F domains:** Prompt and Context Engineering · Applications and Integration

## Learning Objectives

After this page, you should be able to:

- Write clear, task-focused prompts with explicit inputs, constraints, and output expectations
- Separate **system** instructions from **user** content appropriately
- Apply Anthropic-recommended patterns: directness, examples, delimiters, and specificity
- Know when prompting alone is enough vs when you need tools, schemas, or code
- Debug weak outputs by fixing instruction structure—not only adding "please try harder"

## Key Ideas

### Prompt = Program in Natural Language

A production prompt specifies:

1. **Role / objective** — what Claude is trying to accomplish
2. **Context** — documents, history, facts Claude should use
3. **Constraints** — format, tone, length, things to avoid
4. **Success criteria** — how to know the task is done

Vague prompts produce vague outputs. Exams reward **specific, actionable** instructions.

### System vs User vs Assistant

| Layer | API field | Best for |
| --- | --- | --- |
| **System** | `system` parameter | Stable policies, persona, formatting rules, tool-use policy |
| **User** | `messages` with `role: user` | End-user text, retrieved docs, images, tool results |
| **Assistant** | `messages` with `role: assistant` | Prior turns you replay from history |

Put **durable rules** in `system`. Put **variable task data** in `user`. Do not bury critical safety or format rules only in user messages where they may be diluted by later user turns.

### Anthropic Prompting Principles

| Principle | Practice |
| --- | --- |
| **Be direct** | State the task first; avoid excessive preamble |
| **Be specific** | Define format, audience, length, and edge cases |
| **Use structure** | Headings, numbered steps, [XML tags](./xml-tags.md) |
| **Show desired behavior** | [Few-shot](./few-shot.md) examples for ambiguous formats |
| **Separate data from instructions** | Delimiters around user-provided content |
| **Tell Claude what TO DO** | Prefer "Return JSON with keys X, Y" over long don't lists |

### Instruction Hierarchy (Suggested Layout)

```xml
<role>You are a senior technical writer.</role>

<task>Summarize the release notes for an engineering audience.</task>

<constraints>
- Max 150 words
- Use bullet points
- Mention breaking changes first
</constraints>

<document>
{{user_pasted_release_notes}}
</document>
```

See [XML tags](./xml-tags.md) for why angle-bracket delimiters help Claude parse long prompts.

### Output Control Spectrum

From weakest to strongest enforcement:

| Approach | Reliability | Example use |
| --- | --- | --- |
| Natural language ask | Low | Creative brainstorming |
| Detailed format spec + examples | Medium | Consistent prose sections |
| XML / JSON template in prompt | Medium-high | Repeated report shapes |
| Tool with `input_schema` | High | Agent actions |
| Structured output / JSON Schema (API) | Highest | Production parsers |

Exam pattern: when the stem says **must**, **guarantee**, or **production**, pick **schema/tool/API** over prompt-only options.

### Temperature and Task Type

| Task | Typical sampling |
| --- | --- |
| Extraction, classification, JSON | Low temperature |
| Open-ended writing, ideation | Moderate temperature |

Prompt quality and sampling work together—low temperature does not fix ambiguous instructions.

### Long Context Prompting

When sending large documents:

- Put **questions/tasks at the end** (recency helps on long inputs)
- Quote or delimit **only relevant sections** when possible
- Ask Claude to **cite passages** from provided text to reduce hallucination
- For multi-doc, label each source ([XML tags](./xml-tags.md))

See [context window](../09-performance/context-window.md).

### Multimodal Prompting

When including images:

- State what to look for explicitly ("Compare the wiring diagram to the error photo")
- Combine vision with **tool use** when measurements or live data are needed
- Remember images consume tokens—see [vision](../03-api/vision.md)

### Prompt + Tools Together

Strong agent prompts explain:

- When to call which tool
- What to do if tool fails
- That Claude must not invent tool results

Tool definitions are part of the prompt budget—keep descriptions precise.

## Exam Notes

- Prefer **`system` for persistent rules**; user message for variable data.
- **Specific beats verbose**—adding fluff rarely fixes structural ambiguity.
- **Production JSON** questions → structured outputs/tools, not "only output JSON."
- If output ignores format, first check **unclear spec** vs **missing examples** before switching models.
- **Prompt injection** scenarios: never trust user content as instructions—separate untrusted data with delimiters and system-level policy ([security](../11-best-practices/security.md)).

## Production Notes

- Version prompts in git; tag deployments with prompt hash.
- A/B test prompt changes with eval sets—not gut feel alone.
- Externalize prompts for non-engineers only with review workflows.
- Log rendered prompts (redact PII) for failure replay.
- Combine prompts with **validation layers**—parse JSON defensively even with schemas.

## Common Mistakes

- One giant paragraph with no structure
- Contradictory instructions ("be brief" + "include every detail")
- Putting system-level policy only in the latest user message
- Using few-shot examples that disagree with written rules
- Expecting prompt-only guarantees for machine-parseable output
- Changing five variables at once when debugging—can't attribute fixes

## See Also

- [Structured Prompts](./structured-prompts.md)
- [XML Tags](./xml-tags.md)
- [Few-Shot Examples](./few-shot.md)
- [Chain of Thought](./chain-of-thought.md)
- [Prompt Chaining](./prompt-chaining.md)
- [Prompt Debugging](./prompt-debugging.md)
