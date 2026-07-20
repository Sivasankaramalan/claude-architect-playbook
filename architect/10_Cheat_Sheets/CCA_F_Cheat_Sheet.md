# CCA-F Cheat Sheet

Quick public reference for Claude Certified Architect - Foundations preparation.

## Overview

Use this cheat sheet as a short orientation guide before going deeper into the domain folders. It collects high-level study reminders, Claude concepts, and public-safe resource links.

## Public Resources

These are public/community resources. Verify current exam details against official Anthropic sources before booking or publishing study claims.

- [Reddit CCAF discussion](https://www.reddit.com/r/ClaudeAI/comments/1s34iyl/ccaf_the_real_exam_has_more_scenarios_than_the/)
- [Claude Certification Guide](https://claudecertificationguide.com)
- [CertSafari Anthropic page](https://www.certsafari.com/anthropic)
- [Claude Certification Guide mock exam](https://claudecertificationguide.com/mock-exam)
- [WeAreCommunity pass-the-exam talk](https://wearecommunity.io/events/how-to-pass-the-claude-certified-architect-exam/talks/105397)
- [YouTube CCAF playlist](https://youtube.com/playlist?list=PLmiDJB5zE0KGwFtDCSqHfQG7SV6VeQ_31&si=Jtul4ugexv_nPZMI)
- [WeAreCommunity CCAF talk](https://wearecommunity.io/events/719-1000/talks/108755)
- [Practitioner's guide article](https://medium.com/@tselva88/claude-certified-architect-foundations-a-practitioners-guide-to-clearing-the-exam-959ed39262f7)
- [Claude Certification Guide progress page](https://claudecertificationguide.com/learn/progress)

## Readiness Checklist

Do not take the exam until you can check every box:

- [ ] I completed the official learning path and relevant Anthropic Academy courses.
- [ ] I read the official exam guide end to end.
- [ ] I understand the exam structure, domain weights, and six scenarios.
- [ ] I practiced scenario-based questions and can explain why wrong options are wrong.
- [ ] I understand current retake, attempt, and scheduling policies from official sources.
- [ ] I can explain the difference between prompt guidance and programmatic enforcement.

## AI Fluency

AI fluency is not only about prompting. It combines:

- Effective use: choose the right task and workflow.
- Efficient use: reduce waste, retries, and irrelevant context.
- Ethical use: respect privacy, safety, attribution, and policy.
- Safe use: add review, escalation, and deterministic controls where needed.

## 4D Framework

- Delegation: decide what to hand to AI and what humans or code must own.
- Description: provide context, constraints, examples, and desired output shape.
- Discernment: evaluate whether the output is correct, complete, and safe.
- Diligence: use AI responsibly with verification, provenance, and review.

## Interaction Modes

Claude and AI systems can support three broad modes:

- Automation: the system performs a defined task end to end.
- Augmentation: the system assists a human who remains actively involved.
- Agency: the system plans, uses tools, tracks state, and acts across multiple steps.

Exam questions often test whether the architecture matches the risk level of the interaction mode.

## Prompt Engineering Basics

Strong prompts usually include:

- Context: what the model needs to know.
- Role: the perspective or expertise to apply.
- Task: the specific job to complete.
- Examples: especially for ambiguous or edge cases.
- Output format: structure, schema, length, and style.
- Step breakdown: useful for complex tasks.
- Thinking space: instructions to analyze before final output, when appropriate.

Prompting improves probability. It does not guarantee high-risk behavior.

## Discernment

Discernment means judging the output rather than accepting it blindly.

- Check factual correctness.
- Check whether the answer follows constraints.
- Check whether important edge cases were missed.
- Provide feedback and correction.
- Use independent review when self-review is not enough.

## Diligence

Diligence is responsible AI practice:

- Do not publish private or copied training material.
- Do not expose secrets, credentials, internal URLs, or customer data.
- Preserve source attribution and provenance.
- Escalate when the system lacks authority or evidence.
- Use deterministic controls for money, security, compliance, and permissions.

## Claude 101

### Skills

Skills are folders of instructions, scripts, and resources that Claude can load dynamically to improve performance on specialized tasks.

- Anthropic-provided skills are maintained by Anthropic for supported use cases.
- Custom skills are created by users or organizations for specialized workflows and domain-specific tasks.

Skills are best for repeatable workflows with a consistent methodology.

### Projects vs Skills

| Concept | Projects | Skills |
| --- | --- | --- |
| Purpose | Store knowledge Claude can reference | Define processes Claude can execute |
| Best for | Long-term context, reference material, team collaboration | Repeatable workflows, multi-step tasks, consistent methodology |
| Example | Customer hub, research workspace, reference collection | Blog drafting, PR review checklist, PDF creation workflow |
| Persistence | Knowledge remains available across project chats | Instructions load when the skill is invoked or matched |

## Claude Code Customization

Claude Code has several ways to customize behavior.

### CLAUDE.md

`CLAUDE.md` files load persistent instructions into conversations. Use them for conventions that should always apply, such as coding standards, test expectations, or repository-specific architecture notes.

### Claude Code Skills

Skills load on demand when they match the task. They avoid filling every session with instructions that are only relevant sometimes.

### Slash Commands

Slash commands are explicit user-triggered workflows. Use them when the user should intentionally choose the workflow.

### Hooks

Hooks run commands before or after Claude attempts tool use. Use hooks for enforcement, validation, normalization, logging, or guardrails around tool execution.

## MCP

Model Context Protocol (MCP) is a communication layer that provides Claude with context and tools without requiring every integration to be custom-built from scratch.

MCP helps with:

- Tool discovery.
- Resource access.
- Prompt reuse.
- Enterprise and local system integration.
- Clear boundaries between model reasoning and external actions.

## RAG

Retrieval-augmented generation helps Claude answer using external knowledge. Common chunking strategies:

- Chunk by size: simple and predictable.
- Chunk by structure: align chunks with headings, sections, or records.
- Chunk by semantics: group meaningfully related content.

For exam scenarios, remember that retrieval should preserve provenance and return relevant evidence, not dump everything into context.

## Public Repo Note

This cheat sheet removes private organization links and employer-specific readiness items. Keep future additions public-safe: no real exam questions, private training links, copied course content, credentials, or internal URLs.
---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
