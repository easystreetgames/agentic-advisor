# Agentic Frameworks Reference
*Last updated: 2026-07-25 — Always search for updates before answering framework questions.*

*See also: [framework-costs.md](framework-costs.md) for pricing, lock-in, and switching cost analysis*

---

## Framework Landscape Overview

The agentic framework space has fragmented into several distinct approaches. Choosing one depends heavily on your use case: single-agent vs. multi-agent, structured workflows vs. open-ended reasoning, and how much control you want over execution.

**Dominant patterns:**
- **Graph-based orchestration** (LangGraph) — explicit control flow, good for production
- **Role-based multi-agent** (CrewAI, AutoGen) — natural language team composition
- **Code-first minimal** (OpenAI Agents SDK, Pydantic AI) — low abstraction, type-safe
- **Document/RAG-centric** (Haystack) — retrieval-heavy pipelines
- **Enterprise/polyglot** (Semantic Kernel) — Microsoft ecosystem, multi-language

---

## LangGraph (LangChain)

**Status**: Production-ready, widely adopted
**Best for**: Complex, stateful workflows where you need fine-grained control over execution flow

**Strengths:**
- Explicit graph structure makes control flow auditable
- Native persistence and checkpointing (resume interrupted runs)
- Built-in streaming at every node
- Strong human-in-the-loop support
- LangSmith integration for tracing and evaluation
- Large ecosystem and community

**Weaknesses:**
- Verbose — simple tasks require more boilerplate than lighter frameworks
- Learning curve for the graph mental model
- LangChain abstraction overhead underneath

**When to use**: Multi-step pipelines where you need to retry nodes, branch on conditions, pause for human review, or resume from a checkpoint. Production workflows where observability matters.

**Key patterns**: Supervisor nodes, subgraph delegation, conditional edges, interrupt-on-human-input

---

## CrewAI

**Status**: Fast-growing, strong for multi-agent task delegation
**Best for**: Breaking a complex task into roles and letting specialized agents collaborate

**Strengths:**
- Intuitive role/goal/backstory model for defining agents
- Built-in crew orchestration (sequential, hierarchical, parallel)
- Easy to prototype multi-agent workflows quickly
- Good documentation and examples
- Growing tool ecosystem

**Weaknesses:**
- Less control over execution details than LangGraph
- Memory and state management less mature
- Can be opaque when something goes wrong inside a crew

**When to use**: Tasks that map naturally to a team structure (researcher + writer + critic), rapid prototyping of multi-agent ideas, business process automation.

---

## AutoGen (Microsoft)

**Status** (updated 2026-09-28): Maintenance only. Microsoft merged AutoGen with Semantic Kernel into **Microsoft Agent Framework 1.0** (April 2026). AutoGen gets security fixes only; new features land in Agent Framework. Do not start new projects on AutoGen. The community fork continues separately as AG2.
**Best for**: Conversational multi-agent patterns, research scenarios, code execution

**Strengths:**
- Strong support for code-executing agents (runs Python/shell)
- Flexible conversation patterns between agents
- AutoGen Studio for no-code prototyping
- Good for scenarios where agents need to iterate through code + results
- Microsoft backing means enterprise consideration

**Weaknesses:**
- v0.4 rewrite broke backward compatibility — docs/examples are inconsistent
- More complex setup than simpler frameworks
- Conversational pattern can be unpredictable

**When to use**: Data analysis workflows where agents need to write and run code, research agents that iterate, scenarios with human-in-the-loop conversation.

---

## OpenAI Agents SDK

**Status**: Official from OpenAI, production-oriented
**Best for**: Straightforward single or multi-agent workflows using OpenAI models

**Strengths:**
- Very clean, minimal API — low abstraction overhead
- Built-in tool use, handoffs between agents, and guardrails
- First-class tracing via OpenAI dashboard
- Officially maintained and well-documented

**Weaknesses:**
- Designed around OpenAI models (though other models work)
- Less community tooling than LangGraph
- Fewer built-in integrations

**When to use**: Clean production agents using OpenAI models, when you want minimal dependencies, when you want the "official" OpenAI blessed approach.

---

## Claude Agent SDK (Anthropic)

