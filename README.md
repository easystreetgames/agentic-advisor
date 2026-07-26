# Agentic Advisor — Reference Material

This repo contains a living knowledge base on agentic AI frameworks, concepts, and research. The reference files are updated continuously and are the primary thing worth reading here.

## Reference Files

Read in this order if you're building up from scratch. Skip to wherever you already are.

| Step | File | What's in it |
|------|------|-------------|
| 1 | [references/concepts.md](references/concepts.md) | Core vocabulary and patterns: ReAct loop, memory systems, multi-agent topologies, evaluation, failure modes. Start here — everything else assumes this vocabulary. |
| 2 | [references/tool-use.md](references/tool-use.md) | Deep-dive on tool use / function calling: wire protocol, schema design, parallel calls, error handling, MCP internals, security, performance, and anti-patterns. Tool use is the core primitive that all agentic frameworks are built on. |
| 3 | [references/frameworks.md](references/frameworks.md) | Status and selection guidance for 12+ frameworks: LangGraph, CrewAI, AutoGen, OpenAI Agents SDK, Claude Agent SDK, Pydantic AI, Smolagents, and more. Once you understand the primitives, pick your framework here. |
| 4 | [references/framework-costs.md](references/framework-costs.md) | Pricing, vendor lock-in, and switching cost analysis for each framework. Read alongside frameworks.md when making a build-vs-buy or commit decision. |
| 5 | [references/claude-agent-sdk.md](references/claude-agent-sdk.md) | Deep-dive on the Claude Agent SDK: API surfaces, Managed Agents architecture, MCP, thinking/effort, prompt caching, and production patterns. Read if you're building on Claude. |
| — | [references/reading-list.md](references/reading-list.md) | Curated papers, blog posts, and repos worth tracking. Not exhaustive — only things judged worth the time. |
| — | [knowledge/briefing-log.md](knowledge/briefing-log.md) | Append-only log of completed briefings. Skim this to see what topics have changed recently and when. |

## Currency

Each reference file has a `Last updated` date at the top. The field moves fast — treat framework-specific claims older than a few weeks as potentially stale. Concept and pattern files age more slowly.
