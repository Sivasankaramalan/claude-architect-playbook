# Coordinator Pattern

Use hub-and-spoke architecture. The coordinator selects subagents, passes context, receives outputs, handles recovery, and synthesizes final responses. Subagents never call each other directly.

## Key Failure Mode

If synthesis lacks citations even though search produced them, the coordinator probably failed to pass source metadata.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
