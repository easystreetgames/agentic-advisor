# Claude Agent SDK — Deep Dive
*Last updated: 2026-07-25*

*See also: [frameworks.md](frameworks.md) · [framework-costs.md](framework-costs.md) · [concepts.md](concepts.md)*

---

## What It Is

The Claude Agent SDK is Anthropic's first-party stack for building agents powered by Claude models. Everything goes through the Messages API (`POST /v1/messages`) — tool use, structured outputs, thinking, and streaming are all features of this single endpoint. Managed Agents (server-managed stateful agents) is a higher-level surface layered on top.

**Claude Code is the flagship reference implementation.** It uses adaptive thinking at `xhigh` effort, the bash/file/editor/glob/grep tool suite, the skills system, and multi-agent delegation — the same primitives exposed by the SDK.

---

## Three Surfaces

| Surface | What you get | When to use |
|---|---|---|
| **Messages API** | One request, one response | Classification, summarization, extraction, Q&A |
| **Messages API + tool use** | Agentic loop you orchestrate | Multi-step pipelines, custom tools, fine-grained control |
| **Managed Agents** | Anthropic runs the loop and container | Stateful agents with workspace, file ops, long-running tasks |

**Start simple.** Single API calls handle more use cases than people expect. Only reach for Managed Agents when the task genuinely needs open-ended model-driven exploration and Anthropic-hosted compute.

> **Cloud provider note:** Managed Agents is only available on the first-party API and Claude Platform on AWS. Amazon Bedrock and Google Vertex AI do not support it — use the Messages API + tool use on those.

---

## Current Models

| Model | ID | Context | Input $/1M | Output $/1M |
|---|---|---|---|---|
| Claude Opus 4.8 | `claude-opus-4-8` | 1M | $5.00 | $25.00 |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | 1M | $3.00 | $15.00 |
| Claude Haiku 4.5 | `claude-haiku-4-5` | 200K | $1.00 | $5.00 |

Default to Opus 4.8 for most agent work. Sonnet 4.6 is the sweet spot for high-volume production. Haiku 4.5 for subagents and latency-critical simple tasks.

---

## Tool Use

Tool use is the core agentic primitive. The model generates a structured call (name + arguments), your host executes it, the result is fed back. Everything else builds on this loop.

**Two paths:**

- **Tool runner (beta)** — the SDK handles the loop; you define tools as typed functions via decorators (Python `@beta_tool`), Zod schemas (TypeScript), or annotated classes (Java/Go). Recommended for most use cases.
- **Manual loop** — you call the API, detect `stop_reason == "tool_use"`, execute, append results, repeat. Use when you need approval gates, custom logging, or conditional execution.

**Key patterns:**
- Tool descriptions are API contracts. Write them like docstrings. Include *when* to call the tool, not just what it does — newer Opus models reach for tools less often and benefit from explicit trigger conditions.
- Return structured data the model can reason about. Error messages from tools are read by Claude — make them informative.
- Multi-tool calls: Claude can request multiple tools in one response. Handle all before continuing.

**Server-side tools** (Anthropic-hosted, no client execution):

| Tool | Type string | What it does |
|---|---|---|
| Bash | `bash_20250124` | Shell command execution |
| Text editor | `text_editor_20250728` | File read/edit (str_replace_based_edit_tool) |
| Web search | `web_search_20260209` | Live search with dynamic filtering |
| Web fetch | `web_fetch_20260209` | Retrieve URL content |
| Code execution | `code_execution_20260120` | Sandboxed Python container |
| Memory | `memory_20250818` | Cross-session file-based memory |

---

## Thinking & Effort

**Adaptive thinking** (`thinking: {type: "adaptive"}`) is the default for intelligent agent work on Opus 4.6+. Claude decides when and how much to think. Do not use fixed `budget_tokens` on Opus 4.7/4.8 — it returns a 400.

