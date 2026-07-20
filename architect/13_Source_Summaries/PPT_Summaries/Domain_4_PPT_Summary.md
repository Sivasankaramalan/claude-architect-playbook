# Domain 4 PPT Summary

Focus: explicit criteria, few-shot prompting, `tool_use`, JSON schema, validation retry, evaluator-optimizer, Batch API, and independent review.

## What To Master

Domain 4 tests whether you can turn probabilistic text generation into reliable structured workflows. The most important distinction is format validity versus semantic correctness.

## Prompt Design

Good prompts define:

- Role and task.
- Explicit report criteria.
- Edge-case behavior.
- Examples for ambiguous cases.
- Required output structure.

Vague instructions like "be conservative" are weak. Concrete criteria such as "flag comments only when the claimed behavior contradicts actual code behavior" are much stronger.

## Structured Output

For production structured output, prefer `tool_use` with a JSON schema. A prompt that says "return JSON" can still produce markdown, commentary, missing fields, or malformed JSON.

Remember the `tool_choice` modes:

- `auto`: Claude may choose whether to use a tool.
- `any`: Claude must use some tool.
- `tool`: Claude must use one named tool.

Use forced tool choice when a single extraction schema must always be returned.

## Schema Design

- Optional missing data should be nullable, not fabricated.
- Use `other` plus a detail field when real values can fall outside an enum.
- Use `unclear` when the source is ambiguous.
- JSON schema validates structure, not business truth.

## Validation And Review

Use retry-with-specific-feedback for field-level errors. Tell the model exactly what failed, such as invalid date format or a negative amount that must be positive.

Use independent review instances when correctness matters. Same-context self-review is useful but biased by the model's original answer.

## Common Exam Traps

- Choosing prompt-only JSON for guaranteed structure.
- Making absent values required.
- Treating valid JSON as semantically correct.
- Retrying when the source document lacks the information.
- Using same-session self-review for high-risk validation.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
