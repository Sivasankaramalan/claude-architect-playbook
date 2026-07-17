# Parallel Execution

Independent subagents should be spawned in parallel when their tasks do not depend on each other. In Claude-style workflows, this means emitting multiple Task calls in the same coordinator turn rather than spawning sequentially.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
