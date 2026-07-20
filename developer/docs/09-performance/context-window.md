# Context Window

The **context window** is the maximum number of tokens Claude can consider in a single request—system prompt, tools, conversation history, retrieved documents, and multimodal inputs. Context management is central to **Model Selection and Optimization (16.8%)** and **Prompt and Context Engineering (11.0%)**.

## Learning Objectives

After this page, you should be able to:

- Explain input vs output token limits and how they interact with `max_tokens`
- Estimate when conversations approach context limits
- Apply strategies to fit within window without losing task-critical information
- Choose models appropriate for large-context workloads
- Recognize exam scenarios about truncation, forgetting, and long documents

## Key Ideas

### What counts toward context

Everything sent in the **messages** array (and system prompt) consumes input tokens:

- System instructions and `CLAUDE.md`-style persistent rules
- Tool definitions (names, descriptions, schemas)—often underestimated
- Full conversation history including prior tool calls and **tool results**
- RAG chunks, PDFs, images (vision tokens)
- User message for the current turn

Output tokens (`max_tokens`) reserve space for the response and do not count as input, but **input + max_tokens** must fit the model limit.

### Model context sizes (conceptual)

Claude model families ship with large context windows (commonly 200K tokens for recent Sonnet/Opus-class models—verify current docs for exact limits). **Haiku** may differ; always check model cards before exam day.

Large window ≠ use it all blindly—cost and latency scale with tokens.

### Truncation and failure modes

When context exceeds limits:

- API errors (request too large)
- Forced omission if your app pre-truncates history
- "Forgotten" facts from early turns still in UI but not sent to model

Users experience the last case as **silent context loss**.

### Context engineering strategies

| Strategy | When |
| --- | --- |
| **Summarize older turns** | Long chats; keep recent verbatim |
| **Retrieve don't dump** | RAG with top-k chunks vs full corpus |
| **Subagent isolation** | Subtask with clean window + returned summary |
| **Tool-based memory** | Store facts externally; fetch via tools |
| **Prune tool results** | Drop verbose logs from history—see [Pruning](./pruning.md) |
| **Prompt caching** | Large static prefix—see [Caching](./caching.md) |

### Tool definition overhead

Ten poorly written tools can cost thousands of tokens per request. Consolidate tools, shorten descriptions, and remove unused parameters from schemas.

### Multimodal context

Images and PDFs consume substantial tokens. Resize, crop, or page-select before sending. See [Vision](../03-api/vision.md) and [PDF](../03-api/pdf.md).

### Counting tokens

Use Anthropic token counting APIs or SDK helpers before sending large payloads. Build CI checks that fail if system prompt + tools exceed budget.

## Exam Notes

- "Model forgot X from earlier" → **history not sent**, **summarized away**, or **subagent without context**—not "model memory bug"
- Long document Q&A → **retrieval/chunking** or **caching static doc prefix**, not repeated full paste each turn
- Maximize quality within budget → **relevant context only**, not entire log dump
- Large static system prompt reused → **prompt caching** breakpoint
- Context vs session: **resume** sends history; verify what's included programmatically

## Production Notes

### Context budget checklist

- [ ] Token count system + tools at deploy time
- [ ] Cap attachment size and chunk count
- [ ] Summarization policy after N turns or M tokens
- [ ] Store durable facts outside window (DB + tool fetch)
- [ ] Log input token usage per session for anomalies
- [ ] Test longest realistic session in staging

### When to upgrade model vs prune

If task needs **reasoning over huge corpus in one shot**, large-context Sonnet/Opus may fit. If corpus is mostly irrelevant, **RAG + smaller context** beats bigger window—cheaper and often more accurate.

## Common Mistakes

- **Sending full chat UI history** including hidden failed attempts
- **Leaving giant tool JSON in every turn**
- **Assuming 200K means you should use 200K** — latency and cost explode
- **No token monitoring** — surprised by limit errors in prod
- **Caching without stable prefix** — see [Caching](./caching.md)

## See Also

- [Pruning](./pruning.md)
- [Caching](./caching.md)
- [Cost Control](./cost-control.md)
- [Optimization](./optimization.md)
- [Token Pricing](../01-foundations/token-pricing.md)
- [Prompt Caching](../02-prompt-engineering/prompt-caching.md)
