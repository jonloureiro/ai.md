# Multi-Agent Patterns

> Last updated: 2026-02-14
> Status: Current

## Summary

Multi-agent architectures have matured significantly with frameworks like Google ADK, LangChain/LangGraph, and Databricks Agent Framework. Key insight: the orchestration pattern matters more than individual agent quality.

## Sources

- [Developer's guide to multi-agent patterns in ADK — Google Developers Blog](https://developers.googleblog.com/developers-guide-to-multi-agent-patterns-in-adk/)
- [Multi-agent systems — LangChain Documentation](https://python.langchain.com/docs/concepts/multi_agent/)
- [Build and deploy multi-agent AI systems — Databricks](https://www.databricks.com/product/machine-learning/build-and-deploy-multi-agent-ai-systems)

## Key Findings

### 1. Core Patterns

| Pattern | Description | Best For |
|---|---|---|
| **Sequential Pipeline** | Agent A → Agent B → Agent C. Fixed order. | Well-defined workflows with clear phase boundaries |
| **Orchestrator-Worker** | Leader agent delegates to specialized workers | Tasks with dynamic subtask decomposition |
| **Hierarchical** | Multiple levels of orchestration | Large, complex systems with clear domain boundaries |
| **Peer-to-Peer** | Agents communicate directly | Collaborative tasks without clear hierarchy |

### 2. Performance Findings

- **Leader-organized agents complete tasks ~10% faster** than flat architectures (Google ADK research).
- **Context isolation between sub-agents is critical** — shared context creates interference and confusion.
- **Stateful patterns save 40-50% of API calls** on repeated requests by caching agent state.

### 3. Context Isolation Principle

Each agent should receive ONLY the context it needs for its specific task:
- Passing the full conversation history to every agent wastes tokens and degrades performance.
- The orchestrator should summarize and filter context before passing to workers.
- Agent outputs should be structured (not free text) to enable clean handoffs.

### 4. Handoff Design

The quality of inter-agent handoffs determines system quality:
- **Structured specs** (like our architecture spec format) are better than free-text summaries.
- **Explicit assumptions** must be surfaced at handoff points.
- **Each agent should validate its input** before proceeding.

### 5. When NOT to Use Multi-Agent

- When a single prompt can handle the full task
- When the overhead of orchestration exceeds the benefit of specialization
- When context sharing between "agents" is so heavy that isolation provides no benefit

## Implications for This Workspace

- Our Planner → Builder → Reviewer pipeline is a **Sequential Pipeline** — the simplest and most appropriate pattern for our use case.
- The architecture spec serves as a structured handoff document, which aligns with best practices.
- Each agent should validate its input (Builder checks spec completeness, Reviewer checks prompt structure).
- For simple edits, skip the full pipeline — the overhead is not justified.
