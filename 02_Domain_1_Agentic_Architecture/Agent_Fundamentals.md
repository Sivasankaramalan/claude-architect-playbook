# Agent Fundamentals

An agentic system repeatedly calls Claude, inspects `stop_reason`, executes requested tools, appends tool results, and calls Claude again until `end_turn`. The model decides the next action, but the application owns loop control and safety boundaries.

## Exam Anchor

Do not parse response text to determine completion. Text may appear alongside a tool call. `stop_reason` is authoritative.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. All rights reserved.
