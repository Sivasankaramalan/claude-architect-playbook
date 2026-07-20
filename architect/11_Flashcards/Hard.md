# Hard Flashcards

Q: A synthesis agent produces unsourced claims although upstream agents had sources. What is the likely root cause?
A: Coordinator failed to pass structured source metadata to synthesis.

Q: A transfer occurs without AML check 5% of the time despite prompts. Best fix?
A: Tool call interception or prerequisite gate blocking transfer until AML pass.

Q: A 14-file review misses issues in later files and inconsistently flags repeated patterns. Best architecture?
A: Per-file local passes plus cross-file integration pass.

Q: Tool use returns valid JSON but totals do not reconcile. What is missing?
A: Semantic/business validation, possibly retry with specific feedback or conflict fields.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
