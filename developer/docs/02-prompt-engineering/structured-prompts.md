# Structured Prompts

Designing prompts that produce **predictable, parseable outputs**—and knowing when to enforce structure with API features instead of instructions alone.

> **Verify on official docs:** Structured output capabilities and beta headers evolve. Check [structured outputs](https://docs.anthropic.com/en/docs/build-with-claude/structured-outputs) and tool-use docs for current syntax.

**CCDV F domains:** Prompt and Context Engineering · Applications and Integration

## Learning Objectives

After this page, you should be able to:

- Define output **contracts** (fields, types, enums) in prompts clearly
- Compare prompt-only structure vs **tools** vs **JSON Schema / structured outputs**
- Write prompts that pair with downstream parsers safely
- Recognize exam answers requiring **programmatic** enforcement
- Handle partial failures, markdown fences, and schema violations in production

## Key Ideas

### What "Structured" Means

Structured output is data your application can **parse without guessing**:

- JSON objects with fixed keys
- Enumerated labels (`severity: low | medium | high`)
- Tables with required columns
- Tool `input` objects matching JSON Schema

Unstructured prose is fine for humans; **machines need contracts**.

### Three Enforcement Layers

| Layer | Mechanism | Reliability |
| --- | --- | --- |
| **1. Prompt contract** | Describe JSON shape in natural language or template | Medium |
| **2. Tool use** | Force Claude to emit `tool_use` with `input_schema` | High |
| **3. Structured outputs API** | API validates against JSON Schema (beta/feature-flagged) | Highest |

Exams: if reliability is explicit, **layer 3 or 2** beats layer 1.

### Prompt-Only JSON Template Pattern

```xml
<instructions>
Extract invoice fields. Return ONLY valid JSON matching this schema:
{
  "vendor": string,
  "total_usd": number,
  "due_date": "YYYY-MM-DD" | null
}
No markdown fences. No commentary.
</instructions>

<invoice>
{{raw_text}}
</invoice>
```

Improvements that help:

- Repeat **ONLY JSON** once—not ten times
- Provide [one few-shot example](./few-shot.md) with valid JSON
- Set **low temperature**
- Still **validate in code**—models can drift

### Tool-Based Structure (Common Production Pattern)

Define a tool whose only purpose is structured extraction:

```json
{
  "name": "record_invoice",
  "description": "Record parsed invoice fields",
  "input_schema": {
    "type": "object",
    "properties": {
      "vendor": { "type": "string" },
      "total_usd": { "type": "number" },
      "due_date": { "type": ["string", "null"], "format": "date" }
    },
    "required": ["vendor", "total_usd"]
  }
}
```

Use `tool_choice` to force the tool when needed:

```json
"tool_choice": { "type": "tool", "name": "record_invoice" }
```

Response contains `tool_use` block—parse `input` JSON from API types, not free text.

### Structured Outputs (API-Level)

When available, pass a JSON Schema via documented beta headers/parameters. The API constrains generation to valid schema instances—best for pipelines that previously broke on malformed JSON.

Always confirm:

- Model support
- Required beta header strings
- Whether `$ref`, `enum`, and nested objects are supported as expected

### Schema Design Tips

| Tip | Why |
| --- | --- |
| Flatten when possible | Easier parsing and fewer null edge cases |
| Use `enum` for labels | Prevents synonym drift ("High" vs "high") |
| Prefer explicit `required` arrays | Missing fields fail fast in validation |
| Avoid ambiguous optional fields | Document defaults in prompt *and* schema |
| Keep string dates ISO-8601 | Consistent parsing |

### Combining Structure with Reasoning

For complex extraction:

1. Ask Claude to analyze in prose **or** use [chain-of-thought](./chain-of-thought.md) in a separate step
2. Emit final structured object in step 2 ([prompt chaining](./prompt-chaining.md))

Single-step "think then JSON" in one prompt works for moderate tasks; chaining reduces format corruption on hard inputs.

### Handling Markdown Wrappers

Models sometimes wrap JSON in ` ```json ` fences despite instructions. Production parsers should:

- Strip fences defensively
- Use structured outputs/tools to eliminate fences entirely
- Reject and retry with repair prompt only as fallback

## Exam Notes

- **"Guarantee valid JSON matching schema"** → structured outputs or forced tool—not "be careful."
- **`tool_choice: any`** vs **`auto`** vs forced tool—know when scenario requires deterministic tool call.
- Distinguish **tool `input_schema`** from response JSON in plain text—they're different code paths.
- If question mentions downstream **database insert**, reliability beats eloquent prose.
- Partial JSON due to **`max_tokens`** → raise limit or chain steps—not switch to Opus alone.

## Production Notes

- Validate with `ajv`, Pydantic, or equivalent even with API enforcement—defense in depth.
- Log schema validation failures with prompt version and input hash.
- Use structured outputs for ETL; use tools when Claude must **act** on structured data.
- Document nullable vs omitted fields in your API contract docs.
- Retry policy: fix-on-fail prompts consume extra tokens—cap retries.

## Common Mistakes

- Regex-parsing free-form Claude prose in production
- Giant nested schemas when a flat object suffices
- Forgetting `required` in JSON Schema—silent missing keys
- Mixing instructions and JSON example without delimiters—model copies wrong section
- Using forced tools but executing wrong tool name in handler
- Assuming structured outputs remove need for business-logic validation

## See Also

- [Prompting](./prompting.md)
- [XML Tags](./xml-tags.md)
- [Few-Shot Examples](./few-shot.md)
- [Prompt Chaining](./prompt-chaining.md)
- [Tool Use](../04-agent-engineering/tool-use.md)
- [Messages API](../03-api/messages-api.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
