# Evaluations

**Evaluations (evals)** measure whether your Claude application behaves correctly on representative tasks—not just whether the API returned HTTP 200. Evals turn subjective "the model feels worse" into regression-detecting signals you can run in CI and before releases.

This page covers eval types, dataset design, and how Anthropic-style agent development uses evals in the loop. Primary domain: **Eval, Testing, and Debugging (2.6%)**; also supports **Agents and Workflows (14.7%)** and **Prompt and Context Engineering (11.0%)**.

## Learning Objectives

After this page, you should be able to:

- Distinguish offline, online, human, and model-graded evals
- Design task datasets with clear pass/fail or rubric criteria
- Connect eval failures to prompt, tool, or orchestration changes
- Place evals in the development lifecycle (pre-merge, nightly, pre-release)
- Avoid common eval pitfalls: data leakage, flaky LLM judges, and vanity metrics

## Key Ideas

### Why evals matter for Claude apps

LLM outputs are variable. Unit tests assert fixed outputs; evals assert **behavior under constraints**:

- Correct tool selected?
- JSON matches schema?
- Answer grounded in provided context?
- Agent completes within N turns?

Without evals, prompt tweaks and model upgrades are guesswork.

### Eval types

| Type | When | Strength | Weakness |
| --- | --- | --- | --- |
| **Code-based (deterministic)** | CI, every PR | Fast, reproducible | Limited to checkable properties |
| **Human eval** | New capability, rubric quality | Gold standard judgment | Slow, expensive |
| **Model-graded (LLM-as-judge)** | Scale semantic checks | Cheaper than humans | Judge bias; needs calibration |
| **Online / A-B** | Production | Real user signal | Noisy; slower feedback |
| **Regression suite** | After changes | Catches breakage | Must maintain cases |

Combine **deterministic checks first** (schema, tool name, regex, exact match on IDs), then semantic judges for nuance.

### Anatomy of an eval case

Each case should specify:

1. **Input** — user message, system prompt snapshot, tool definitions, optional fixtures
2. **Expected behavior** — not always exact text; may be "must call `get_order` with order_id" or "refuse harmful request"
3. **Graders** — functions or rubrics that return pass/fail or score
4. **Metadata** — domain tag, difficulty, source (prod failure, synthetic)

```text
eval_case: refund_eligible
  input: "Can I refund order 8821?"
  graders:
    - tool_called: get_order
    - schema_valid: refund_response
    - no_hallucinated_policy: rubric_v2
```

### Prompt and agent eval scope

Eval at the layer you control:

| Layer | Example assertion |
| --- | --- |
| Single-turn prompt | Output contains required fields |
| Tool use | `stop_reason` + correct `tool_use` block |
| Multi-turn agent | Task done in ≤ 5 turns, no forbidden tools |
| End-to-end | API + MCP + DB fixture → business outcome |

Start narrow; expand to E2E only for critical paths.

### Eval-driven iteration loop

```text
Baseline eval score
      │
      ▼
Change prompt / tools / model
      │
      ▼
Re-run eval suite
      │
      ├── score ↑ → ship (with spot human review)
      └── score ↓ → revert or fix before merge
```

Treat eval datasets as **versioned code**—review additions like test fixtures.

### Production-sourced evals

Best cases come from real failures:

1. Log failure with trace (see [Observability](./observability.md))
2. Anonymize and minimize context
3. Add expected behavior after human triage
4. Prevent recurrence in CI

## Exam Notes

- **Eval domain (2.6%)**: know offline vs A/B vs human—not specific vendor product names
- "How do you know a prompt change didn't regress?" → **regression eval suite**, not "manual spot check only"
- Deterministic graders beat prompt-only self-checks when the stem says **automated** or **CI**
- Model-graded evals need **calibration** against human labels—watch for trap answers that say "let the model grade itself with no baseline"
- Evals complement but do not replace **guardrails** and **schema validation**

## Production Notes

### Minimum eval program

- [ ] 20–50 golden cases per critical workflow (start smaller, grow from incidents)
- [ ] Deterministic graders for tool selection and structured output
- [ ] Nightly full suite; PR subset for speed
- [ ] Track score trends by model version and prompt hash
- [ ] Human review queue for cases where judge disagrees with deterministic checks

### Metrics that matter

- **Pass rate** on golden set
- **Tool accuracy** — right tool, right args
- **Turn count** — agent efficiency
- **Cost per successful task** — ties to [Cost Control](../09-performance/cost-control.md)

Avoid optimizing only **BLEU-like text similarity** when business logic is tool- and schema-driven.

## Common Mistakes

- **Evaluating only final natural language** — ignoring wrong tools that accidentally sound plausible
- **Single mega-prompt change without baseline** — no diff signal
- **Leaking eval answers into prompts** — near-duplicate phrasing in system prompt
- **Flaky LLM judge without threshold** — random score swings block CI
- **No evals for failure modes** — only happy-path cases

## See Also

- [Testing](./testing.md)
- [Debugging](./debugging.md)
- [Guardrails](./guardrails.md)
- [Agent Best Practices](../04-agent-engineering/agent-best-practices.md)
- [Structured Prompts](../02-prompt-engineering/structured-prompts.md)
