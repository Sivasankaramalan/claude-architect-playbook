# Structured Data Extraction Walkthrough Summary

Covers Scenario 6: `tool_choice` modes, nullable fields, enum escape patterns, syntax vs semantic validation, retry with field-level feedback, prompt caching, Batch API, and structured extraction architecture.

## Architecture

A reliable extraction pipeline usually has these stages:

1. Classify or route the document if multiple schemas are possible.
2. Force the correct extraction tool when a schema is known.
3. Extract into a JSON schema with nullable optional fields.
4. Validate syntax and business semantics separately.
5. Retry with specific field-level feedback when correction is possible.
6. Escalate or mark uncertainty when the source lacks enough evidence.
7. Persist structured output with provenance.

## Schema Design Lessons

- Required fields are dangerous when the source may not contain the value.
- Use `null` for absent optional data.
- Use `other` plus a detail field for known values outside a fixed enum.
- Use `unclear` when the document is ambiguous.
- Include conflict fields when downstream systems need reconciliation, such as calculated total versus stated total.

## `tool_choice` Lessons

- `auto` is optional and should not be used when tool use must happen.
- `any` forces some tool call and is useful when Claude must choose among multiple document-specific tools.
- `tool` forces one named tool and is useful for a known schema.

## Validation Lessons

JSON schema ensures shape and type. It does not ensure the extracted value is true. Add semantic validation for totals, dates, category rules, and downstream invariants.

## Practice Exercise

Design an invoice extraction schema with:

- Vendor name.
- Invoice date.
- Due date.
- Optional purchase order number.
- Line items.
- Stated total.
- Calculated total.
- Conflict flag.
- Currency enum with `other` and detail.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. All rights reserved.
