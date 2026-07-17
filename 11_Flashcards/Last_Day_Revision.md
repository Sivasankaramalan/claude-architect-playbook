# Last Day Revision Flashcards

Q: Guarantee output shape?
A: `tool_use` with schema and appropriate `tool_choice`.

Q: Guarantee workflow ordering?
A: Programmatic gate or hook.

Q: Handle summarization loss of exact values?
A: Persistent case-facts block.

Q: Handle rare enum values?
A: `other` plus detail field.

Q: Handle ambiguous enum values?
A: `unclear`.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. All rights reserved.
