# Domain 5 PPT Summary

Focus: context windows, attention placement, compression, summarization, structured errors, hallucination prevention, and reliability.

## What To Master

Domain 5 tests whether your architecture keeps the right information available at the right time. Large context windows help, but they do not remove attention, summarization, or stale-context problems.

## Context Risks

- Long conversations can bury important details.
- Summaries can lose exact amounts, dates, order IDs, and customer commitments.
- Verbose tool results can crowd out more important context.
- Resumed sessions can contain stale tool outputs after files or systems change.

## Preservation Strategies

Use a persistent facts block for critical details:

- Customer IDs.
- Order IDs.
- Dates and amounts.
- Policy decisions.
- Open issues.
- Commitments already made to the user.

Use summaries for narrative history, not for exact transactional facts.

## Attention Placement

Put durable rules and role constraints near the beginning. Put the current task, recent user request, and required output format near the end. Do not bury critical instructions in the middle of large context.

## Reliability Patterns

- Trim tool outputs before adding them to context.
- Retrieve targeted evidence instead of loading everything.
- Use multi-pass workflows for large codebases or large document sets.
- Preserve provenance when synthesizing multiple sources.
- Escalate when the system lacks authority, evidence, or policy clarity.

## Common Exam Traps

- Choosing a larger context window as the first fix.
- Relying on summarization to preserve exact numbers.
- Keeping full verbose tool outputs in every turn.
- Resuming a stale session after files changed without targeted re-reading.
- Producing confident synthesis without source provenance.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. All rights reserved.
