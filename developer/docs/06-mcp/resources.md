# MCP Resources

**Resources** are MCP’s **read-only context** primitive — documents, records, file slices, or API snapshots the host loads into the model’s context **without** executing a mutating action. Think “GET endpoints” for the LLM.

Public reference: [MCP concepts — resources](https://modelcontextprotocol.io/docs/learn/architecture).

## Learning Objectives

- Distinguish **resources** from **tools** and **prompts**
- Describe when application-controlled reads beat model-invoked tools
- Explain `resources/list` and `resources/read` flow
- Choose resources for large static corpora vs tools for live queries
- Avoid using tools for side-effect-free reads when resources fit

## Key Ideas

### Control model

| Primitive | Who initiates | Side effects |
| --- | --- | --- |
| **Tools** | Model (tool_use) | Allowed — design carefully |
| **Resources** | Host / application | Should be **read-only** |
| **Prompts** | User / UI | Templates only |

Resources are **application-controlled** — the host decides what to attach, when, and how much fits the context window.

### When to use resources

Good fits:

- Policy PDFs, schema docs, style guides
- Ticket descriptions, read-only CRM fields
- Database row snapshots (non-mutating)
- File contents referenced by URI

Poor fits:

- “Create invoice” → **tool**
- “Run arbitrary SQL” → **tool** with validation, not a resource
- Dynamic computation with business logic → usually **tool**

### URI and metadata

Servers advertise resources via `resources/list`:

- **URI** — stable identifier (`file:///…`, `db://…`, custom scheme)
- **Name / description** — human + model-facing summary
- **MIME type** — hints for parsing

Clients fetch content with `resources/read` for a given URI.

### Resources vs stuffing context manually

Both put text in context. Resources win when:

- Content comes from **external systems** standardized across hosts
- You want **discovery** (list available docs) without hardcoding paths in prompts
- Multiple apps reuse the same MCP server catalog

### Size and context window

Resources still consume **tokens**. Patterns:

- Summarize large resources server-side before read
- Paginate or offer **resource templates** with parameters
- Prefer resources for **reference** material; tools for **targeted fetch** by query

Pair with [context pruning](../09-performance/pruning.md) strategies in long sessions.

### Subscriptions (awareness)

MCP supports resource **subscription** notifications when data changes — useful for live dashboards. Foundational exam focus remains list/read semantics.

## Exam Notes

- **“Expose handbook for model to cite”** → **resource**, not mutating tool.
- **“Update record status”** → **tool**, not resource.
- Resources do **not** replace tools for **actions** — classic distractor pairing.
- Application-controlled vs model-controlled is a **favorite** multiple-choice axis.
- Read-only **does not** mean safe — sensitive data still needs auth on `resources/read`.

## Production Notes

- Enforce same **auth** on read as on write paths.
- Redact PII in resource payloads when full record isn’t needed.
- Cache reads with TTL; log access for compliance.
- Stable URIs — changing URIs breaks agent prompts referencing them.
- Monitor token size of common resources; offer “summary” resource variants.

## Common Mistakes

- Implementing `delete_user` as a “resource” because it returns JSON
- Loading every resource into context at session start — context bloat
- No description on list entries — model ignores available docs
- Confusing MCP resources with **prompt caching** API feature (different layer)
- Using tools for static docs the app could prefetch once per session

## See Also

- [MCP Tools](./tools.md)
- [MCP Prompts](./prompts.md)
- [MCP Introduction](./introduction.md)
- [Context Window](../09-performance/context-window.md)
- [Prompt Caching](../02-prompt-engineering/prompt-caching.md)