**Effort** (`output_config: {effort: "low"|"medium"|"high"|"xhigh"|"max"}`) controls thinking depth and overall token spend. Default is `high`. Use `xhigh` for most coding and agentic work (it's the Claude Code default). Use `max` when correctness matters more than cost.

| Effort | Use when |
|---|---|
| `low` | Subagents, simple tasks, latency-critical paths |
| `medium` | Balanced cost/quality |
| `high` | Most intelligence-sensitive work |
| `xhigh` | Coding, agentic loops (Opus 4.7/4.8 only) |
| `max` | Highest stakes, cost insensitive (Opus-tier only) |

For Opus 4.8: at `xhigh`/`max`, set `max_tokens` ≥ 64K to give the model room to think and act across tool calls.

---

## Managed Agents

### Core concepts

Four resources, always in this order:

1. **Agent** — persisted, versioned config: model, system prompt, tools, MCP servers, skills. **Create once, reference by ID.** Never `agents.create()` in the hot path.
2. **Environment** — container template (networking, packages). Reusable across agents.
3. **Session** — one run of an agent in an environment. Produces an event stream.
4. **Events** — SSE stream of what the agent is doing; you send user messages and tool results in.

```
Agent (config) → Session (per run) → Events (stream in/out)
                    ↓
Environment → Container (tool execution workspace)
```

### Mandatory flow

```python
# ONE-TIME SETUP — store these IDs
environment = client.beta.environments.create(name="...", config={"type": "cloud", "networking": {"type": "unrestricted"}})
agent = client.beta.agents.create(name="...", model="claude-opus-4-8", tools=[{"type": "agent_toolset_20260401"}])

# EVERY RUN
session = client.beta.sessions.create(agent=agent.id, environment_id=environment.id)
client.beta.sessions.events.send(session.id, events=[{"type": "user.message", "content": [{"type": "text", "text": "..."}]}])

with client.beta.sessions.events.stream(session.id) as stream:
    for event in stream:
        if event.type == "session.status_terminated": break
        if event.type == "session.status_idle" and event.stop_reason.type != "requires_action": break
```

**Stream-first.** Open the SSE stream before sending the kickoff event — the stream only delivers events emitted after it opens.

### Tools in Managed Agents

Three kinds declared on the agent:

| Type | What runs it | Use for |
|---|---|---|
| `agent_toolset_20260401` | Anthropic (in the session container) | bash, read, write, edit, glob, grep, web_search, web_fetch |
| `mcp_toolset` | Anthropic (via MCP server proxy) | Third-party integrations |
| `custom` | Your application | Business logic, auth-required APIs |

For custom tools: agent fires `agent.custom_tool_use`, session goes idle, you send back `user.custom_tool_result`.

### MCP integration

MCP (Model Context Protocol) is Anthropic's standard for tool/resource exposure. Any MCP-compatible server can expose tools to Claude.

In Managed Agents: declare the MCP server on the agent (no auth there), put credentials in a vault, attach the vault to the session.

```python
agent = client.beta.agents.create(
    model="claude-opus-4-8",
    mcp_servers=[{"type": "url", "name": "github", "url": "https://api.githubcopilot.com/mcp/"}],
    tools=[
        {"type": "agent_toolset_20260401"},
        {"type": "mcp_toolset", "mcp_server_name": "github"},
    ],
)
session = client.beta.sessions.create(agent=agent.id, environment_id=env.id, vault_ids=[vault.id])
```

Credentials never enter the container — Anthropic's proxy injects them after requests leave the sandbox.

In the Messages API directly: use `mcp_servers` param (beta) to connect Claude to remote MCP servers inline.

### Skills

Skills are reusable, filesystem-based resources that give agents domain-specific knowledge: workflows, context, best practices. They load on demand when the task is relevant.

Pre-built Anthropic skills: `xlsx`, `docx`, `pptx`, `pdf`.

```python
agent = client.beta.agents.create(
    skills=[
        {"type": "anthropic", "skill_id": "xlsx"},
        {"type": "custom", "skill_id": "skill_abc123", "version": "latest"},
    ]
)
```

Claude Code's own skills system is the same mechanism — each skill is a folder with a `SKILL.md`.

### Memory

Two patterns for persistence:

- **Memory tool** (`memory_20250818`) — Claude reads/writes files in a `/memories` directory. Cross-session via your implementation. Available in the Messages API.
- **Memory stores** (Managed Agents) — workspace-scoped persistent stores mounted as `/mnt/memory/<name>/` in the container. Attach via `resources` at session creation. Survives across sessions. Includes audit trail (memory versions) and optimistic concurrency.

### Multi-agent

A coordinator agent can delegate to other agents within one session. Each subagent runs in its own thread with isolated conversation history but shared container filesystem.

```python
orchestrator = client.beta.agents.create(
    multiagent={"type": "coordinator", "agents": [reviewer.id, test_writer.id]},
)
```

Max 20 agents in roster, 25 concurrent threads, 1 level of delegation.

---

## Human-in-the-Loop

**Approval gates** via permission policies on tools:

```json
{"type": "agent_toolset_20260401", "configs": [{"name": "bash", "permission_policy": {"type": "always_ask"}}]}
```

When triggered: agent emits `agent.tool_use` event, session goes idle, you respond with `user.tool_confirmation` (allow or deny).

**Outcomes** (Managed Agents) — define what "done" looks like as a gradeable rubric. The harness runs iterate → grade → revise until the rubric is satisfied or `max_iterations` is reached.

```python
client.beta.sessions.events.send(session.id, events=[{
    "type": "user.define_outcome",
    "description": "Build a DCF model for Costco in .xlsx",
    "rubric": {"type": "text", "content": RUBRIC_MD},
    "max_iterations": 5,
}])
```

---

## Prompt Caching

Caching is a prefix match. Any byte change anywhere invalidates everything after it. Render order: `tools → system → messages`.

Key rules for agents:
- Keep the system prompt frozen. Don't interpolate timestamps or per-user IDs into it.
- Don't change tools mid-conversation — adding/removing tools renders at position 0 and invalidates everything.
- Switching models mid-session also invalidates the cache.
- Use top-level `cache_control: {"type": "ephemeral"}` on `messages.create()` to auto-place the breakpoint at the last cacheable block.

Verify with `response.usage.cache_read_input_tokens`. If it's 0 across repeated identical-prefix requests, a silent invalidator is at work.

---

## Compaction

For long-running conversations that may exceed the 1M context window, use server-side compaction. The API summarizes earlier context when it approaches the trigger threshold.

```python
client.beta.messages.create(
    betas=["compact-2026-01-12"],
    context_management={"edits": [{"type": "compact_20260112"}]},
    messages=messages,
)
# Append response.content (not just the text) — compaction blocks must be preserved
messages.append({"role": "assistant", "content": response.content})
```

Available on Opus 4.8, 4.7, 4.6, and Sonnet 4.6. Managed Agents handles this automatically.

---

## Computer Use

Claude can interact with GUIs — screenshots, mouse, keyboard. Two modes:

- **Server-hosted** — Anthropic runs the environment. Declare the tool, Claude does the rest.
- **Self-hosted** — You provide the desktop environment and execute actions client-side. Best for internal tools, sensitive environments.

Computer use on Opus 4.7/4.8 supports up to 2576px long edge. Send screenshots at 1080p for a good performance/cost balance. Coordinates are 1:1 with actual pixels — no scale-factor math needed.

---

## Production Patterns

**Context management in long runs:**
- Context editing — clears stale tool results and thinking blocks as the conversation grows. Prunes, doesn't summarize.
- Compaction — summarizes when approaching the window limit.
- Memory (file-based or stores) — cross-session persistence.

**Cost control:**
- `effort: "medium"` is often the best cost/quality balance.
- Task Budgets (`output_config: {task_budget: {type: "tokens", total: N}}`) tells the model how many tokens it has for a full loop — it self-moderates (Opus 4.7/4.8, beta).
- Prompt caching for shared context. Break-even at 2 requests with 5-min TTL.
- Haiku 4.5 for subagents on simple tasks.

**Reliability:**
- Set `max_tokens` generously (≥16K for non-streaming, ≥64K for streaming). Truncation mid-thought is worse than slightly higher costs.
- Iteration limits — always cap agentic loops. The API's `pause_turn` stop reason fires when server-side sampling hits its iteration cap; re-send to continue.
- Tool descriptions with explicit trigger conditions reduce under-use on Opus 4.8 (which reaches for tools more conservatively).

**Observability:**
- `span.model_request_end` events include `model_usage` for per-turn cost tracking.
- Managed Agents sessions are inspectable in the Console at `platform.claude.com/workspaces/default/sessions/{id}`.
- Webhooks (`managed-agents-2026-04-01`) for async session state notifications.

---

## SDK Availability

| Language | Messages API | Tool runner | Managed Agents |
|---|---|---|---|
| Python | ✅ | ✅ (beta) | ✅ |
| TypeScript | ✅ | ✅ (beta) | ✅ |
| Go | ✅ | ✅ (beta) | ✅ |
| Java | ✅ | ✅ (beta) | ✅ |
| Ruby | ✅ | ✅ (beta) | ✅ |
| C# | ✅ | ✅ (beta) | ✅ |
| PHP | ✅ | ✅ (beta) | ✅ |

`pip install anthropic` / `npm install @anthropic-ai/sdk`

The Anthropic CLI (`ant`) handles the control plane: create/update agents and environments from version-controlled YAML, inspect sessions, stream events from the terminal. Use the SDK for the data plane (sessions, events in application code).

---

## When to Use Claude Agent SDK vs Other Frameworks

Use Claude Agent SDK when:
- You're committed to Claude models (this is the strongest signal)
- You want Anthropic's first-party abstractions (Managed Agents, skills, outcomes, memory stores)
- Computer use or long-context tasks are central
- You want minimal dependencies and the "official" path

Consider alternatives when:
- You need LangGraph's graph-based orchestration and checkpointing
- You need CrewAI's role-based multi-agent composition
- You want model-agnostic tooling (Pydantic AI)
- You're in the .NET ecosystem (Semantic Kernel)

For framework cost/lock-in analysis, see [framework-costs.md](framework-costs.md).
