# API Overview

High-level map of Anthropic's Claude **developer platform**: the Messages API surface, related APIs, authentication, and how requests flow through a typical integration.

> **Verify on official docs:** Endpoints, beta headers, and SDK versions evolve. Start from [Anthropic API reference](https://docs.anthropic.com/en/api/messages) and [developer platform](https://docs.anthropic.com/).

**CCDV F domains:** Applications and Integration · Model Selection and Optimization

## Learning Objectives

After this page, you should be able to:

- Describe the **Messages API** request/response lifecycle
- List major API capabilities: streaming, tools, vision, batches, token counting, caching
- Explain authentication, versioning (`anthropic-version`), and beta feature headers
- Interpret **`stop_reason`** and **`usage`** in responses
- Choose the right API feature for common exam scenarios (not re-architect unnecessarily)

## Key Ideas

### Platform Architecture

```
┌─────────────┐     HTTPS JSON      ┌──────────────────┐
│  Your app   │ ──────────────────► │ Anthropic API    │
│  (SDK/HTTP) │ ◄────────────────── │ (Claude models)  │
└─────────────┘   response + usage  └──────────────────┘
        │
        ▼
  Your storage: history, tool execution, auth, billing
```

Your application owns **orchestration**: persisting messages, executing tools, retries, and user sessions.

### Core API: Messages

`POST /v1/messages` is the primary integration point.

**Request essentials:**

| Field | Purpose |
| --- | --- |
| `model` | Which Claude snapshot to invoke |
| `max_tokens` | Maximum output tokens to generate |
| `messages` | Conversation array (`user` / `assistant` roles) |
| `system` | System instructions (string or structured blocks) |
| `tools` | Tool definitions for `tool_use` |
| `tool_choice` | Auto, none, any, or force specific tool |
| `temperature`, `top_p`, `top_k` | Sampling controls |
| `stop_sequences` | Custom stop strings |
| `metadata` | Optional tracing IDs (e.g., `user_id`) |

**Response essentials:**

| Field | Purpose |
| --- | --- |
| `content` | Array of blocks: `text`, `tool_use`, thinking (model-dependent) |
| `stop_reason` | Why generation ended—**drives agent loops** |
| `usage` | `input_tokens`, `output_tokens`, cache fields if applicable |
| `id`, `model`, `role` | Metadata for logging |

### `stop_reason` Values (Critical for Agents)

| Value | Meaning | Typical next step |
| --- | --- | --- |
| `end_turn` | Natural completion | Return response to user |
| `tool_use` | Model wants to call tool(s) | Execute tools; append `tool_result` messages; call API again |
| `max_tokens` | Hit output limit | Increase `max_tokens` or ask model to continue |
| `stop_sequence` | Matched a custom stop | Handle per your protocol |

Developer exam gold: **`tool_use` → execute → append results → recall API** until `end_turn`.

### Tool Use Flow

1. Send `tools` definitions with messages
2. Model responds with `tool_use` content blocks (name + `input` JSON)
3. Your app executes the tool locally or via backend
4. Append `user` message with `tool_result` blocks (include `tool_use_id`)
5. Call Messages API again with full history

Never skip step 4—exams and production failures often trace to missing tool results.

### Streaming

`stream: true` returns **Server-Sent Events** incrementally (`message_start`, `content_block_delta`, `message_delta`, etc.).

| Mode | When to use |
| --- | --- |
| **Synchronous** | Short responses, batch pipelines, simple scripts |
| **Streaming** | Chat UIs, long generations, better perceived latency |

Streaming changes **delivery**, not the underlying model behavior. You still parse final `stop_reason` and tool blocks.

### Structured Output

Anthropic supports constraining outputs (e.g., JSON matching a schema) via documented mechanisms that evolve—often **`tools` with strict schemas** or dedicated structured-output beta headers. For exams: prefer **programmatic enforcement** (schema/tool) over "return valid JSON only" prompts.

See [structured prompts](../02-prompt-engineering/structured-prompts.md).

### Prompt Caching

Mark cacheable content with `cache_control` breakpoints (typically long stable system prompts). Subsequent requests with matching prefixes read from cache at reduced cost/latency. Requires supported models and minimum token thresholds—see [prompt caching](../02-prompt-engineering/prompt-caching.md).

### Message Batches API

Async bulk processing at discounted rates for non-real-time workloads. Submit many Messages requests in one batch job; poll for results. Wrong for interactive chat; right for offline ETL, eval runs, document processing.

### Token Counting API

Count tokens **before** sending large prompts to avoid overflow and estimate cost. Use instead of guessing character lengths.

### Files API (When Enabled)

Upload and reference files per current docs for document workflows. Pair with Messages API for PDF/document Q&A patterns.

### Authentication and Headers

| Header | Purpose |
| --- | --- |
| `x-api-key` | API key (server-side only—never expose in client browsers) |
| `anthropic-version` | Dated API version (e.g., `2023-06-01`) |
| `anthropic-beta` | Comma-separated beta features (structured outputs, etc.) |

Use official SDKs (`@anthropic-ai/sdk`, `anthropic` Python) to manage headers and typing.

### Error Classes (Overview)

| HTTP | Meaning | Typical action |
| --- | --- | --- |
| 400 | Bad request / invalid params | Fix payload |
| 401 | Auth failure | Rotate/check API key |
| 403 | Permission / policy | Review org settings |
| 429 | Rate limit | Exponential backoff + jitter |
| 529 | Overloaded | Retry with backoff |

Details: [error handling](../03-api/error-handling.md), [retries](../03-api/retries.md), [rate limits](../03-api/rate-limits.md).

### SDK vs Raw HTTP

SDKs provide typed requests, streaming helpers, and retries. Exams may show pseudo-code—map it to the same fields regardless of language.

## Exam Notes

- **"Guarantee JSON shape"** → structured output / tool schema—not stronger adjectives in the prompt.
- **Agent stuck after tool call** → missing `tool_result` in history.
- **Bulk overnight job** → Message Batches, not 10k synchronous calls.
- **UI feels slow** → streaming first; model tier second.
- **`tool_choice: {type: "tool", name: "..."}`** forces a specific tool when the scenario requires deterministic tool invocation.
- Know difference between **`system`** parameter vs a `user` message pretending to be system instructions—prefer real `system` for policy.

## Production Notes

- Terminate TLS; store keys in secrets manager; rotate regularly.
- Implement centralized retry policy for 429/529 with max attempts.
- Log `id`, `model`, `stop_reason`, and `usage` on every call.
- Validate tool inputs before execution (schema + auth)—models can hallucinate arguments.
- Use `metadata.user_id` for abuse tracking—not PII you don't need.
- Pin `anthropic-version` and test before upgrading.

## Common Mistakes

- Calling Claude from browser frontends with exposed API keys
- Treating API as stateful—forgetting to resend conversation history
- Ignoring `stop_reason: tool_use` and showing empty UI to users
- Using synchronous API for massive parallel offline jobs
- Omitting `tool_use_id` when returning `tool_result` blocks
- Upgrading SDK/API version without reading breaking change notes

## See Also

- [Messages API](../03-api/messages-api.md)
- [Streaming](../03-api/streaming.md)
- [Tool Use](../04-agent-engineering/tool-use.md)
- [LLM Basics](./llm-basics.md)
- [Token Pricing](./token-pricing.md)
- [Prompt Caching](../02-prompt-engineering/prompt-caching.md)
