# CLAUDE.md — Prompt Engineering Workspace

See `AGENTS.md` for complete workspace instructions (shared across all AI coding assistants).

## Agents

- `/prompt-architect` — Main entry point. Use for quick edits, workspace maintenance, and orchestrating the pipeline.
- `/prompt-architect-planner` — Discovery and Architecture phases. Use when starting a new agent design.
- `/prompt-architect-builder` — Redaction phase. Use when writing or editing system prompts.
- `/prompt-architect-reviewer` — Test and Enhance phases. Use when reviewing or validating prompts.

## Skills

- `/agent-creator` — Orchestrate the D.A.R.T.E. pipeline to create a new agent from concept to approved system prompt
- `/skill-creator-darte` — Orchestrate the D.A.R.T.E. pipeline to create a new skill. Integrates with external `/skill-creator` for detailed implementation guidance

## Quick Start

For full new agent creation, use `/agent-creator`

For creating skills, use `/skill-creator-darte`

For detailed skill implementation guidance (anatomy, progressive disclosure, packaging), use the external `/skill-creator`

For quick edits or conversational guidance, invoke `/prompt-architect`.
