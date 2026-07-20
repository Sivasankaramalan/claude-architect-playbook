# Domain 4 Exam Patterns

- Guaranteed structure: `tool_use` + `tool_choice`, not prompt-only JSON.
- Fabricated absent values: make field nullable and optional.
- Rare enum value: add `other` plus detail field.
- Ambiguous value: add `unclear`.
- Invalid output retry: append specific field-level errors.
- High false positives: add explicit criteria or disable noisy category while improving prompts.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
