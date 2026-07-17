# My Mock Questions

## Mock 1

Scenario: A refund agent sometimes processes refunds before identity verification even though the prompt says verification is mandatory. What is the best fix?

A. Add stronger wording to the prompt.
B. Add few-shot examples.
C. Add a programmatic prerequisite gate that blocks refund execution until verified customer ID exists.
D. Ask the model to self-report confidence.

Answer: C. Mandatory financial workflow ordering requires deterministic enforcement.

## Mock 2

Scenario: An invoice extractor fabricates `PO-UNKNOWN` because `po_number` is required but many invoices do not include a PO. What is the best fix?

Answer: Make `po_number` optional and nullable with a description that null means absent.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
