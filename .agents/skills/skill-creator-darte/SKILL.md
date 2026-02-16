---
name: skill-creator-darte
description: Orchestrate the D.A.R.T.E. pipeline to create a new skill from concept to approved SKILL.md. Integrates with the external skill-creator skill for detailed implementation guidance. Use when creating new skills that require structured planning and review.
---

# Skill Creator (D.A.R.T.E. Pipeline)

Run the full D.A.R.T.E. pipeline for skill creation.

Use this skill for orchestration. For detailed authoring guidance, also invoke external `/skill-creator`.

## Use When

- User needs a new skill from scratch
- You need structured planning -> writing -> review

## Step 0: Reference SkillsMP First

Before designing, review similar public skills:

- `/skillsmp-search recent`
- `/skillsmp-search popular` (or `stars`)
- https://skillsmp.com for keyword/semantic search

Use findings as reference patterns only. Do not copy/extend external skills directly.

## Pipeline

### 1) Plan (`/prompt-architect-planner`)

- Input: skill requirements
- Output: skill architecture spec
- Save: `skills/[name]/skill-architecture-spec.md`

### 2) Build (`/prompt-architect-builder`)

- Input: approved architecture spec
- Output: `SKILL.md` (+ optional references/scripts/assets)
- Save: `skills/[name]/SKILL.md`

### 3) Review (`/prompt-architect-reviewer`)

- Input: architecture spec + `SKILL.md`
- Output: scenarios, evaluation, decision, fix list
- Save:
  - `skills/[name]/test-scenarios.md`
  - `skills/[name]/evaluation.md`

## Decision Loop

- **APPROVE**: done.
- **APPROVE WITH NOTES**: Builder updates, then review again.
- **REJECT**: return to Planner (architecture) or Builder (writing).

Typical loop:

```text
Planner -> Builder -> Reviewer -> Builder (if needed) -> Reviewer
```

## Deployment Notes

- Canonical source: `.agents/skills/[name]/SKILL.md`
- Keep platform-specific variants only when required by frontmatter differences.
- If targeting OpenCode and Claude Code, handle unsupported Claude-only fields via graceful degradation.

## Quick Edit Path

For small changes to existing skills:
1. Builder for targeted edit
2. Reviewer for quick validation

## Completion Checklist

- [ ] SkillsMP references reviewed
- [ ] `skills/[name]/skill-architecture-spec.md`
- [ ] `skills/[name]/SKILL.md`
- [ ] `skills/[name]/test-scenarios.md`
- [ ] `skills/[name]/evaluation.md`
- [ ] Deploy to `.agents/skills/[name]/SKILL.md`
- [ ] Name validated (`^[a-z0-9]+(-[a-z0-9]+)*$`, max 64)
- [ ] Body concise (target under 500 lines / 5000 tokens)
- [ ] Files in English
