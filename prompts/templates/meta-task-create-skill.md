# Meta-Task: Create a New Skill

> Use this template to orchestrate the full D.A.R.T.E. pipeline for creating a new skill. Follow the steps in order. Each step maps to one of the three PromptArchitect agents.

## Prerequisites

Before starting, you need:
- A clear idea of what the skill should do (even if rough)
- Knowledge of which platform(s) will host the skill (Claude Code, OpenCode, or both)
- Understanding of whether the skill should be user-invocable (`/skill-name`), agent-invocable (auto-loaded), or both

## Quick Reference: Agent vs. Skill

Not sure if you need an agent or a skill? Use this guide:

| You need a SKILL if: | You need an AGENT if: |
|---|---|
| Extending an existing agent's capabilities | Defining a persistent identity |
| Task-specific instructions | Cognitive bias and reasoning strategy |
| Should be under 500 lines / 5000 tokens | Can be larger, full system prompt |
| Loaded on demand mid-conversation | Always loaded at session start |
| No identity or persona needed | Identity is core to behavior |
| Multiple can be active simultaneously | One agent active at a time |

## Quick Checklist Reference

Before starting, ensure you can answer:
- [ ] What is the skill's purpose?
- [ ] Which platform(s) will it target?
- [ ] Is it user-invocable or agent-invocable?
- [ ] Does it need arguments?

(See full checklist at the end of this document)

## Pipeline

### Step 1: Plan (prompt-architect-planner)

Invoke the `prompt-architect-planner` agent (run `/prompt-architect-planner` or use agent selector) and provide your initial requirements.

**Input**: Your description of what the skill should do. Include as much or as little as you know — the Planner will ask for what is missing.

**Expected Output**: A complete skill architecture spec in the standard format:
- Requirements Summary (purpose, platforms, invocation mode, scope)
- Naming (validated kebab-case)
- Frontmatter Design (standard + Claude Code extensions if applicable)
- Content Strategy (body vs. references/ vs. scripts/ vs. assets/)
- Progressive Disclosure Plan
- Argument Design (if applicable)
- Cross-Platform Compatibility Plan

**When to proceed**: When the Planner delivers the skill architecture spec AND you have reviewed and approved it. Do not skip to Step 2 with an incomplete spec.

**Save the output to**: `prompts/skills/[skill-name]/skill-architecture-spec.md`

---

### Step 2: Build (prompt-architect-builder)

Invoke the `prompt-architect-builder` agent and provide the skill architecture spec from Step 1.

**Input**: The complete skill architecture spec. Copy-paste or reference the file.

**Expected Output**: A complete SKILL.md file with:
- YAML frontmatter (standard fields + applicable Claude Code extensions)
- Body content (task-focused instructions, not identity)
- Supporting files if needed (references/, scripts/, assets/)
- Platform variants if targeting both Claude Code and OpenCode

**When to proceed**: When the Builder delivers the complete SKILL.md and you have done an initial read-through. You do not need to approve it — that is the Reviewer's job.

**Save the output to**:
- `.claude/skills/[skill-name]/SKILL.md` (Claude Code version)
- `.opencode/skills/[skill-name]/SKILL.md` (OpenCode version, if different)
- Any supporting files to their respective directories

---

### Step 3: Review (prompt-architect-reviewer)

Invoke the `prompt-architect-reviewer` agent and provide the SKILL.md from Step 2, along with the skill architecture spec from Step 1 for reference.

**Input**: The complete SKILL.md AND the skill architecture spec.

**Expected Output**:
- 4 skill-specific test scenarios (standard invocation, argument edge cases, cross-platform compatibility, context budget and size)
- Evaluation scores across 7 dimensions (description quality, naming, conciseness, progressive disclosure, argument handling, cross-platform, invocation design)
- Decision (APPROVE, APPROVE WITH NOTES, or REJECT)
- Improvement recommendations if applicable

**Possible outcomes**:

