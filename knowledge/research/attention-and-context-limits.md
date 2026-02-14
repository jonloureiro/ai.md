# Attention and Context Window Limits

> Last updated: 2026-02-14
> Status: Current

## Summary

Despite million-token context windows, practical model performance degrades significantly well before the theoretical limit. The "lost-in-the-middle" phenomenon and attention scarcity create effective limits far below the advertised maximum.

## Sources

- [Context engineering vs. prompt engineering — Elasticsearch Labs](https://www.elastic.co/search-labs/blog/context-engineering-vs-prompt-engineering)
- [Lost in the Middle: How Language Models Use Long Contexts — Stanford/Berkeley, 2023](https://arxiv.org/abs/2307.03172)
- [Effective context engineering for AI agents — Anthropic Engineering Blog](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)

## Key Findings

### 1. The 32K Practical Limit

Research shows model performance noticeably degrades around **32K tokens** even with context windows of 128K-1M+. This is not a hard cliff but a gradual degradation:

- **0-8K tokens**: Optimal performance. Model attends to everything effectively.
- **8K-32K tokens**: Good performance with careful placement. Critical info should be at the beginning or end.
- **32K-128K tokens**: Noticeable degradation. Information in the middle is increasingly ignored.
- **128K+ tokens**: Usable for retrieval tasks but unreliable for tasks requiring synthesis across the full context.

### 2. Lost-in-the-Middle

Models exhibit a U-shaped attention curve:
- **Strong attention**: Beginning of context (primacy effect)
- **Strong attention**: End of context (recency effect)
- **Weak attention**: Middle of context

This means:
- Critical instructions should be placed at the START of the system prompt.
- Recent, task-relevant context should be placed at the END (just before the user message).
- Reference material, examples, and supplementary info can go in the middle.

### 3. Attention Scarcity

Even within the effective context range, attention is a finite resource:
- Each additional token competes for attention with every other token.
- High-noise context (verbose instructions, irrelevant examples) dilutes attention from high-signal content.
- A shorter, higher-quality prompt often outperforms a longer, more comprehensive one.

### 4. Practical Strategies

| Strategy | Description | Token Savings |
|---|---|---|
| **Aggressive curation** | Remove any content that does not directly serve the task | 30-50% |
| **Dynamic loading** | Load context on demand instead of static inclusion | 40-70% |
| **Structured compression** | Replace verbose text with structured formats (tables, schemas) | 20-40% |
| **Example selection** | Use 2-3 high-quality examples instead of 5-10 mediocre ones | 30-50% |

## Implications for This Workspace

- System prompts we generate should target **under 4K tokens** for most agents, **under 8K** for complex agents.
- The 10-component structure should be seen as a checklist, not a mandate — skip components that add no value for the specific agent.
- The Reviewer should flag prompts that exceed 8K tokens as a potential efficiency issue.
- Few-shot examples should be 2-3 high-quality ones, not 5+ for coverage.
- Front-load identity + inviolable rules. Put examples and edge cases later.
