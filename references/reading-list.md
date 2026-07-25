# Agentic AI Reading List
*Last updated: 2026-07-25 — Search for newer resources before recommending.*

*See also: [frameworks.md](frameworks.md) · [framework-costs.md](framework-costs.md)*

---

## Foundational Papers

**ReAct: Synergizing Reasoning and Acting in Language Models** (Yao et al., 2022)
- The paper that established the dominant agent pattern
- [arxiv.org/abs/2210.03629](https://arxiv.org/abs/2210.03629)

**Toolformer: Language Models Can Teach Themselves to Use Tools** (Schick et al., 2023)
- How models learn tool use; background for understanding function calling
- [arxiv.org/abs/2302.04761](https://arxiv.org/abs/2302.04761)

**Chain-of-Thought Prompting Elicits Reasoning in Large Language Models** (Wei et al., 2022)
- Foundation for why step-by-step reasoning improves agent performance
- [arxiv.org/abs/2201.11903](https://arxiv.org/abs/2201.11903)

**Reflexion: Language Agents with Verbal Reinforcement Learning** (Shinn et al., 2023)
- Agents that learn from their own failed attempts via reflection
- [arxiv.org/abs/2303.11366](https://arxiv.org/abs/2303.11366)

**Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection** (Asai et al., 2023)
- Adaptive retrieval — model decides when retrieval is needed
- [arxiv.org/abs/2310.11511](https://arxiv.org/abs/2310.11511)

---

## Practical Guides & Architecture

**Anthropic: Building Effective Agents**
- Anthropic's own guide on agentic patterns, when to use agents vs. simpler solutions
- [anthropic.com/research/building-effective-agents](https://www.anthropic.com/research/building-effective-agents)

**LangChain: LangGraph Documentation**
- Official docs for the most widely-used production graph framework
- [langchain-ai.github.io/langgraph](https://langchain-ai.github.io/langgraph/)

**OpenAI Agents SDK Documentation**
- Official docs for OpenAI's agentic framework
- [openai.github.io/openai-agents-python](https://openai.github.io/openai-agents-python/)

---

## Evaluation

**AgentBench: Evaluating LLMs as Agents** (Liu et al., 2023)
- Benchmark suite for agent capabilities across diverse tasks
- [arxiv.org/abs/2308.03688](https://arxiv.org/abs/2308.03688)

**WebArena: A Realistic Web Environment for Building Autonomous Agents** (Zhou et al., 2023)
- Benchmark for web-based agents; good reference for realistic task difficulty
- [arxiv.org/abs/2307.13854](https://arxiv.org/abs/2307.13854)

---

## Blogs & Newsletters Worth Following

**Lilian Weng's Blog (lilianweng.github.io)**
- Deep technical posts on agents, RAG, and LLM internals; consistently excellent
- Posts to know: "LLM Powered Autonomous Agents" (2023)

**Simon Willison's Weblog (simonwillison.net)**
- Practical takes on using LLMs and agents; good signal-to-noise ratio

**The Gradient (thegradient.pub)**
- Research-forward coverage of ML and AI, including agents

**Interconnects (interconnects.ai) — Nathan Lambert**
- Focused on RLHF, alignment, and the frontier model landscape

---

## Key GitHub Repos to Watch

- **langchain-ai/langgraph** — LangGraph framework
- **microsoft/autogen** — AutoGen multi-agent framework
- **crewAIInc/crewAI** — CrewAI framework
- **openai/openai-agents-python** — OpenAI Agents SDK
- **pydantic/pydantic-ai** — Pydantic AI framework
- **huggingface/smolagents** — Smolagents
- **anthropics/anthropic-sdk-python** — Claude SDK
- **modelcontextprotocol/specification** — MCP standard

---

## To Research
*Topics to look up next time the knowledge base is updated:*
- Latest SWE-bench and similar coding agent benchmarks
- Current state of computer use / browser agent frameworks
- Agent memory systems — new approaches beyond basic vector retrieval
- Multi-agent coordination protocols emerging post-MCP
