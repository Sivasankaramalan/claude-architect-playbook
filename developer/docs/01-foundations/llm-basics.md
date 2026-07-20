# LLM Basics

Core language-model concepts every Claude developer needs before writing API calls, designing prompts, or estimating cost. This page covers how LLMs behave in production—not deep ML theory.

> **Verify on official docs:** Model capabilities, context limits, and feature availability change. Check [Anthropic docs](https://docs.anthropic.com/) for current details.

**CCDV F domains:** Model Selection and Optimization · Prompt and Context Engineering · Applications and Integration

## Learning Objectives

After this page, you should be able to:

- Explain how an LLM turns text (tokens) into a probabilistic next-token prediction
- Distinguish **context window**, **max output tokens**, and **total billable usage**
- Describe why LLMs are **non-deterministic** and how `temperature` / `top_p` affect behavior
- Identify when a problem is a **model limitation** vs a **prompt / integration** issue
- Connect LLM behavior to API fields like `stop_reason`, `usage`, and message roles

## Key Ideas

### What an LLM Does

A large language model reads a sequence of **tokens** (subword units, not always whole words) and predicts the most likely continuation. Claude does not execute code, query databases, or browse the web unless your application gives it tools and returns results in the conversation.

```
Your app → [system + history + user message] → Claude → [text and/or tool_use blocks] → Your app
```

Each API call is largely **stateless**: Claude "remembers" prior turns only because **you resend conversation history** on every request.

### Tokens and Context

| Concept | Meaning | Developer impact |
| --- | --- | --- |
| **Input tokens** | Everything you send: system prompt, messages, tool definitions, images (converted to tokens) | Drives cost and latency; counts toward context limit |
| **Output tokens** | Text (and some structured blocks) Claude generates | Capped by `max_tokens`; also billed |
| **Context window** | Maximum tokens the model can consider in one request (input + output budget) | Long docs need chunking, retrieval, or summarization |
| **Total usage** | `usage.input_tokens` + `usage.output_tokens` in the response | Use for billing dashboards and optimization |

Rough rule of thumb: **1 token ≈ 4 characters** in English, but count with the [token counting API](https://docs.anthropic.com/en/docs/build-with-claude/token-counting) for accuracy.

### Roles and Message Structure

Claude's Messages API uses explicit roles:

| Role | Typical content |
| --- | --- |
| `system` | Instructions, persona, policies (often cached in production) |
| `user` | End-user input, documents, images, tool results |
| `assistant` | Prior model outputs, including `tool_use` blocks |

The model learns task behavior from **all** of this context—not from hidden internal memory between calls.

### Non-Determinism and Sampling

Claude samples from a probability distribution over next tokens. The same prompt can produce different answers unless you control randomness.

| Parameter | Effect |
| --- | --- |
| `temperature` | Higher → more varied; lower → more focused (often 0–1) |
| `top_p` | Nucleus sampling; alternative to temperature |
| `top_k` | Limits candidate tokens (model-dependent support) |

For **production tasks requiring consistency** (classification, extraction, structured JSON), prefer **low temperature** plus **schema / tool enforcement**, not prompt pleading alone.

### Capabilities vs Limitations

**Claude is strong at:** reasoning over text, following structured instructions, code generation, summarization, tool selection when schemas are clear.

**Claude cannot reliably:** guarantee factual freshness without retrieval, perform exact arithmetic on huge lists without tools, access private systems without integration, or remember past sessions unless you persist history.

Treat the model as a **reasoning layer** in your stack—not a database, cron job, or authorization service.

### Hallucinations and Grounding

A **hallucination** is confident but incorrect output. Mitigations developers use:

- Provide source documents in context (RAG)
- Require citations tied to provided text
- Use tools for live data (APIs, search, DB queries)
- Validate outputs programmatically (JSON Schema, business rules)

Exam questions often ask for the **most reliable production fix**—usually grounding + validation, not "prompt harder."

### Multimodal Basics

Vision-capable models accept **image blocks** inside `user` messages (base64 or URL, per docs). Images consume tokens. For document workflows, also consider **PDF support** and text extraction patterns described in the API docs.

### Extended Thinking (When Available)

Some models support **extended thinking**—internal reasoning tokens before the visible answer. This improves hard reasoning/coding tasks at higher latency and cost. Enable via model-specific request parameters documented for your chosen model; verify current model IDs before the exam.

## Exam Notes

- **Context window ≠ max_tokens.** Context is total capacity; `max_tokens` limits *output* only.
- If a scenario describes **forgetting earlier conversation**, the fix is usually **your app not resending history**—not a model bug.
- **Non-determinism** is expected; exams favor API-level controls (temperature, schemas, tools) over "rewrite the entire architecture."
- Distinguish **knowledge cutoff / training** from **live data**—questions about current prices, news, or private records imply **tools or retrieval**.
- **Token counting** matters for cost and context questions; prefer official counting over hand estimates when options mention it.

## Production Notes

- Log `usage` on every call; alert on context approaching model limits.
- Keep system prompts stable and cacheable; put volatile user content last (see [prompt caching](../02-prompt-engineering/prompt-caching.md)).
- Set `max_tokens` intentionally—too low truncates answers (`stop_reason: max_tokens`); too high wastes budget on runaway outputs.
- Persist conversation history in **your** store; design retention and PII policies explicitly.
- Use idempotent retries only with care—LLM outputs may differ between attempts.

## Common Mistakes

- Assuming Claude remembers users across sessions without sending history
- Confusing characters with tokens when estimating cost or fit
- Using high temperature for structured extraction tasks
- Blaming the model when tool results were never appended to messages
- Sending entire corpora in one prompt instead of retrieval or chunking
- Ignoring `stop_reason` when debugging truncated or incomplete responses

## See Also

- [Claude Models](./claude-models.md)
- [API Overview](./api-overview.md)
- [Token Pricing](./token-pricing.md)
- [Messages API](../03-api/messages-api.md)
- [Context Window](../09-performance/context-window.md)
- [Prompting](../02-prompt-engineering/prompting.md)
