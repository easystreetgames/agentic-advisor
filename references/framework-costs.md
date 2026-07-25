# Framework Costs & Commitments
*Last updated: 2026-07-25*

All frameworks listed here are open source and free to run. The real cost question is about vendor lock-in, optional platform fees, and migration overhead.

---

## Financial Costs

| Framework | Free to Run | Optional Paid Tier | LLM Lock-in |
|-----------|-------------|-------------------|-------------|
| **LangGraph** | Yes | LangSmith (~$39/mo for teams), LangGraph Platform (enterprise) | None — model-agnostic |
| **CrewAI** | Yes | CrewAI+ cloud platform (tiered pricing) | None |
| **AutoGen** | Yes | Azure AI costs if using Azure OpenAI | Soft OpenAI/Azure bias |
| **OpenAI Agents SDK** | Yes | None | Strong OpenAI API dependency |
| **Claude Agent SDK** | Yes | None | Strong Anthropic API dependency |
| **Pydantic AI** | Yes | None | None — fully model-agnostic |
| **Smolagents** | Yes | HuggingFace Inference API (free tier limited) | HuggingFace bias, but avoidable |
| **Semantic Kernel** | Yes | Azure AI costs if Azure-hosted | Soft Azure/OpenAI bias |
| **DSPy** | Yes | None | None |
| **Agno** | Yes | Agno Cloud (pricing unclear) | None |

---

## Strategic Commitments

### High switching cost

- **LangGraph** — the graph mental model seeps into your architecture. Nodes, edges, checkpoints, state schemas — once you've built on this, migrating is a rewrite. LangSmith tracing becomes sticky fast.
- **OpenAI Agents SDK / Claude Agent SDK** — handoff patterns and guardrails are provider-specific. Swapping LLM providers means rethinking agent boundaries, not just a config change.

### Medium switching cost

- **CrewAI** — role/crew abstractions are opinionated enough that porting to another framework isn't trivial, but the concepts translate
- **Semantic Kernel** — Azure/M365 integration depth is the lock-in vector, not the framework itself
- **AutoGen** — the v0.4 rewrite already proved they'll break you; factor in upgrade churn

### Low switching cost

- **Pydantic AI** — minimal abstractions, type-safe interfaces, fully model-agnostic. Easiest to migrate from or wrap
- **DSPy** — the optimization artifact (compiled prompts) is portable; the paradigm is the commitment, not the framework
- **Smolagents** — lightweight by design; small surface area to escape from

---

## Practical Takeaway

The real cost is almost never the license — it's LangSmith/LangGraph Platform creep if you go deep on LangChain's ecosystem, and API vendor lock-in if you commit hard to OpenAI Agents SDK or Claude Agent SDK without abstraction layers. Pydantic AI has the most favorable cost/commitment profile if you want optionality.

---

*See also: [frameworks.md](frameworks.md) for technical strengths/weaknesses per framework · [claude-agent-sdk.md](claude-agent-sdk.md) for Claude Agent SDK deep-dive*
