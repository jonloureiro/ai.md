# Multi-Agent Patterns

> Last updated: 2026-02-15
> Status: Current

## Summary

Multi-agent architectures have matured significantly with frameworks like Google ADK, LangChain/LangGraph, and Databricks Agent Framework. Key insight: the orchestration pattern matters more than individual agent quality. The orchestrator pattern is now the dominant approach for complex agent systems.

## Sources

- [Developer's guide to multi-agent patterns in ADK — Google Developers Blog](https://developers.googleblog.com/developers-guide-to-multi-agent-patterns-in-adk/)
- [Multi-agent systems — LangChain Documentation](https://python.langchain.com/docs/concepts/multi_agent/)
- [Build and deploy multi-agent AI systems — Databricks](https://www.databricks.com/product/machine-learning/build-and-deploy-multi-agent-ai-systems)
- [How we built our multi-agent research system — Anthropic](https://www.anthropic.com/engineering/multi-agent-research-system)
- [Orchestrating multiple agents — OpenAI Agents SDK](https://openai.github.io/openai-agents-python/multi_agent/)
- [AI Agent Orchestration Patterns — Azure Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/ai-agent-design-patterns)

## Key Findings

### 1. Core Patterns

| Pattern | Description | Best For |
|---|---|---|
| **Sequential Pipeline** | Agent A → Agent B → Agent C. Fixed order. | Well-defined workflows with clear phase boundaries |
| **Orchestrator-Worker** | Leader agent delegates to specialized workers | Tasks with dynamic subtask decomposition |
| **Hierarchical** | Multiple levels of orchestration | Large, complex systems with clear domain boundaries |
| **Peer-to-Peer** | Agents communicate directly | Collaborative tasks without clear hierarchy |

### 2. Orchestrator Pattern Deep Dive

The orchestrator is NOT a simple router. Key differences:

| Aspect | Router | Orchestrator |
|---|---|---|
| **Scope** | Passive routing only | Full lifecycle (plan, delegate, integrate, track) |
| **State** | Stateless | Maintains execution state and context |
| **Results** | Passes control entirely | Receives results and synthesizes |
| **Adaptation** | Fixed rules | Adjusts strategy mid-flight based on outcomes |

**Four responsibilities of an orchestrator:**
1. **Delegation** — Decompose tasks, choose agents, parallelize when possible
2. **Context curation** — Filter/summarize what each sub-agent receives (not full history)
3. **State tracking** — Track completion, handle failures, maintain continuity
4. **Result integration** — Validate outputs, synthesize partial results, decide if more work needed

**Handoff patterns:**
- **Manager pattern** (recommended): Orchestrator invokes sub-agent as tool → sub-agent returns result → orchestrator integrates. Orchestrator retains control.
- **Handoff pattern** (decentralized): Agent A transfers full control to Agent B with complete history. One agent owns conversation at a time. Better for customer support scenarios.

### 3. Performance Findings

- **Leader-organized agents complete tasks ~10% faster** than flat architectures (Google ADK research).
- **Context isolation between sub-agents is critical** — shared context creates interference and confusion.
- **Stateful patterns save 40-50% of API calls** on repeated requests by caching agent state.
- **Multi-agent systems use ~15× more tokens** than single-agent chats (Anthropic). Context curation is where you recoup cost.

### 4. Context Isolation Principle

Each agent should receive ONLY the context it needs for its specific task:
- Passing the full conversation history to every agent wastes tokens and degrades performance.
- The orchestrator should summarize and filter context before passing to workers.
- Agent outputs should be structured (not free text) to enable clean handoffs.
- Anthropic's approach: sub-agents return distilled findings, not raw data. Lead agent compresses into Memory.

### 5. Handoff Design

The quality of inter-agent handoffs determines system quality:
- **Structured specs** (like our architecture spec format) are better than free-text summaries.
- **Explicit assumptions** must be surfaced at handoff points.
- **Each agent should validate its input** before proceeding.

### 6. When NOT to Use Multi-Agent

- When a single prompt can handle the full task
- When the overhead of orchestration exceeds the benefit of specialization
- When context sharing between "agents" is so heavy that isolation provides no benefit

## Implications for This Workspace

- Our Planner → Builder → Reviewer pipeline is a **Sequential Pipeline** — appropriate for full agent creation.
- A new **Orchestrator agent** (`prompt-architect`) can serve as entry point, delegating to specialists when needed.
- **Hybrid model recommended**: Keep planner/builder/reviewer as both agents (direct invocation) AND subagents (invocable by orchestrator). Maximum flexibility.
- The architecture spec serves as a structured handoff document, which aligns with best practices.
- For simple edits, the orchestrator handles directly — no pipeline overhead.
