# XML Tags

Using XML-style tags to structure Claude prompts: separating instructions, data, examples, and tool context so the model parses long prompts reliably.

> **Verify on official docs:** Anthropic explicitly recommends XML tags in [use XML tags](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-xml-tags) guidance.

**CCDV F domains:** Prompt and Context Engineering

## Learning Objectives

After this page, you should be able to:

- Explain why XML tags improve clarity on long or multi-part prompts
- Choose tag names that describe content semantically
- Nest tags safely for documents, examples, and multi-file context
- Combine XML structure with system prompts and untrusted user input
- Avoid tag-related pitfalls in exams and production

## Key Ideas

### Why XML Tags?

Claude is trained to attend to **structured delimiters**. Tags help the model answer:

- What is instruction vs data?
- Where does one document end and another begin?
- Which example is canonical?

Especially valuable for:

- Long RAG contexts with multiple sources
- Multi-step instructions
- [Few-shot](./few-shot.md) example blocks
- Untrusted user content (prompt injection mitigation **partial**—not complete)

### Basic Pattern

```xml
<instructions>
Summarize the pull request for a non-technical PM.
Highlight user impact first.
</instructions>

<pr_diff>
{{diff_text}}
</pr_diff>

<output_format>
- 3 bullets max
- No jargon without explanation
</output_format>
```

Tag names are **semantic**, not magical—`<document>` vs `<doc>` both work if consistent.

### Common Tag Categories

| Tag type | Examples | Purpose |
| --- | --- | --- |
| **Task** | `<instructions>`, `<task>`, `<goal>` | What to do |
| **Data** | `<document>`, `<email>`, `<code>`, `<user_query>` | Untrusted or variable input |
| **Format** | `<output_format>`, `<schema>`, `<example_output>` | Shape of answer |
| **Examples** | `<example>`, `<examples>` | Few-shot demonstrations |
| **Context** | `<background>`, `<policy>`, `<glossary>` | Reference material |

### Nesting and Multiple Documents

```xml
<documents>
  <document index="1" source="policy.pdf">
    {{chunk_1}}
  </document>
  <document index="2" source="faq.md">
    {{chunk_2}}
  </document>
</documents>

<question>
{{user_question}}
</question>
```

Attributes like `index` and `source` help Claude cite the right document—useful for support bots and RAG.

### XML vs Markdown Headings

| Approach | Pros | Cons |
| --- | --- | --- |
| **XML tags** | Strong boundaries; great for user data islands | Verbose |
| **Markdown headings** | Readable in git diffs | Weaker delimiting for untrusted blobs |

Anthropic docs favor XML for **machine/user content separation**; markdown is fine for internal engineer-readable prompt templates.

### Pairing with System Prompts

Typical split:

- **`system`**: role, safety policy, global formatting rules
- **`user` message**: XML-wrapped task + documents

Do not duplicate conflicting rules across system and tagged sections.

### XML and Tool Use

Tool definitions live in API `tools` array—not XML. But prompts often use tags for **tool policy**:

```xml
<tool_policy>
Call search_docs before answering product questions.
If no relevant doc, say you don't know—do not guess.
</tool_policy>
```

Clear policy reduces wrong tool selection in agent loops.

### Prompt Injection Note

Wrapping user input:

```xml
<untrusted_user_content>
{{user_message}}
</untrusted_user_content>
```

helps Claude treat content as **data**, not instructions—but **does not replace** authorization, output validation, and system-level safety rules. See [security](../11-best-practices/security.md).

### Escaping and Raw Content

If user content contains `</document>`-like sequences:

- Sanitize or escape closing tags in untrusted input
- Use unique tag names (`<user_doc_7f3a>`) for high-risk pipelines
- Prefer storing user HTML as escaped text

## Exam Notes

- Questions about **organizing long prompts** or **separating instructions from data** → XML tags or equivalent clear delimiters.
- **Multiple documents in context** → nested `<documents>` with identifiers—not concatenation without labels.
- XML tags ≠ API XML protocol—still JSON Messages API underneath.
- Don't pick XML when question asks for **JSON Schema enforcement**—that's structured outputs/tools.

## Production Notes

- Standardize tag vocabulary across your prompt library (lint templates in CI).
- Keep tag names stable for [prompt caching](../02-prompt-engineering/prompt-caching.md)—changing tags breaks cache prefixes.
- Template engines (Jinja, etc.) should HTML/XML-escape user variables by default.
- For localization, keep tags English even if content is translated—consistent parsing.

## Common Mistakes

- Inconsistent tag names (`<doc>` vs `<document>`) across prompts
- Forgetting closing tags—models cope but quality can drop
- Putting instructions *inside* untrusted user XML island
- Over-nesting ten levels without reason—flat is often enough
- Assuming tags alone prevent prompt injection
- Changing delimiter strategy without re-running evals

## See Also

- [Prompting](./prompting.md)
- [Structured Prompts](./structured-prompts.md)
- [Few-Shot Examples](./few-shot.md)
- [Prompt Chaining](./prompt-chaining.md)
- [Security](../11-best-practices/security.md)
