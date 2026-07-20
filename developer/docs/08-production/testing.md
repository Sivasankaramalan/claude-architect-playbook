# Testing

**Testing** Claude applications combines traditional software tests (unit, integration) with **LLM-specific** checks: tool routing, schema compliance, agent loop termination, and regression evals. The exam's **Eval, Testing, and Debugging (2.6%)** domain expects you to know *what* to test and *how*, not a specific test framework brand.

## Learning Objectives

After this page, you should be able to:

- Layer unit, integration, contract, and eval tests for Claude apps
- Test tool definitions, MCP servers, and orchestration without calling the model when possible
- Design CI-friendly deterministic tests vs nightly model-in-the-loop suites
- Mock API responses for fast feedback while keeping live smoke tests
- Connect test failures to prompt, tool, or infrastructure layers

## Key Ideas

### Test pyramid for LLM apps

```text
                    ┌─────────────┐
                    │ Human / E2E │  few, expensive
                    ├─────────────┤
                    │ Eval suite  │  golden tasks, model calls
                    ├─────────────┤
                    │ Integration │  API + tools + DB fixtures
                    ├─────────────┤
                    │ Unit        │  parsers, validators, routing
                    └─────────────┘
```

Most tests should **not** call Claude—test your code paths deterministically.

### What to unit test (no model)

- JSON schema validators and output parsers
- Tool argument validators (reject invalid IDs before execution)
- Message history builders (`tool_result` pairing with `tool_use_id`)
- Retry/backoff logic
- Prompt template rendering (snapshot static strings)
- Routing rules (intent → model or tool choice)

### Integration tests

- Messages API against **staging** with fixed prompts and low temperature
- MCP server contract: list tools, invoke with fixture args
- Agent loop: mock model responses from recorded fixtures, assert tool execution order
- Rate limit and error injection (429 simulator)

Use **recorded fixtures** (`vcr`-style) for repeatable assistant messages in CI.

### Contract tests for tools and MCP

Tools are APIs the model calls:

| Contract element | Test |
| --- | --- |
| Input schema | Required fields, enums |
| Output shape | Stable JSON for model consumption |
| Errors | Structured `{ code, message, retryable }` |
| Idempotency | Duplicate calls safe where claimed |
| Latency | Timeout behavior |

Breaking changes to tool outputs cause silent agent degradation—version tools explicitly.

### Eval tests (see Evaluations)

- Golden conversations with graders in CI (nightly or PR label)
- Compare scores across prompt/model changes
- Fail build on regression beyond threshold

Separate **fast PR gate** (10 cases) from **full suite** (100+).

### Agent loop tests

Assert control flow via `stop_reason`:

1. Fixture response `stop_reason: tool_use` with specific tool
2. Assert your executor runs that tool and appends `tool_result`
3. Fixture `stop_reason: end_turn`
4. Assert loop exits and output validation runs

Do not test by string-matching "I'll use the search tool" in model prose—assert structured `tool_use` blocks.

### Smoke and canary

After deploy:

- Live single-turn health check
- Critical path eval subset
- Compare error rate to baseline (canary)

## Exam Notes

- Automated regression after prompt change → **eval suite / deterministic checks**, not manual only
- Tool behavior guarantee → **server validation + contract tests**, not prompt
- Fast CI → **mock model or fixture responses**; live API for smoke only
- Testing structured output → **schema validation tests**
- Agent infinite loop → test **max turns** and `stop_reason` handling

## Production Notes

### Testing checklist

- [ ] Unit tests for all validators and history assembly
- [ ] MCP/tool contract tests in CI
- [ ] Recorded agent loop fixtures for regression
- [ ] Eval golden set versioned with prompts
- [ ] Staging smoke test post-deploy
- [ ] Load test rate limits separately from functional tests

### Environments

| Env | Purpose |
| --- | --- |
| Local | Mocks + optional dev API key |
| CI | Fixtures; scheduled live eval job |
| Staging | Production-like tools, fake data |
| Prod | Synthetic monitors only |

Never run destructive integration tests against production tools.

## Common Mistakes

- **Only manual chat testing** — no regression detection
- **Every test hits live API** — slow, flaky, costly CI
- **Asserting exact natural language** — brittle; assert structure and graders
- **No tests for tool errors** — agent loops break in prod first
- **Skipping MCP contract tests** — schema drift undetected

## See Also

- [Evaluations](./evaluations.md)
- [Debugging](./debugging.md)
- [Guardrails](./guardrails.md)
- [Tool Results](../04-agent-engineering/tool-results.md)
- [Agent Loop](../04-agent-engineering/agent-loop.md)