**Status**: Evolving; Claude Code itself is a flagship example
**Best for**: Agents powered by Claude models, especially with tool use and long-context tasks

**Strengths:**
- Designed for Claude's strengths (long context, strong instruction following, tool use)
- Claude Code is a production reference implementation
- Strong computer use / browser agent capabilities
- Managed Agents surface (server-managed stateful agents with Anthropic-hosted tool execution)
- Good for tasks requiring careful reasoning and minimal hallucination

**Weaknesses:**
- Less third-party tooling than OpenAI ecosystem
- SDK still evolving rapidly

**When to use**: Claude-powered agents, computer use workflows, tasks where reasoning quality is critical over raw speed.

*See also: [claude-agent-sdk.md](claude-agent-sdk.md) for a full deep-dive on surfaces, Managed Agents, tool patterns, and production guidance.*

---

## Pydantic AI

**Status**: Stable, growing adoption among Python practitioners
**Best for**: Type-safe, structured agents where you want IDE support and reliable outputs

**Strengths:**
- Full Pydantic validation for inputs and outputs
- Excellent developer experience (IDE autocomplete, type checking)
- Model-agnostic (OpenAI, Anthropic, Gemini, local models)
- Clean, minimal API without heavy abstractions
- Structured outputs as a first-class concept

**Weaknesses:**
- Less built-in orchestration than LangGraph/CrewAI
- Smaller community

**When to use**: When you need reliable structured outputs, type safety matters, or you want a minimal well-designed API. Good for production microservices.

---

## Smolagents (HuggingFace)

**Status**: Lightweight, opinionated
**Best for**: Code-first agents using HuggingFace models and tools

**Strengths:**
- Very lightweight — minimal dependencies
- Agents write and execute Python code as their primary action mode ("code agents")
- Easy to use with HuggingFace model hub
- Good for research and experimentation

**Weaknesses:**
- Less suited for complex orchestration
- Smaller ecosystem
- Code execution has security considerations

**When to use**: Research/experimentation, HuggingFace model users, when you want an agent that reasons via code.

---

## Semantic Kernel (Microsoft)

**Status**: Mature, enterprise-focused
**Best for**: Enterprise teams, especially .NET/C# shops or those in Microsoft ecosystem

**Strengths:**
- .NET first (unique — most frameworks are Python-only)
- Enterprise integrations (Azure, Microsoft 365)
- Good plugin/skill system
- Stable and well-documented

**Weaknesses:**
- Heavier than newer frameworks
- Python support is secondary to .NET
- Less cutting-edge than newer entrants

**When to use**: Enterprise .NET environments, Azure-centric deployments, teams where C# is primary.

---

## DSPy (Stanford)

**Status**: Research → production, growing interest
**Best for**: Optimizing prompts and chains automatically using examples

**Strengths:**
- "Programming not prompting" — define signatures, let DSPy optimize
- Can auto-optimize prompts given training examples
- Model-agnostic
- Novel approach to prompt engineering

**Weaknesses:**
- Different mental model — requires adjustment from traditional prompting
- Optimization runs can be slow and expensive
- Less suited for dynamic, interactive agents

**When to use**: When you have examples and want auto-optimized prompts, pipelines with well-defined input/output specs, research workflows.

---

## Agno (formerly Phidata)

**Status**: Rebranded and growing
**Best for**: Quick agent + knowledge base + UI setups

**Strengths:**
- Built-in RAG and knowledge base support
- Bundled UI (AgnoApp) for rapid demos
- Multi-modal support

**Weaknesses:**
- Less granular control than LangGraph
- Rebranding created some community confusion

---

## Framework Selection Guide

| Need | Recommended |
|------|------------|
| Production stateful pipeline | LangGraph |
| Multi-agent task delegation | CrewAI |
| Code-executing agents | AutoGen or Smolagents |
| Type-safe structured outputs | Pydantic AI |
| OpenAI-native clean API | OpenAI Agents SDK |
| Claude-powered agents | Claude Agent SDK |
| Enterprise .NET | Semantic Kernel |
| Auto-optimize prompts | DSPy |
| Fast RAG + UI prototype | Agno |
