# Architecture Decision Framework

## Decision Flow

1. Is the problem caused by missing instructions? Add explicit criteria or few-shot examples.
2. Is the problem caused by ambiguous tool choice? Fix tool descriptions, names, and boundaries.
3. Is the problem caused by high-risk action ordering? Add programmatic prerequisite gates or hooks.
4. Is the problem caused by context loss? Persist structured facts or retrieve targeted history.
5. Is the problem caused by broad scope or attention dilution? Decompose into smaller passes.
6. Is the problem caused by absent source data? Return null, unclear, or escalate; do not retry blindly.
7. Is the problem caused by multi-source conflict? Preserve provenance and expose uncertainty.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
