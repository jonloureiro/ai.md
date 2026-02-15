---
name: agent-creator
description: Orchestrate the D.A.R.T.E. pipeline to create a new agent from concept to approved system prompt
---

# Agent Creator

Orchestrate the full D.A.R.T.E. pipeline for creating a new agent. Follow the steps in order.

## Prerequisites

- Clear idea of what the agent should do (can be rough)
- Know which model will run it, or let the Planner decide

## Step 1: Plan

Invoke `/prompt-architect-planner` with your initial requirements.

**Input**: Your description of what the agent should do. The Planner will ask for what's missing.

**Output**: Complete architecture spec with requirements summary, identity design, reasoning strategy, context strategy, tool design, safety layer, few-shot strategy, and success criteria.

**Save to**: `agents/[agent-name]/architecture-spec.md`

Proceed when the spec is delivered and you approve it. Do not skip to Step 2 incomplete.

---

## Step 2: Build

Invoke `/prompt-architect-builder` with the architecture spec from Step 1.

**Input**: The complete architecture spec.

**Output**: Complete system prompt with metadata, implementation notes, and the prompt itself.

**Save to**: `agents/[agent-name]/system-prompt.md`

Proceed when delivered. Initial read-through is sufficient — the Reviewer will validate.

---

## Step 3: Review

Invoke `/prompt-architect-reviewer` with the system prompt (Step 2) and architecture spec (Step 1).

**Input**: Complete system prompt AND architecture spec.

**Output**: Test scenarios (4), evaluation scores (7 dimensions), decision (approve/reject), and improvement recommendations.

**Save to**:
- `agents/[agent-name]/test-scenarios.md`
- `agents/[agent-name]/evaluation.md`

**Outcomes**:
- **APPROVE**: Save. Prompt is ready.
- **APPROVE WITH NOTES**: Save, then return to Builder (Step 2) with requested improvements.
- **REJECT**: If architectural issues, return to Planner (Step 1). If writing issues, return to Builder (Step 2).

---

## Iteration

Most agents require 1-2 iterations before approval. Typical flow:

```
Planner -> Builder -> Reviewer -> [APPROVE WITH NOTES] -> Builder -> Reviewer -> [APPROVE]
```

More than 3 iterations indicates a fundamental architecture issue. Return to Planner.

---

## Quick Edit Shortcut

For small changes to existing approved prompts:
1. Go directly to Builder with the specific change
2. Then Reviewer for quick review (not full pipeline)

---

## Final Checklist

Before considering complete:

- [ ] Architecture spec: `agents/[name]/architecture-spec.md`
- [ ] System prompt: `agents/[name]/system-prompt.md`
- [ ] Test scenarios: `agents/[name]/test-scenarios.md`
- [ ] Evaluation: `agents/[name]/evaluation.md`
- [ ] Deployed to `.claude/agents/[name].md`
- [ ] Deployed to `.opencode/agents/[name].md`
- [ ] All files in English
- [ ] Version assigned (v1.0 for first version)
