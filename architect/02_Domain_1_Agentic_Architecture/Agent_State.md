# Agent State

Subagents do not share memory automatically. Each invocation receives only the prompt and context explicitly supplied. Resuming an old session can introduce stale tool results if files or systems changed.

## Options

- Resume: continue when prior context is still valid.
- Fork: explore divergent approaches from a shared baseline.
- Fresh start with summary: best when old tool results are stale.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
