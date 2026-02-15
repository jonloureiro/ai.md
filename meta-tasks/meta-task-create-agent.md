# Meta-Task: Create a New Agent

> Use this template to orchestrate the full D.A.R.T.E. pipeline for creating a new agent prompt. Follow the steps in order. Each step maps to one of the three PromptArchitect agents.

## Prerequisites

Before starting, you need:
- A clear idea of what the agent should do (even if rough)
- Knowledge of which model will run the agent (or a willingness to let the Planner decide)

## Pipeline

### Step 1: Plan (prompt-architect-planner)

Switch to the `prompt-architect-planner` agent and provide your initial requirements.

**Input**: Your description of what the agent should do. Include as much or as little as you know — the Planner will ask for what is missing.

**Expected Output**: A complete architecture spec in the standard format (requirements summary, identity design, reasoning strategy, context strategy, tool design, safety layer, few-shot strategy, success criteria).

**When to proceed**: When the Planner delivers the architecture spec AND you have reviewed and approved it. Do not skip to Step 2 with an incomplete spec.

**Save the output to**: `agents/[agent-name]/architecture-spec.md`

---

### Step 2: Build (prompt-architect-builder)

Switch to the `prompt-architect-builder` agent and provide the architecture spec from Step 1.

**Input**: The complete architecture spec. Copy-paste or reference the file.

**Expected Output**: A complete system prompt with metadata, implementation notes, and the prompt itself in the standard delivery format.

**When to proceed**: When the Builder delivers the system prompt AND you have done an initial read-through. You do not need to approve it — that is the Reviewer's job.

**Save the output to**: `agents/[agent-name]/system-prompt.md`

---

### Step 3: Review (prompt-architect-reviewer)

Switch to the `prompt-architect-reviewer` agent and provide the system prompt from Step 2, along with the architecture spec from Step 1 for reference.

**Input**: The complete system prompt AND the architecture spec.

**Expected Output**: Test scenarios (4), evaluation scores (7 dimensions), decision (approve/reject), and improvement recommendations.

**Possible outcomes**:

- **APPROVE**: Save the test scenarios and evaluation. The prompt is ready.
- **APPROVE WITH NOTES**: Save everything. Apply the suggested improvements by going back to the Builder (Step 2) with the specific changes requested.
- **REJECT**: Review the rejection reasons. If the issues are architectural, go back to the Planner (Step 1). If the issues are in the writing, go back to the Builder (Step 2).

**Save the output to**:
- `agents/[agent-name]/test-scenarios.md`
- `agents/[agent-name]/evaluation.md`

---

## Iteration

Most agents require 1-2 iterations before approval. This is normal and expected. The typical flow is:

```
Planner -> Builder -> Reviewer -> [APPROVE WITH NOTES] -> Builder (fixes) -> Reviewer -> [APPROVE]
```

If you find yourself in more than 3 iterations, the architecture spec likely has a fundamental issue. Go back to the Planner.

## Quick Edit Shortcut

For small changes to existing, approved prompts:
1. Go directly to the Builder with the specific change request.
2. Then go to the Reviewer for a quick review (not full pipeline).

## Final Checklist

Before considering an agent complete, verify:

- [ ] Architecture spec saved to `agents/[name]/architecture-spec.md`
- [ ] System prompt saved to `agents/[name]/system-prompt.md`
- [ ] Test scenarios saved to `agents/[name]/test-scenarios.md`
- [ ] Evaluation saved to `agents/[name]/evaluation.md`
- [ ] Agent file copied to `.claude/agents/[name].md` (for Claude Code)
- [ ] Agent file copied to `.opencode/agents/[name].md` (for OpenCode)
- [ ] All files written in English
- [ ] Prompt version number assigned (v1.0 for first version)