- **APPROVE**: Save the test scenarios and evaluation. The skill is ready.
- **APPROVE WITH NOTES**: Save everything. Apply the suggested improvements by going back to the Builder (Step 2) with the specific changes requested.
- **REJECT**: Review the rejection reasons. If the issues are architectural, go back to the Planner (Step 1). If the issues are in the writing, go back to the Builder (Step 2).

**Save the output to**:
- `prompts/skills/[skill-name]/test-scenarios.md`
- `prompts/skills/[skill-name]/evaluation.md`

---

## Iteration

Most skills require 1-2 iterations before approval. This is normal and expected. The typical flow is:

```
Planner -> Builder -> Reviewer -> [APPROVE WITH NOTES] -> Builder (fixes) -> Reviewer -> [APPROVE]
```

If you find yourself in more than 3 iterations, the architecture spec likely has a fundamental issue. Go back to the Planner.

## Quick Edit Shortcut

For small changes to existing, approved skills:
1. Go directly to the Builder with the specific change request.
2. Then go to the Reviewer for a quick review (not full pipeline).

## Skill File Structure

When complete, a skill has the following structure:

```
.claude/skills/[skill-name]/
├── SKILL.md           # Required — main instructions
├── references/        # Optional — additional documentation
├── scripts/           # Optional — executable code
└── assets/            # Optional — templates, images, data files

.opencode/skills/[skill-name]/
├── SKILL.md           # Required — main instructions (may differ in frontmatter)
├── references/        # Optional — additional documentation
├── scripts/           # Optional — executable code
└── assets/            # Optional — templates, images, data files

prompts/skills/[skill-name]/
├── skill-architecture-spec.md    # From Planner
├── test-scenarios.md             # From Reviewer
└── evaluation.md                 # From Reviewer
```

## Final Checklist

Before considering a skill complete, verify:

- [ ] Skill architecture spec saved to `prompts/skills/[name]/skill-architecture-spec.md`
- [ ] SKILL.md saved to `.claude/skills/[name]/SKILL.md`
- [ ] SKILL.md saved to `.opencode/skills/[name]/SKILL.md` (if targeting OpenCode)
- [ ] Test scenarios saved to `prompts/skills/[name]/test-scenarios.md`
- [ ] Evaluation saved to `prompts/skills/[name]/evaluation.md`
- [ ] All files written in English
- [ ] Skill version number assigned (v1.0 for first version; increment minor for updates, major for breaking changes)
- [ ] Name validated: `^[a-z0-9]+(-[a-z0-9]+)*$`, max 64 chars
- [ ] Description is clear, specific, and includes keywords for agent discovery
- [ ] SKILL.md body is under 500 lines / 5000 tokens (or justified if over)
- [ ] Progressive disclosure is used (essential info in body, details in references/)
- [ ] Cross-platform compatibility is handled (if targeting both platforms)

## Cross-Platform Notes

### Claude Code-Only Features

These frontmatter fields work in Claude Code but NOT OpenCode:
- `argument-hint`: Hint shown during autocomplete
- `disable-model-invocation`: Prevents auto-loading
- `user-invocable`: Hides from `/` menu when false
- `allowed-tools`: Pre-approves specific tools
- `model`: Overrides model for this skill
- `context`: Runs in forked subagent
- `agent`: Which subagent for forked context
- `hooks`: Lifecycle hooks

If your skill uses these features and targets OpenCode:
1. Note the degradation in the skill architecture spec
2. The Builder will produce two SKILL.md variants
3. The OpenCode version will omit the unsupported fields
4. The skill should still function correctly on OpenCode (possibly with manual invocation instead of auto-loading)

### Platform Search Paths

**Claude Code searches** (in priority order):
1. Enterprise-managed skills
2. `~/.claude/skills/` (personal)
3. `.claude/skills/` (project)
4. Plugin skills

**OpenCode searches** (walking up from cwd to git root):
1. `.opencode/skills/`
2. `~/.config/opencode/skills/`
3. `.claude/skills/` (cross-compatible)
4. `~/.claude/skills/` (cross-compatible)
5. `.agents/skills/` (generic)

## Related Templates

- `meta-task-create-agent.md`: For creating full agents (persistent identity, reasoning strategy)
