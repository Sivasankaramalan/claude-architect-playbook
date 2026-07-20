# PDF

Claude can process **PDF documents** as part of Messages API requests — either via document content blocks, the Files API, or platform-supported PDF ingestion patterns. CCDV F focuses on choosing the right approach for document Q&A, extraction, and agent workflows.

**Domain focus:** Applications and Integration (33%) — document ingestion and multimodal integration.

## Learning Objectives

After this page you should be able to:

- Attach PDFs to messages using supported document/PDF content patterns.
- Compare inline PDF blocks, Files API references, and image-based fallbacks.
- Design document Q&A flows that respect context limits and token costs.
- Combine PDF analysis with tool use (`stop_reason: tool_use`) for structured extraction.
- Identify exam traps around persistence, validation, and history.

## Key Ideas

### PDF in the Messages API

PDFs enter the model context as **document content** (exact block type per current API — commonly a `document` block with base64 PDF bytes or a `file_id` from [Files API](./files.md)).

Typical pattern:

```json
{
  "role": "user",
  "content": [
    {
      "type": "document",
      "source": {
        "type": "base64",
        "media_type": "application/pdf",
        "data": "<base64-encoded pdf>"
      }
    },
    { "type": "text", "text": "Summarize section 3 and list action items." }
  ]
}
```

Verify block names against latest docs — exam questions test **concepts**, not memorized JSON keys.

### Three ways to send PDFs

| Method | Best for |
| --- | --- |
| **Inline document block** | Small PDFs, prototypes, single-shot analysis |
| **Files API reference** | Large or reused PDFs across many requests — [files.md](./files.md) |
| **Page images (fallback)** | When you need pixel layout analysis rare in text extraction — heavier token cost |

### Token and context impact

- PDFs can consume **large input token counts** — multi-hundred-page docs may exceed context or be expensive.
- Strategies: chunking, retrieve relevant pages (RAG), pre-extract text with your pipeline, or ask targeted questions per section.

### PDF + structured extraction + tools

Exam favorite pattern:

1. User attaches PDF + "extract invoice fields as JSON."
2. Model emits `tool_use` with structured `input` → `stop_reason: tool_use`.
3. Your app validates JSON against business rules — **schema validates shape, not semantic truth** (wrong totals can still parse).
4. Return `tool_result` (or `is_error: true` if validation fails), continue loop.

Force extraction with `tool_choice: { type: "tool", name: "extract_invoice" }` when the question demands guaranteed tool invocation.

### PDF in agent loops

Long document workflows often combine:

- **Retrieval tool** (search chunks) → multiple tool rounds
- **Summarization** across turns with **full history appended**
- Human approval before acting on extracted commitments — see [../04-agent-engineering/agent-best-practices.md](../04-agent-engineering/agent-best-practices.md)

Each round: check `stop_reason` — only stop agent on `end_turn`.

### PDF vs plain text upload

- Text/markdown extracts are cheaper in tokens if layout doesn't matter.
- PDF preserves tables, headers, and structure the model can reason over natively — prefer when formatting carries meaning.

## Exam Notes

- **"Analyze uploaded contract"** → document content in message or Files API — not paste raw binary in text field.
- **"Guarantee JSON output from PDF"** → tool with schema + `tool_choice`, plus server validation.
- **"Follow-up question about same PDF"** → keep document in history or re-attach; API is stateless.
- **Context exceeded** → chunk/RAG distractor beats "use smaller model" alone.
- Wrong: assuming one PDF upload persists forever without re-reference in `messages`.

## Production Notes

- Redact sensitive PDFs before cloud upload; classify data handling per compliance tier.
- Pre-flight **page count and size** limits; reject oversize at API gateway.
- Cache extracted text in **your** store for repeat queries — don't re-tokenize 200 pages every turn.
- Log token usage per document job for billing alerts.
- When extraction fails, return `tool_result` with `is_error: true` and actionable message — model can recover in next turn.
- Test scanned PDFs — OCR quality affects answers; may need dedicated OCR tool in the loop.

## Common Mistakes

- Sending PDF as **`text` content** (garbled/base64 visible to model incorrectly).
- One-shot upload without plan for **follow-up turns** in conversation.
- Trusting model-extracted numbers for billing/legal without **validation**.
- Using full PDF when **search tool + chunks** is the scalable architecture.
- Ignoring **`max_tokens`** on long summarization tasks (`stop_reason: max_tokens`).

## See Also

- [Files](./files.md)
- [Vision](./vision.md)
- [Messages API](./messages-api.md)
- [Tool Use](../04-agent-engineering/tool-use.md)
- [Tool Results](../04-agent-engineering/tool-results.md)
- [Context Window](../09-performance/context-window.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
