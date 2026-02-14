# Context Engineering

> Last updated: 2026-02-14
> Status: Current

## Summary

"Context Engineering" has replaced "Prompt Engineering" as the dominant framing for working with LLMs. The shift — driven by voices like Andrej Karpathy and Tobi Lutke — reframes the discipline from "how to talk to the model" to "what information the model has access to when it generates each token."

## Sources

- [Effective context engineering for AI agents — Anthropic Engineering Blog](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Context Engineering: The Definitive 2025 Guide — FlowHunt](https://www.flowhunt.io/blog/context-engineering/)
- [Context engineering vs. prompt engineering — Elasticsearch Labs](https://www.elastic.co/search-labs/blog/context-engineering-vs-prompt-engineering)

## Key Findings

### 1. Core Definition (Anthropic)

> "Find the smallest set of high-signal tokens that maximize the likelihood of some desired outcome."

Context is working memory. Every token spent reduces the model's attention budget for the actual task. The job is not writing clever phrases — it is curating what enters the model's attention window.

### 2. Context Engineering vs. Prompt Engineering

| Aspect | Prompt Engineering | Context Engineering |
|---|---|---|
| Focus | How you phrase the instruction | What information is available at inference time |
| Scope | The system prompt | System prompt + retrieved docs + tool results + conversation history + memory |
| Metaphor | Writing a good question | Building the right workspace for a knowledge worker |
| Key skill | Wordsmithing | Information architecture and curation |

### 3. Four Types of Context

1. **Static context**: Always present in the system prompt (identity, rules, format)
2. **Dynamic context**: Loaded at runtime (RAG results, user history, tool outputs)
3. **Ephemeral context**: Generated during the conversation (scratchpad, intermediate results)
4. **Compressed context**: Summaries of earlier conversation or documents

### 4. Practical Implications

- **Less is more**: Adding more context often degrades performance. Curate aggressively.
- **Placement matters**: Critical instructions belong at the beginning and end of the prompt (U-shaped attention curve).
- **Signal-to-noise ratio**: A 2K-token prompt with high-signal context outperforms a 10K-token prompt with diluted context.
- **Dynamic loading**: Prefer loading relevant context on demand over stuffing everything into the system prompt.

## Implications for This Workspace

- Every prompt we design should be evaluated on token efficiency, not just completeness.
- Architecture specs should explicitly define the three-tier context strategy (static, dynamic, compressed).
- "Add more context" is not a valid improvement suggestion without removing something else.
