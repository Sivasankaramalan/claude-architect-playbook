# Medium Flashcards

Q: Why do subagents miss prior findings?
A: They do not inherit coordinator context; context must be passed explicitly.

Q: What is the fix for wrong tool selection?
A: Improve tool names, descriptions, inputs, outputs, and boundaries.

Q: When should `tool_choice: any` be used?
A: When Claude must call some tool but can choose which one, such as mixed document routing.

Q: Why is same-session self-review weak?
A: The model is biased by its own generation context.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
