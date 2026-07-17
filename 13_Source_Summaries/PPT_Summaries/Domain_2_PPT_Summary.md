# Domain 2 PPT Summary

Focus: tool descriptions, MCP architecture, structured errors, tool distribution, built-in tools, project vs user MCP configuration.

## What To Master

Domain 2 tests whether Claude can safely and reliably interact with external systems. The model is only as good as the tool interface you expose to it.

## Tool Design Principles

Good tools have:

- A narrow purpose.
- Clear names that do not overlap.
- Descriptions that explain when to use the tool and when not to use it.
- Typed inputs with validation.
- Predictable structured outputs.
- Structured error responses.

Tool descriptions are not decoration. They are the main signal Claude uses when deciding which tool to call.

## Structured Errors

Useful tool errors should tell the agent what happened and what recovery path is valid.

- Transient error: retry may work.
- Validation error: fix the input before trying again.
- Business-rule error: explain policy or choose a different allowed action.
- Permission error: escalate or request authorization.

Avoid generic messages like "operation failed." They force the model to guess.

## Tool Distribution

Do not give every agent every tool. Broad tool access increases wrong calls and reduces reliability. Scope tools by role and split generic tools into more precise tools when the model must choose between similar actions.

## MCP Configuration

Use project-level MCP configuration for shared team integrations. Use user-level configuration for personal debugging or local-only tools. Credentials should come from environment variables or secret managers, never committed files.

## Common Exam Traps

- Fixing wrong tool selection only with prompt examples while tool descriptions remain vague.
- Retrying non-retryable business errors.
- Returning unstructured error strings.
- Using one broad tool where two narrow tools would be safer.
- Putting shared configuration only in a personal user-level file.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
