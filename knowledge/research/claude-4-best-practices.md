# Claude 4.x Prompting Best Practices

> Last updated: 2026-02-14
> Status: Current

## Summary

Claude 4.x (Opus 4.6, Sonnet 4.5, Haiku 4.5) introduced significant changes from Claude 3.x that affect how system prompts should be written. Several patterns that worked for older models are now counterproductive.

## Sources

- [Prompting best practices — Claude 4 API Docs](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/claude-4-best-practices)
- [Extended thinking / Adaptive thinking — Claude API Docs](https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking)

## Key Findings

### 1. Breaking Changes from Claude 3.x

| Feature | Claude 3.x | Claude 4.x |
|---|---|---|
| Extended thinking | `extended_thinking` parameter | Replaced by **adaptive thinking** — model decides when to think deeply |
| Prefilled responses | Supported (`assistant` turn prefill) | **Discontinued** — do not use |
| System prompt sensitivity | Moderate | **High** — aggressive instructions cause overtriggering |
| Instruction following | Sometimes ignores nuance | Very literal — means what you write, so write carefully |

### 2. System Prompt Sensitivity

Claude 4.x is significantly more responsive to system prompt instructions than previous generations. This means:

- **Aggressive language backfires**: Phrases like "CRITICAL: You MUST ALWAYS..." cause the model to overtrigger on edge cases. The model tries so hard to comply that it applies rules where they do not belong.
- **Proportional emphasis**: Use emphasis (caps, bold, "important") sparingly and only for genuinely critical instructions. If everything is critical, nothing is.
- **Trust the model more**: Claude 4.x understands nuance better. You can use natural language and heuristics instead of rigid rule lists.

### 3. Adaptive Thinking

- The model now autonomously decides when to engage deep reasoning (formerly extended thinking).
- You do not need to explicitly prompt "think step by step" for most tasks — the model does this when needed.
- For complex tasks where you want to ensure deep reasoning, you can still use structured CoT, but keep it as guidance rather than a mandate.

### 4. Practical Prompting Patterns

**Do:**
- Use natural, clear language
- Give principles and heuristics rather than exhaustive rules
- Provide examples for complex or ambiguous behaviors
- Front-load the most important instructions
- Use consistent formatting (pick one delimiter style)

**Do not:**
- Use ALL CAPS for every instruction
- Prefix everything with "IMPORTANT:", "CRITICAL:", "YOU MUST"
- Write long lists of edge case rules (use examples instead)
- Use prefilled assistant responses
- Assume patterns from Claude 3.x still apply

### 5. Instruction Hierarchy

Claude 4.x respects instruction hierarchy more reliably:
1. System prompt (highest priority)
2. Conversation history
3. User message (current turn)

This means system prompt instructions genuinely override user attempts to change behavior — but also means poorly written system prompts have outsized negative effects.

## Implications for This Workspace

- The Builder should default to natural language over aggressive formatting.
- Emphasis markers should be reserved for 2-3 genuinely critical rules per prompt, not sprinkled everywhere.
- The Reviewer should flag overtriggering risk when reviewing prompts with excessive emphasis.
- Prefilled responses should never appear in any prompt we generate.
- Architecture specs should note when adaptive thinking is sufficient vs. when explicit CoT is needed.
