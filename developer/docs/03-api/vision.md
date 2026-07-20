# Vision

Claude accepts **images** as content blocks inside Messages API requests, enabling multimodal reasoning over screenshots, diagrams, photos, and UI mockups. CCDV F tests how you structure image content, respect limits, and combine vision with tools and agents.

**Domain focus:** Applications and Integration (33%) — multimodal request shape and implementation.

## Learning Objectives

After this page you should be able to:

- Embed images in `messages` using supported content block types (`image` with base64 or URL source).
- State image format, size, and count constraints at a high level.
- Combine vision inputs with tool use and streaming in one conversation.
- Choose vision-capable models and appropriate resolution strategies.
- Avoid common multimodal mistakes in production and on the exam.

## Key Ideas

### Where images live

Images are **content blocks** inside a `user` (or tool-result) message — not a separate API:

```json
{
  "role": "user",
  "content": [
    {
      "type": "image",
      "source": {
        "type": "base64",
        "media_type": "image/jpeg",
        "data": "<base64-encoded bytes>"
      }
    },
    { "type": "text", "text": "What error is shown in this screenshot?" }
  ]
}
```

Alternative source type: **`url`** (where supported) — fetch from HTTPS URL instead of inline base64.

### Supported formats (typical)

- **JPEG, PNG, GIF, WebP** — verify current docs for exact list.
- Each image consumes **tokens** based on dimensions (not just file byte size) — large images cost context.

### How the model "sees" images

- Images are encoded into the model context like text — they count toward the **context window**.
- Multiple images can appear in one message or across history turns.
- Pair images with **clear text instructions** — "describe" vs "extract table" vs "find the bug".

### Vision + tools + agents

Vision does not change the agent loop:

1. User sends image + question.
2. Model may return `tool_use` (e.g., crop, OCR pipeline, database lookup) — `stop_reason: tool_use`.
3. Append `tool_result`, continue until `end_turn`.

**Schema ≠ semantic truth** still applies — if the model extracts structured fields from an image via a tool, validate in application code.

### Model selection

- Use **vision-capable** Claude models (current Sonnet/Opus/Haiku tiers support vision — confirm for your snapshot).
- For **many small images** or high volume, consider Haiku for cost; Opus/Sonnet for complex diagram reasoning.

### Resolution and preprocessing

- Extremely large images may be **downscaled** internally — for fine text/OCR, crop or preprocess client-side.
- Screenshots for UI debugging: full-screen captures often work; tiny text may need zoom/crop.

### PDFs vs images

- Multi-page documents → prefer [PDF support](./pdf.md) or [Files API](./files.md) over rasterizing every page as PNG unless you need pixel-perfect UI analysis.

## Exam Notes

- **"User uploads a screenshot"** → image content block in `messages`, not a separate Vision API endpoint.
- **"Analyze diagram in chat"** → multimodal message + optional tools; history must include image in the turn where it was sent.
- **Token budget question** → images consume input tokens; compress or resize when context is tight.
- **Wrong answer:** sending image URL as plain text string without structured `image` block.
- Vision scenarios still test **`stop_reason`** if the follow-up is tool-based extraction.

## Production Notes

- **Never** embed secrets in screenshots before sending to the API — redact PII and credentials.
- Cache stable system prompts; images usually change every turn — poor cache hit rate on image-heavy threads.
- Store images in your blob storage; send URLs or base64 per request — Anthropic does not persist your images between calls.
- Validate file type and size **before** upload; reject unsupported media early.
- Log image dimensions and token estimates for cost monitoring.
- For repeated analysis of the same asset, keep the image in history or re-send consistently — model has no memory otherwise.

## Common Mistakes

- Putting base64 in a `text` block instead of an **`image` block with `source`**.
- Omitting `media_type` on base64 sources.
- Assuming the model remembers an image from **ten turns ago** without it still being in `messages`.
- Sending 20 full-resolution photos and hitting **context limit** — resize or summarize first.
- Using Message Batches for **interactive image chat** (wrong pattern).
- Trusting extracted JSON from vision+tools without **app validation**.

## See Also

- [Messages API](./messages-api.md)
- [Files](./files.md)
- [PDF](./pdf.md)
- [Context Window](../09-performance/context-window.md)
- [Tool Use](../04-agent-engineering/tool-use.md)
