# Files API

The **Files API** lets you upload documents and other assets to Anthropic-managed storage and reference them in Messages API requests. It simplifies handling large or reusable files compared to inlining base64 on every call.

**Domain focus:** Applications and Integration (33%) — file upload lifecycle and message integration.

## Learning Objectives

After this page you should be able to:

- Describe the upload → reference → use-in-message workflow for the Files API.
- Explain when Files API beats inline base64 or public URLs.
- Integrate file references with multimodal and PDF/document workflows.
- Apply retention, access, and security considerations in production.
- Pick the right document strategy on exam scenarios (inline image vs file upload vs PDF block).

## Key Ideas

### Workflow overview

```
1. Upload file  →  receive file_id
2. Reference file_id in message content (per docs for file/document blocks)
3. Model reads content in context for that request
4. Manage lifecycle (delete when no longer needed)
```

Exact content block shapes evolve — always match the **current Anthropic Files API reference** when implementing. Conceptually: you attach **`file_id`** instead of re-uploading bytes each turn.

### When to use Files API

| Scenario | Approach |
| --- | --- |
| Large PDF, repeated queries | Upload once, reference by ID across turns |
| One-off small screenshot | Inline base64 image block — [vision.md](./vision.md) |
| User attachment in SaaS app | Upload to Files API, store `file_id` in your DB |
| Bulk offline analysis | Message Batches + file references |

### Files vs inline content

| | Files API | Inline base64 |
| --- | --- | --- |
| Payload size per request | Small (ID reference) | Large (full bytes) |
| Reuse across calls | Easy | Must resend or keep in history |
| Setup | Upload step required | Single request |
| Best for | Large/reused documents | Small images, prototypes |

### Conversation history

- If the model must refer to a document **later in the thread**, the file reference (or the content derived from it) must remain accessible in **`messages` history** or be re-attached — same stateless API rule as text.
- Do not assume Anthropic remembers a file from a prior session without your app re-supplying context.

### Files + tools + agents

Typical agent pattern:

1. User uploads contract PDF → your backend stores `file_id`.
2. User asks question → message includes file reference + text.
3. Model returns `tool_use` to search clause database → `stop_reason: tool_use`.
4. Append `tool_result`, continue until `end_turn`.

Tool results can include **extracted text snippets** from files your tools processed — keep `tool_use_id` pairing correct.

### Supported file types

- Documents (PDF, plain text, etc.) and images depending on current platform support — verify MIME types in official docs before exam day.
- Unsupported types → convert client-side (e.g., DOCX → PDF/text) before upload.

## Exam Notes

- **"10 MB PDF queried many times"** → Files API upload + reference, not base64 every request.
- **"Reduce request payload"** → file ID reference vs inline data.
- **"Where is conversation state?"** → still in your app's `messages` array — Files API is storage, not chat memory.
- Distractor: storing only `file_id` in your DB without including it in the **message** when the model needs it again.
- File workflows still use **`stop_reason`** for tool-based extraction scenarios.

## Production Notes

- **Delete** unused files to control storage and compliance exposure.
- Virus-scan and validate uploads on **your** edge before forwarding to Anthropic.
- Never expose `file_id` to other tenants — treat as capability-bearing resource ID in multi-tenant apps.
- Log upload size, MIME type, and associated user/session for audit.
- Combine with **prompt caching** for stable system prompts, not necessarily for changing file content.
- Rate limits apply to upload endpoints — see [rate-limits.md](./rate-limits.md).

## Common Mistakes

- Uploading once but **never referencing** the file in subsequent messages when user asks follow-ups.
- Confusing Files API with **Message Batches** (batch = async job queue, files = storage).
- Sending megabytes of base64 when **`file_id`** is the intended pattern.
- Assuming file upload **replaces** tool use for structured extraction — often you still need tools + validation (schema ≠ truth).
- Skipping **access control** on who can reference a given `file_id`.

## See Also

- [PDF](./pdf.md)
- [Vision](./vision.md)
- [Messages API](./messages-api.md)
- [Rate Limits](./rate-limits.md)
- [Security](../08-production/safety.md)

---

Copyright (c) 2026 Sivasankaramalan Gunasekarasivam. Licensed under the MIT License.
