# Decision Tree

```text
Need guaranteed behavior?
  yes -> hook, gate, schema, permission check, CI rule, or forced tool_choice
  no  -> prompt criteria, few-shot examples, temperature, critique

Tool chosen incorrectly?
  yes -> review descriptions, names, inputs, outputs, boundaries

Subagent missed context?
  yes -> pass structured context explicitly with metadata

Output valid JSON but wrong value?
  yes -> semantic validation or independent review, not just JSON schema

Long conversation lost facts?
  yes -> persistent case facts block or external retrieval
```
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. All rights reserved.
