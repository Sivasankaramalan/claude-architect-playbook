# Domain 3 Claude Code Summary

Focus: Claude Code configuration, memory hierarchy, rules, skills, slash commands, plan mode, direct execution, hooks, permissions, and CI/CD workflows.

## What To Master

This domain tests whether you can configure Claude Code for repeatable team workflows rather than relying on one-off chat instructions.

## Configuration Hierarchy

Know where instructions live and who receives them:

- User-level memory is personal and not shared with teammates.
- Project-level memory is appropriate for repository-wide team conventions.
- Directory-level guidance is useful when subfolders follow different rules.
- Modular rule files help avoid one large, hard-to-maintain instruction file.

If a new teammate does not receive important behavior after cloning a repo, the likely issue is that guidance was stored in a personal user-level location rather than the project.

## Skills, Rules, And Slash Commands

- Use rules for persistent project conventions.
- Use skills for reusable task-specific workflows that load when relevant.
- Use slash commands for explicit workflows the user chooses to invoke.

The exam often tests when to use each mechanism, not just what they are called.

## Plan Mode Versus Direct Execution

Use plan mode for complex, risky, architectural, multi-file, or ambiguous tasks. Use direct execution for small, clear, low-risk edits where the path is obvious.

## Claude Code In CI/CD

CI workflows need structured, actionable, low-noise output. Good CI prompts define:

- Review scope.
- Severity criteria.
- Output format.
- What to ignore.
- When to fail versus when to comment.

False positives are a production problem. If developers stop trusting automated review, even valid findings lose value.

## Common Exam Traps

- Putting team instructions in user-level configuration.
- Using direct execution for an ambiguous large refactor.
- Treating skills, rules, and slash commands as interchangeable.
- Producing verbose CI feedback with unclear severity.
- Asking for "better review" instead of defining explicit review criteria.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. All rights reserved.
