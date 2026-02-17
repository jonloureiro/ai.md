# j-researcher Skill Research

> Last updated: 2026-02-17
> Status: Current

## Summary

This research defines the operational contract for a new `j-researcher` skill used by `j-*` agents. The skill should be Factory-first but cross-platform compatible, enforce evidence-only reasoning, require high-volume parallel research, verify current date before research, and persist findings into `./knowledge`.

## Sources

- Local architecture specs:
  - `agents/j-orc/architecture-spec.md`
  - `agents/j-prd/architecture-spec.md`
  - `agents/j-tec/architecture-spec.md`
  - `agents/j-exe/architecture-spec.md`
- External local reference:
  - `/Users/jon/Workspaces/jonloureiro/tidy-archer/knowledge/research/j-agents.md`
- Local knowledge base:
  - `knowledge/research/context-engineering.md`
  - `knowledge/research/multi-agent-patterns.md`
  - `knowledge/research/platform-agent-mechanics.md`
  - `knowledge/research/claude-4-best-practices.md`
  - `knowledge/research/task-execution-autonomy-guardrails.md`
  - `knowledge/research/skill-creation-architecture.md`
- Factory docs:
  - `https://docs.factory.ai/factory-docs-map.md`
  - `https://docs.factory.ai/cli/configuration/skills`
  - `https://docs.factory.ai/cli/configuration/custom-droids`
  - `https://docs.factory.ai/cli/getting-started/how-to-talk-to-a-droid`

## Key Findings

### 1) `j-*` pipeline is orchestration-first with persistent evidence files

- `j-orc` owns lifecycle routing and state persistence.
- Required bundle for each feature slug: `task_plan.md`, `findings.md`, `progress.md`.
- Approval gates are strict: PRD approval before Tech Spec approval; Tech Spec approval before tasks/execution.

### 2) Specialists are evidence-driven and phase-gated

- `j-prd`: clarification-first, template-locked, approval-gated.
- `j-tec`: evidence-first, RF traceability, approval-gated.
- `j-exe`: read-before-act, validation/test gate, blocked-state discipline.
- Common pattern: no completion without explicit gate satisfaction and persisted evidence.

### 3) External `j-agents.md` confirms stable contracts and artifacts

- Confirms same `j-orc -> j-prd -> j-tec -> j-exe` pipeline.
- Confirms planning-with-files structure and compact output contracts per agent.
- Reinforces that high-quality operation depends on artifact traceability, not implicit memory.

### 4) Factory skills favor focused frontmatter and reusable operational playbooks

- A skill is a directory with `SKILL.md` and optional support files.
- `name` and `description` are key trigger metadata.
- Skills should be narrow, verifiable, and composable.
- Verification steps should be explicit in the skill instructions.

### 5) Factory custom droids provide isolated contexts and `Task`-based delegation

- Custom droids are invokable as `subagent_type` targets via Task.
- Subagent execution uses fresh context windows (context isolation), which supports parallel research swarms.
- This aligns with a deep-research skill that mandates parallel operations and explicit evidence logging.

### 6) Research quality must optimize signal and avoid invented knowledge

- Context engineering guidance: prioritize high-signal tokens and explicit evidence.
- Prompt best practices: avoid over-aggressive instruction style; keep mandatory rules clear and minimal.
- Multi-agent guidance: orchestrator/worker patterns perform better when handoffs are structured and evidence-backed.

### 7) Required behavior for `j-researcher`

The skill should enforce the following hard gates:

1. **Date Gate**: verify and record current date/time before any research work.
2. **Knowledge Gate**: read `knowledge/INDEX.md` and relevant local research before external exploration.
3. **Evidence Gate**: no claim can rely on unstated or assumed knowledge.
4. **Parallelism Gate**: run at least two parallel waves with high volume (Wave 1 >= 8 ops, Wave 2 >= 6 ops).
5. **Dual-Source j-agent Gate**: investigate `agents/j-*` and external `j-agents.md` in parallel when task touches j-agents.
6. **Persistence Gate**: persist findings to `knowledge/research/*.md` and update `knowledge/INDEX.md`.
7. **Unknown Handling Gate**: unresolved claims must be marked `UNKNOWN` with next research action.

## Recommended Skill Output Contract

`j-researcher` should require this output shape from the executing agent:

```md
STATUS: <active|blocked|completed>
DATE_VERIFIED: <timestamp>
SOURCES_USED:
- <path-or-url>
EVIDENCE_SUMMARY:
- <claim -> source -> implication>
KNOWLEDGE_UPDATES:
- <knowledge/research/file.md>
- <knowledge/INDEX.md updated: yes|no>
UNKNOWNS:
- <item or none>
NEXT_ACTION:
- <single next step>
```

## Implications for This Workspace

- `j-researcher` should be added as canonical skill in `.agents/skills/j-researcher/SKILL.md`.
- The skill should remain platform-compatible by using standard frontmatter fields only.
- The skill should explicitly encode date-first, knowledge-first, parallel-first, and persistence-first behavior.
