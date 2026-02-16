---
name: agent-creator
description: Orchestrate the D.A.R.T.E. pipeline to create a new agent from concept to approved system prompt
---

# Agent Creator

Run the full D.A.R.T.E. pipeline to create an agent.

## Use When

- User wants a new agent from scratch
- You need structured planning -> writing -> validation

## Pipeline

### 1) Plan (`/prompt-architect-planner`)

- Input: initial requirements
- Output: architecture spec
- Save: `agents/[name]/architecture-spec.md`

### 2) Build (`/prompt-architect-builder`)

- Input: approved architecture spec
- Output: system prompt
- Save: `agents/[name]/system-prompt.md`

### 3) Review (`/prompt-architect-reviewer`)

- Input: architecture spec + system prompt
- Output: scenarios, evaluation, decision, fix list
- Save:
  - `agents/[name]/test-scenarios.md`
  - `agents/[name]/evaluation.md`

## Decision Loop

- **APPROVE**: done.
- **APPROVE WITH NOTES**: return to Builder, then review again.
- **REJECT**: return to Planner (architecture issue) or Builder (writing issue).

Typical loop:

```text
Planner -> Builder -> Reviewer -> Builder (if needed) -> Reviewer
```

## Quick Edit Path

For small changes to existing agents:
1. Builder for targeted edit
2. Reviewer for quick validation

## Completion Checklist

- [ ] `agents/[name]/architecture-spec.md`
- [ ] `agents/[name]/system-prompt.md`
- [ ] `agents/[name]/test-scenarios.md`
- [ ] `agents/[name]/evaluation.md`
- [ ] Deploy to `.claude/agents/[name].md` and `.opencode/agents/[name].md`
- [ ] Files in English
