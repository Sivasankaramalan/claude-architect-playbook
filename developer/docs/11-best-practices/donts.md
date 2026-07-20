# Don'ts

Anti-practices that cause **exam wrong answers**, production incidents, and runaway cost in Claude applications. Pair with [Do's](./dos.md) and [Checklist](./checklist.md).

## Learning Objectives

After this page, you should be able to:

- Recognize common Claude developer mistakes in scenario questions
- Avoid prompt-only solutions when programmatic controls exist
- Identify security, context, and agent-loop don'ts quickly under time pressure

## Key Ideas

### API and agent loop

- **Don't** ignore `stop_reason` or continue looping after `end_turn`.
- **Don't** call the model again without appending `tool_result` for each `tool_use`.
- **Don't** retry 400-class client errors indefinitely—they won't succeed without a fix.
- **Don't** assume streaming reduces total generation time for long outputs (only TTFT UX).
- **Don't** use synchronous Messages API for massive overnight bulk—use Batches.

### Prompts and context

- **Don't** put secrets, API keys, or raw PII in system prompts or logs.
- **Don't** merge untrusted RAG/web content into the system prompt as instructions.
- **Don't** send entire chat UI history including failed attempts without curation.
- **Don't** rely on "remember X forever" in prose—persist facts externally.
- **Don't** use dynamic timestamps in cached prefix blocks (breaks prompt caching).

### Tools and MCP

- **Don't** expose one mega-tool that can do everything (shell + email + DB).
- **Don't** return raw stack traces or internal paths to the model/user.
- **Don't** ship overlapping tool names/descriptions that confuse routing.
- **Don't** grant MCP servers production admin credentials by default.
- **Don't** skip server-side validation because "the model will get it right."

### Architecture and agents

- **Don't** default to multi-agent/supervisor for every problem—single loop often wins.
- **Don't** assume subagents inherit coordinator conversation automatically.
- **Don't** run unbounded agent loops without max turns and cost caps.
- **Don't** parallelize dependent writes (race conditions).
- **Don't** choose Opus for every step to mask prompt/tool bugs.

### Safety and compliance

- **Don't** treat model refusal as your only app security layer.
- **Don't** auto-execute payments, deletes, or permission changes on model intent alone.
- **Don't** log full prompts in production without redaction and retention policy.
- **Don't** bypass hooks/permissions in CI "just to make tests pass."

### Testing and deployment

- **Don't** ship prompt changes without regression evals or staging validation.
- **Don't** deploy tool schema changes out of sync with backend handlers.
- **Don't** share one production API key across all environments.
- **Don't** evaluate only happy paths—failure modes need tests too.

## Exam Notes

- Wrong answer often = one of these don'ts dressed as a plausible shortcut
- "Tell the model to…" for security → almost always weaker than enforcement option
- Fancy architecture don't when stem describes simple tool lookup
- "Log everything" don't when privacy/production stem

## Production Notes

Use don'ts as **code review labels**. If a PR triggers three or more, escalate design review before merge.

## Common Mistakes

- Memorizing don'ts without matching to **positive alternative** (see Do's)
- Ignoring don'ts in "internal only" tools that later face customers
- Confusing **don't use multi-agent** with **never use multi-agent**—use when justified

## See Also

- [Do's](./dos.md)
- [Anti-Patterns](./anti-patterns.md)
- [Security](./security.md)
- [Safety](../08-production/safety.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
