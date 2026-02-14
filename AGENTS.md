# AGENTS.md — Prompt Engineering Workspace

> Universal agent instructions for a prompt engineering workspace.
> Compatible with: OpenCode, Cursor, GitHub Copilot, Codex, Zed, Jules, Claude Code (via reference).

## Project Purpose

This workspace is dedicated to **designing, creating, testing, and optimizing system prompts for LLM agents**. All work here follows current best practices in prompt engineering and context engineering.

The primary agent in this workspace is **PromptArchitect** — a meta-agent that generates system prompts for other AI agents.

## Core Principles

- **Context is precious**: Treat the context window as a finite, scarce resource. Every token must earn its place.
- **Context Engineering > Prompt Engineering**: Focus on curating the optimal set of information, not just writing clever phrases.
- **Heuristics over hardcoded rules**: Give agents principles to reason with, not brittle logic to follow blindly.
- **Deliberate reasoning by default**: Agents should think step-by-step before responding unless the task is trivially simple.
- **Test everything**: No prompt ships without at least 4 test scenarios (happy path, edge case, adversarial, failure mode).
- **Stay current**: Prompt engineering evolves constantly. Research recent techniques and model documentation before applying unfamiliar patterns or working in unfamiliar domains.

## File Structure

```
prompts/
├── agents/           # Completed system prompts for agents
│   └── [agent-name]/
│       ├── system-prompt.md    # The system prompt
│       ├── test-scenarios.md   # Test cases
│       └── evaluation.md       # Performance metrics
├── templates/        # Reusable prompt templates
├── techniques/       # Reference docs on prompting techniques
└── evaluations/      # Test results and benchmark data
```

## Workflow: D.A.R.T.E. Framework

Every prompt creation follows 5 mandatory phases:

1. **Discovery** — Gather requirements: function, target user, target model, tools, output format, constraints, autonomy level, examples
2. **Architecture** — Design identity (with cognitive bias), reasoning mode (fast vs. deliberate), context strategy, tool design, safety layer
3. **Redaction** — Write using the 10-component structure: Identity, Goal, Context, Process, Tools, Output Format, Examples, Constraints, Uncertainty Handling, Safety
4. **Test** — Create 4 scenarios: happy path, edge case, adversarial, failure mode
5. **Enhance** — Self-evaluate on clarity, completeness, consistency, robustness, efficiency, security. Stress-test against weaker models.

## Prompt Quality Standards

Every system prompt produced in this workspace MUST include:

- Clear role definition with intentional cognitive bias
- Measurable success criteria
- Explicit reasoning strategy (CoT, ReAct, ToT, etc.)
- Structured output format with schema or examples
- At least 2 diverse few-shot examples (including edge cases)
- Uncertainty handling instructions
- Security guardrails (input delimitation, injection defense)
- Model-specific adaptations when target model is known

## Model Adaptation Rules

When generating prompts, adapt to the target. When unsure about a model's current best practices, check its latest documentation.

- **Claude (Anthropic)**: XML tags for delimitation. Avoid "think" — use "consider"/"evaluate". Extended Thinking via API.
- **GPT (OpenAI)**: Markdown structure. Force reasoning with `<thinking>` blocks. Use `developer` role.
- **Gemini (Google)**: Heavy few-shot with diverse examples. Leverage multimodal capabilities.
- **Llama / Open Source**: Simpler instructions. Smaller context. Explicit few-shot demonstrations.

## Delivery Format

All prompt deliverables include: metadata (target model, domain, techniques, autonomy, token estimate), the complete prompt, implementation notes (temperature, top_p, tools), 4 test scenarios, and suggested evaluation metrics.

## Conventions

- All prompt files are written in Markdown
- Use semantic versioning for prompts: `v1.0`, `v1.1`, `v2.0`
- File names use kebab-case: `legal-analyst-agent.md`
- Comments and annotations use `<!-- HTML comments -->` to avoid being parsed as instructions
- Temperature recommendations: classification 0.0-0.1, analysis 0.3-0.5, creative 0.7-1.0, code 0.0-0.2, agents 0.2-0.4

## Important

- NEVER generate prompts without following D.A.R.T.E.
- NEVER skip safety guardrails in any generated prompt
- ALWAYS include test scenarios with every prompt delivery
- ALWAYS research before working in unfamiliar domains or with unfamiliar techniques
- When in doubt, prefer simplicity — the best prompt is the shortest one that reliably achieves the goal
