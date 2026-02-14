# CLAUDE.md — Prompt Engineering Workspace

This workspace is dedicated to designing, creating, testing, and optimizing system prompts for LLM agents.

## Primary Agent

Use the `/prompt-architect` agent (`.claude/agents/prompt-architect.md`) for all prompt creation and optimization tasks. Switch to it when asked to design, write, review, or improve any system prompt or agent configuration.

## Core Principles

- Context is precious — every token in a prompt must earn its place
- Context Engineering > Prompt Engineering — curate optimal information, not just clever phrases
- Heuristics over hardcoded rules — give agents principles, not brittle logic
- Deliberate reasoning by default — agents should think before responding unless the task is trivially simple
- Test everything — no prompt ships without 4 test scenarios (happy path, edge case, adversarial, failure mode)
- Stay current — prompt engineering evolves fast; research recent techniques before applying unfamiliar patterns

## Workflow

Every prompt follows the D.A.R.T.E. framework: Discovery → Architecture → Redaction → Test → Enhance. Never skip phases.

## File Structure

```
prompts/agents/[name]/system-prompt.md   — The prompt
prompts/agents/[name]/test-scenarios.md  — Test cases
prompts/agents/[name]/evaluation.md      — Metrics
prompts/templates/                        — Reusable templates
```

## Conventions

- Prompt files in Markdown with semantic versioning (v1.0, v1.1)
- File names in kebab-case: `legal-analyst-agent.md`
- All prompts include: identity with cognitive bias, success criteria, reasoning strategy, output format, few-shot examples, uncertainty handling, safety guardrails
- Adapt to target model: Claude uses XML tags, GPT uses Markdown, Gemini uses heavy few-shot
- When working with an unfamiliar domain, research current best practices before generating

## Important

- NEVER generate prompts without D.A.R.T.E.
- NEVER skip safety guardrails
- ALWAYS include test scenarios
- ALWAYS include uncertainty handling
- When unsure about a model's current capabilities, check its latest documentation
- Prefer simplicity — shortest prompt that reliably achieves the goal
