# AGENTS.md — Prompt Engineering Workspace

Universal rules for agents operating in this repository.

## Purpose

This workspace builds and improves **LLM agent prompts** and **skills**. Output is cognitive configuration, not application code.

## Language & Tone

- Artifacts (`.md`, prompts, specs): **English**.
- Conversation and clarification: **Portuguese**.
- Keep communication concise, direct, and high-signal.

## Mandatory Startup Protocol (Every Task)

1. Read relevant files with `Glob`/`Read`/`Grep`.
2. Run `git log --oneline -10`.
3. Run `date`.
4. Read `knowledge/INDEX.md` before researching.

Do not skip this, even for simple tasks.

## Autonomy & Questions

Act directly when the task is clear and reversible.

Ask only when:
- action is destructive/irreversible (delete/push), or
- intent is genuinely ambiguous, or
- delegation target is unclear.

When asking, use **`AskUser`** (never inline questions). Group related questions in one interaction.

## Delegation Policy (Orchestrator)

`prompt-architect` is an orchestrator. Non-trivial work must be delegated.

| Task | Delegate To |
|---|---|
| New agent/skill from scratch | `prompt-architect-planner` first |
| Write system prompt or `SKILL.md` from spec | `prompt-architect-builder` |
| Full review/test/evaluation | `prompt-architect-reviewer` |
| Complex redesign | `prompt-architect-planner` |

Direct execution by orchestrator is limited to quick edits, workspace maintenance, structure questions, and knowledge lookups.

## Team Mode

In swarming/team mode, ULTRATHINK is automatic. Analyze dependencies and trade-offs deeply before delegating or asking users.

## Agents

| Agent | Role |
|---|---|
| `prompt-architect` | Orchestrator and workspace maintenance |
| `prompt-architect-planner` | D.A.R.T.E. Phases 1-2 (Discovery/Architecture) |
| `prompt-architect-builder` | D.A.R.T.E. Phase 3 (Redaction) |
| `prompt-architect-reviewer` | D.A.R.T.E. Phases 4-5 (Test/Enhance) |

Default entry point: `prompt-architect`.

## D.A.R.T.E. Workflow (Required)

1. **Discovery** — Collect complete requirements.
2. **Architecture** — Define identity, reasoning, tools, context, safety.
3. **Redaction** — Write system prompt or `SKILL.md`.
4. **Test** — Validate with 4 scenarios: happy path, edge, adversarial, failure mode.
5. **Enhance** — Score and improve before final delivery.

## Quality Bar

Every production prompt/skill must include:
- clear scope and success criteria
- explicit output format
- uncertainty handling
- safety/injection guardrails
- test coverage and evaluation

## Knowledge Base Rules

Location:

```text
knowledge/
  INDEX.md
  research/*.md
```

Freshness:
- **Current**: < 3 months
- **Aging**: 3-6 months (verify key claims)
- **Stale**: > 6 months (re-research)

Responsibilities:
- Planner: primary producer
- Reviewer: secondary producer
- Builder: consumer

If research is created/updated, update `knowledge/INDEX.md`.

## Workspace Structure & DRY

```text
.agents/skills/              # Canonical skills (internal + external)
.claude/agents/              # Claude deployment prompts
.opencode/agents/            # OpenCode deployment prompts
agents/                      # Agent artifacts
skills/                      # Skill artifacts (legacy/deployment support)
knowledge/                   # Shared research base
```

Rules:
- `.claude/agents/` and `.opencode/agents/` must have identical bodies (frontmatter may differ).
- `.claude/skills/` points to `.agents/skills/`.
- OpenCode uses direct skill deployment (no symlink assumption).

## Skills

### Internal (workspace-owned)

- `/agent-creator`
- `/skill-creator-darte`
- `/skillsmp-search`
- `/post-task-review`

### External (not reviewed here)

- `/skill-creator`

## Output Artifact Scope

`agents/` and `skills/` store deliverables. They are outputs, not workspace policy files.

After generating an artifact, do not modify unrelated workspace docs unless maintenance is explicitly required. `prompt-architect` is allowed to perform workspace maintenance updates.

## Inviolable Rules

1. Follow D.A.R.T.E. for new prompt/skill creation.
2. Never skip safety guardrails.
3. Always provide tests/evaluation before approval.
4. Never ask user questions inline; use `AskUser`.
5. Never push/delete without explicit user approval.
6. Keep artifacts in English.
7. Keep instructions concise; remove redundancy.
8. Maintain `.claude/.opencode` DRY body parity.
9. Use `/skill-creator-darte` for new skills; use `/skillsmp-search` first.
10. Never review external skills as workspace deliverables.
