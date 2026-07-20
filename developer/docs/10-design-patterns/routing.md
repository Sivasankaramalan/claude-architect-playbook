# Routing

**Routing** sends each request to the right model, prompt, tool set, or workflow path—optimizing cost, latency, and quality. Heavily tested under **Model Selection and Optimization (16.8%)** and **Agents and Workflows (14.7%)**.

## Learning Objectives

After this page, you should be able to:

- Implement intent-based and rule-based routing before expensive agent paths
- Match model tier (Haiku/Sonnet/Opus) to task complexity
- Route to specialized prompts and tool subsets safely
- Combine routing with guardrails and eval metrics
- Avoid misrouting that sends hard tasks to cheap models

## Key Ideas

### Routing layers

```text
Request
   │
   ├─ Layer 1: Rules (regex, metadata, auth role)
   ├─ Layer 2: Classifier (Haiku or embeddings)
   ├─ Layer 3: LLM planner (complex ambiguous cases)
   └─ Layer 4: Fallback / human queue
```

Use the **cheapest accurate layer** first.

### Model routing

| Signal | Route to |
| --- | --- |
| FAQ, classification, extraction | Haiku-class |
| General coding, agents, chat | Sonnet-class |
| Multi-step reasoning, high stakes | Opus-class |
| User tier premium | Higher model allowance |

Log routing decisions for eval: did cheap route succeed?

### Tool routing

Don't expose 30 tools to every request:

- **Intent → tool subset** reduces wrong tool picks
- **`tool_choice`** force specific tool when path is known (e.g., always `lookup_order` after order_id detected)

See [Tool Use](../04-agent-engineering/tool-use.md).

### Prompt routing

Maintain specialized system prompts:

- `support_billing.md`
- `support_technical.md`

Router selects template; avoids one prompt trying to cover everything poorly.

### Semantic routing

Embedding similarity or small classifier maps query → department workflow. Calibrate with eval set; watch drift when product changes.

### Safety routing

Sensitive or injection-suspect inputs → restricted tools, human review queue, or refusal path—route **before** full agent with write tools.

## Exam Notes

- High volume simple queries → **Haiku router + specialized path**
- Known next step after parsing order ID → **`tool_choice`**, not hope
- Wrong tool often selected → **reduce tools via routing**, not only better descriptions
- Cost optimization without quality loss → **route easy to Haiku**, hard stays Sonnet/Opus
- Security-sensitive → **restrict tool surface** on route

## Production Notes

### Routing checklist

- [ ] Routing decision logged with reason code
- [ ] Eval coverage per route
- [ ] Fallback when classifier confidence low
- [ ] A/B test new routes with canary traffic
- [ ] Monitor cost and success rate by route

## Common Mistakes

- **Opus for routing** — expensive; use Haiku/rules
- **Haiku for complex coding agent** — rework loops cost more
- **All tools always enabled** — routing ignored
- **No fallback** — low-confidence misroutes silently
- **Routing only in prompt** ("if billing…") — enforce in code

## See Also

- [Optimization](../09-performance/optimization.md)
- [Latency](../09-performance/latency.md)
- [Architecture](./architecture.md)
- [Claude Models](../01-foundations/claude-models.md)
- [Manager Pattern](../04-agent-engineering/manager-pattern.md)
