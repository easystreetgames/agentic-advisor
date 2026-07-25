# Agentic AI Concepts Reference
*Last updated: 2026-07-25*

*See also: [frameworks.md](frameworks.md) · [framework-costs.md](framework-costs.md)*

---

## Core Agent Loop

The fundamental pattern underlying all agentic systems:

```
Observe → Think → Act → Observe → ...
```

**ReAct (Reason + Act)** — the dominant pattern. The LLM alternates between reasoning steps ("I need to find the current stock price") and action steps (calling a tool). Introduced by Yao et al. 2022, now standard.

**Planning → Execution** — separate the planning phase (generate a plan) from the execution phase (work through the plan). Useful when the plan needs human review. Risk: plans go stale as execution reveals new information.

**Reflection** — after acting, the agent critiques its own output before returning it. Adds latency but substantially improves quality on complex tasks.

---

## Tool Use / Function Calling

The mechanism by which LLMs take actions in the world. The model generates a structured call (name + arguments), the host executes it, the result is fed back.

**Key considerations:**
- Tool descriptions are prompt engineering — clarity here matters more than elsewhere
- Tools should be atomic and composable, not monolithic
- Return structured data the model can reason about, not raw text dumps
- Error messages from tools are read by the LLM — make them informative

**MCP (Model Context Protocol)** — Anthropic's standard for tool/resource exposure. Allows any server to expose tools, resources, and prompts to any MCP-compatible client. Growing adoption as a lingua franca for agent integrations.

---

## Memory Systems

| Type | What it is | Implementation |
|------|-----------|----------------|
| **In-context** | Information in the current context window | Concatenation, summarization |
| **Episodic** | Logs of past interactions | Vector DB retrieval |
| **Semantic** | Facts and knowledge | Knowledge graphs, vector search |
| **Procedural** | How to do things | Prompts, few-shot examples |
| **Working** | Scratch pad for current task | In-context, structured state |

**Practical note**: Most production systems use in-context + episodic (vector retrieval). Full knowledge graphs are complex; prefer structured in-context state when possible.

---

## Multi-Agent Patterns

**Supervisor / Orchestrator** — one agent routes tasks to specialized subagents. The supervisor doesn't do work directly; it delegates. Good for: parallelism, specialization.

**Pipeline / Sequential** — output of one agent is input to the next. Simple, predictable, easy to debug. Good for: ETL-style transformations, document processing.

**Peer-to-peer / Debate** — agents communicate laterally, critique each other's outputs. Good for: improving quality on subjective tasks, avoiding single-agent blindspots.

**Hierarchical** — supervisors manage sub-supervisors manage workers. Scales to complex tasks. Hard to debug.

**Swarm** — many lightweight agents with a shared goal and simple handoff rules. Good for: exploration, parallel search. Hard to control.

**Key tradeoff**: More agents = more parallelism and specialization, but also more coordination overhead, more tokens, and harder debugging. Start with the simplest topology that works.

---

## Agent Evaluation

The hardest problem in production agentic systems. Key dimensions:

**Task success rate** — did it accomplish the goal? Define this precisely (binary, rubric, human judgment).

**Trajectory quality** — did it take an efficient path, or did it waste steps / get stuck in loops?

**Tool use accuracy** — did it call the right tools with correct arguments?

**Faithfulness** — did it stick to facts, or hallucinate? Critical for agents with access to external data.

**Latency and cost** — agents amplify LLM costs. Track tokens and wall clock time per task.

**Robustness** — does performance degrade gracefully with unusual inputs, errors, or adversarial tool outputs?

**Evaluation approaches:**
- LLM-as-judge (fast, scalable, biased toward verbose outputs)
- Human evaluation (ground truth, slow)
- Unit tests on tool calls (objective, brittle)
- End-to-end regression suite on canonical tasks

---

## Agentic RAG

RAG (Retrieval Augmented Generation) + agency = the agent decides what to retrieve and when, rather than one retrieval at the start.

**Patterns:**
- **Iterative RAG** — retrieval in a loop: retrieve, reason, retrieve more if needed
- **Multi-hop RAG** — chain retrievals where each result informs the next query
- **Self-RAG** — model predicts whether retrieval is needed at all (reduces unnecessary calls)
- **HyDE** — generate a hypothetical answer, use it as the retrieval query (improves recall)

---

## Prompt Engineering for Agents

Different from single-turn prompting:

**System prompt structure matters more** — define the agent's persona, constraints, and tools clearly. Include what NOT to do.

**Tool descriptions are API contracts** — write them like docstrings. Include parameter types, what each param does, and what the tool returns.

**Chain-of-thought scaffolding** — explicitly ask the model to reason before acting. "Think step by step before calling any tools."

**Stopping conditions** — tell the agent when it's done. Without explicit stopping criteria, agents loop.

**Error recovery** — include instructions for what to do when a tool fails. Otherwise agents either give up or retry infinitely.

---

## Human-in-the-Loop (HITL)

Patterns for involving humans in agentic workflows:

**Interrupt on uncertainty** — agent pauses and asks for clarification when confidence is low
**Approval gates** — certain actions (send email, execute code, spend money) require human approval
**Review before delivery** — agent completes task, human reviews before output is finalized
**Async collaboration** — agent works, flags items, human reviews on their schedule

**Implementation**: LangGraph has native interrupt support. Most other frameworks handle this via custom control flow.

---

## Key Failure Modes

**Prompt injection** — malicious content in retrieved documents or tool outputs overrides agent instructions. Mitigate with: tool output sanitization, privileged/unprivileged context separation.

**Infinite loops** — agent retries indefinitely or oscillates between states. Mitigate with: max iteration limits, visited-state tracking.

**Context overflow** — agent loses early context as conversation grows. Mitigate with: summarization, structured state management.

**Hallucinated tool calls** — model invents tools that don't exist or arguments that don't match schema. Mitigate with: strict tool schema validation, structured output parsing.

**Cascade failures** — in multi-agent systems, one agent's error propagates. Mitigate with: error handling between agents, fallback strategies.

**Overthinking** — agent reasons in circles without acting. Mitigate with: time/step limits, "act now" prompting.

---

## Emerging Patterns (as of mid-2026)

**Agents as APIs** — wrapping agent workflows as standard HTTP endpoints for integration with existing systems

**Persistent agent state** — agents that maintain state across sessions (not just within one run)

**Agent identity** — agents with persistent identities, reputations, and learned preferences

**Multi-modal agents** — agents that see, hear, and interact with GUIs (computer use)

**Speculative execution** — run multiple agent paths in parallel, select best result

**Agent observability** — treating agent traces like distributed system traces (OpenTelemetry patterns applied to agents)
