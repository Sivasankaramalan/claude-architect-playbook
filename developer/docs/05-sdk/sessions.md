# Sessions

A **session** is a persisted agent run — conversation state, tool history, and metadata — that Claude Code or the Agent SDK can **resume** or **continue** instead of starting from zero. Session management is a frequent CCDV F scenario axis: *resume vs fresh context*.

## Learning Objectives

- Explain resume vs continue vs new session
- Know when stale tool results or poisoned context require a fresh session
- Use CLI flags (`--resume`, `--continue`) and SDK session options
- Relate sessions to subagents (separate session boundaries)
- Avoid confusing API `messages[]` history with Claude Code session files

## Key Ideas

### Why sessions matter

Long agent tasks span many model turns and tool calls. Persisting state enables:

- **Resume** after crash, rate limit, or user pause
- **Continue** the latest session in a project directory
- **Audit** what the agent did across turns

Without session discipline, teams re-send huge histories manually or lose work mid-task.

### Resume vs continue vs fresh

| Action | When to use | Risk if misapplied |
| --- | --- | --- |
| **Resume** (specific session id) | Known good checkpoint; task still valid | Resumes stale MCP/tool results |
| **Continue** (latest session) | Pick up where you left off in this project | May inherit bad assumptions |
| **Fresh session** | Wrong tools called, poisoned context, wrong branch | Loses useful detail unless summarized |

**Exam heuristic:** If the stem mentions **incorrect tool results in history** or **wrong codebase state** → fresh session + injected summary, not resume.

### Claude Code CLI (public patterns)

Common flags (verify exact names in current CLI docs):

| Flag | Purpose |
| --- | --- |
| `-p` / `--print` | Headless prompt mode |
| `--resume <id>` | Resume a specific session |
| `--continue` | Continue the most recent session in project |
| `--output-format json` | Structured output for automation |

Headless mode is how CI pipelines reuse the same harness as interactive Claude Code.

### Agent SDK sessions

The SDK exposes session identifiers through streamed messages. Pass **`resume`** (or equivalent session option) in `query()` options to attach to an existing run.

Pattern:

1. First `query()` → capture `session_id` from stream metadata.
2. Later `query(..., options={ resume: session_id })` → continue agent loop with preserved state.

Exact field names differ slightly between Python and TypeScript SDKs — check the SDK reference for your language.

### Sessions vs conversation history (Messages API)

| Concept | Messages API | Claude Code / Agent SDK |
| --- | --- | --- |
| State | You store `messages[]` | Harness stores session artifacts |
| Tool results | You append manually | Harness appends automatically |
| Resume | Re-send full history | Session id / continue flags |

CCDV F may ask either framing — map the stem to the right layer.

### Sessions and subagents

Subagents may have **their own** session boundaries. Coordinator session state does **not** automatically merge subagent internal turns. Pass explicit summaries or artifacts back to the coordinator.

### SessionStart / SessionEnd hooks

Shell hooks can inject environment context at **SessionStart** (recent git log, ticket id) or cleanup at **SessionEnd**. For static project conventions, **CLAUDE.md** is often simpler; hooks suit **dynamic** session bootstrap.

## Exam Notes

- **Stale tool output in history** → new session, not `--continue`.
- **Long task interrupted by network** → resume same session id if context still valid.
- **Subagent work** does not auto-appear in parent session — pass results explicitly.
- Do not confuse **prompt caching** breakpoints with **session resume** — different features.
- “User closed laptop mid-refactor, context still correct” → continue/resume is reasonable.

## Production Notes

- Persist **session ids** in your job queue or ticket system for support escalations.
- On branch switch or deploy, **invalidate** sessions or start fresh with a diff summary.
- Cap session length — periodic checkpoint summaries reduce context rot.
- Redact secrets from session logs; tool results may contain tokens or PII.
- In multi-tenant SaaS, **isolate** session storage per tenant — never share session ids.
- Document runbooks: when operators should `--continue` vs start clean.

## Common Mistakes

- Resuming after MCP server returned **wrong-environment** data
- Assuming `--continue` fixes bad tool *selection* (descriptions / scoping issue)
- Manually truncating history in API apps while Claude Code session still holds full state
- Treating subagent completion as automatically merged into coordinator session
- Using resume when the exam stem implies **independent review in fresh context**

## See Also

- [Agent SDK](./agent-sdk.md)
- [Memory](./memory.md)
- [Agent Loop](../04-agent-engineering/agent-loop.md)
- [Context Window](../09-performance/context-window.md)
- [Tracing](./tracing.md)
