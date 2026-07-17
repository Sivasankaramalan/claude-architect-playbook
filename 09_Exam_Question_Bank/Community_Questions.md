# Community Questions

This public repository intentionally does not include exam dumps, copied practice-bank content, or questions extracted from non-public materials.

Use this file to collect original community-created practice questions that teach the underlying architecture patterns without reproducing protected exam content.

## Contribution Rules

- Write original scenarios in your own words.
- Do not copy questions from real exams, paid practice tests, private training decks, employer materials, or certification dumps.
- Include an explanation of the principle, not just the answer.
- Prefer scenario-based questions that test architecture judgment.

## Example Question

Scenario: A support agent sometimes executes refunds before customer identity is verified, even though the system prompt says identity verification is required. What is the best fix?

A. Add stronger wording to the system prompt.
B. Add more examples showing correct refund behavior.
C. Add a programmatic prerequisite gate that blocks refund execution until verified customer identity exists.
D. Ask the model to report confidence before each refund.

Correct answer: C. Financial actions require deterministic enforcement. Prompt instructions and examples improve likelihood but do not guarantee ordering.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. All rights reserved.
