---
name: skill-creator-darte
description: Orchestrate the D.A.R.T.E. pipeline to create a new skill from concept to approved SKILL.md. Integrates with the external skill-creator skill for detailed implementation guidance. Use when creating new skills that require structured planning and review.
---

# Skill Creator (D.A.R.T.E. Pipeline)

Orchestrate the full D.A.R.T.E. pipeline for creating a new skill. Follow the steps in order.

This skill provides the pipeline structure. For detailed guidance on skill anatomy, progressive disclosure, and bundling resources, also invoke the external **`/skill-creator`** skill.

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

## Prerequisites

Before starting, ensure you can answer:
- What is the skill's purpose?
- Which platform(s) will it target? (Claude Code, OpenCode, or both)
- Is it user-invocable (`/skill-name`) or agent-invocable (auto-loaded)?
- Does it need arguments?

## Step 1: Plan

Invoke `/prompt-architect-planner` with your initial requirements.

**Input**: Your description of what the skill should do. The Planner will ask for what's missing.

**Output**: Complete skill architecture spec including:
- Requirements Summary (purpose, platforms, invocation mode, scope)
- Naming (validated kebab-case)
- Frontmatter Design
- Content Strategy (body vs. references/ vs. scripts/ vs. assets/)
- Progressive Disclosure Plan
- Argument Design (if applicable)
- Cross-Platform Compatibility Plan

**Save to**: `skills/[skill-name]/skill-architecture-spec.md`

Proceed when the spec is delivered and you approve it. Do not skip to Step 2 incomplete.

---

## Step 2: Build

Invoke `/prompt-architect-builder` with the architecture spec from Step 1.

**Input**: The complete skill architecture spec.

**Output**: Complete SKILL.md with:
- YAML frontmatter (name, description, optional Claude Code extensions)
- Body content (task-focused instructions)
- Supporting files if needed (references/, scripts/, assets/)

**Save to**: `skills/[skill-name]/SKILL.md`

**Deployment**:
- Copy to `.claude/skills/[skill-name]/` (for Claude Code)
- Copy to `.opencode/skills/[skill-name]/` (for OpenCode)

**Tip**: For detailed implementation guidance (anatomy, progressive disclosure patterns, packaging), invoke the external `/skill-creator` skill.

---

## Step 3: Review

Invoke `/prompt-architect-reviewer` with the SKILL.md (Step 2) and architecture spec (Step 1).

**Input**: Complete SKILL.md AND architecture spec.

**Output**:
- 4 skill-specific test scenarios (standard invocation, argument edge cases, cross-platform compatibility, context budget)
- Evaluation scores across 7 dimensions (description quality, naming, conciseness, progressive disclosure, argument handling, cross-platform, invocation design)
- Decision (APPROVE, APPROVE WITH NOTES, or REJECT)
- Improvement recommendations

**Save to**:
- `skills/[skill-name]/test-scenarios.md`
- `skills/[skill-name]/evaluation.md`

**Outcomes**:
- **APPROVE**: Save. Skill is ready.
- **APPROVE WITH NOTES**: Save, then return to Builder (Step 2) with requested improvements.
- **REJECT**: If architectural issues, return to Planner (Step 1). If writing issues, return to Builder (Step 2).

---

## Iteration

Most skills require 1-2 iterations before approval. Typical flow:

```
Planner -> Builder -> Reviewer -> [APPROVE WITH NOTES] -> Builder -> Reviewer -> [APPROVE]
```

More than 3 iterations indicates a fundamental architecture issue. Return to Planner.

---

## Quick Edit Shortcut

For small changes to existing approved skills:
1. Go directly to Builder with the specific change
2. Then Reviewer for quick review (not full pipeline)

---

## Skill File Structure

When complete, a skill has the following structure:

```
skills/[skill-name]/
├── skill-architecture-spec.md    # From Planner
├── SKILL.md                      # From Builder (Primary Source)
├── test-scenarios.md             # From Reviewer
├── evaluation.md                 # From Reviewer
├── references/                   # Optional — additional documentation
├── scripts/                      # Optional — executable code
└── assets/                       # Optional — templates, images, data files

.claude/skills/[skill-name]/      # DEPLOYMENT ONLY
├── SKILL.md
└── ...

.opencode/skills/[skill-name]/    # DEPLOYMENT ONLY
├── SKILL.md
└── ...
```

---

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
3. The OpenCode version omits unsupported fields

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

---

## Final Checklist

Before considering a skill complete, verify:

- [ ] Skill architecture spec saved to `skills/[name]/skill-architecture-spec.md`
- [ ] SKILL.md saved to `skills/[name]/SKILL.md`
- [ ] Test scenarios saved to `skills/[name]/test-scenarios.md`
- [ ] Evaluation saved to `skills/[name]/evaluation.md`
- [ ] SKILL.md copied to `.claude/skills/[name]/SKILL.md` (Deployment)
- [ ] SKILL.md copied to `.opencode/skills/[name]/SKILL.md` (Deployment)
- [ ] All files written in English
- [ ] Skill version assigned (v1.0 for first version)
- [ ] Name validated: `^[a-z0-9]+(-[a-z0-9]+)*$`, max 64 chars
- [ ] Description is clear, specific, and includes keywords for agent discovery
- [ ] SKILL.md body is under 500 lines / 5000 tokens (or justified if over)
- [ ] Progressive disclosure is used (essential info in body, details in references/)
- [ ] Cross-platform compatibility is handled (if targeting both platforms)
