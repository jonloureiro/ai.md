# Task Plan

## Context
- Feature Slug: `[FEATURE_SLUG]`
- Workspace Path: `tasks/prd-[FEATURE_SLUG]/`
- Goal: `[GOAL_STATEMENT]`
- Owner: `[PRIMARY_AGENT_OR_OWNER]`

## Current State
- Current Phase: `[PHASE: discovery|prd|techspec|tasks|execution|closure]`
- Overall Status: `[STATUS: active|blocked|completed]`
- Last Updated: `[TIMESTAMP: YYYY-MM-DD HH:MM TZ]`
- Next Action: `[NEXT_ACTION]`

## Phase Tracker
| Phase | Status | Entry Criteria | Exit Criteria | Main Artifacts |
|---|---|---|---|---|
| discovery | `[pending|in_progress|complete|blocked]` | Request and context captured | Scope, constraints, and assumptions captured | `findings.md`, `progress.md` |
| prd | `[pending|in_progress|complete|blocked]` | Discovery complete | `prd.md` drafted and reviewed | `prd.md`, `findings.md` |
| techspec | `[pending|in_progress|complete|blocked]` | `prd.md` available | `techspec.md` drafted with RF traceability | `techspec.md`, `findings.md` |
| tasks | `[pending|in_progress|complete|blocked]` | `prd.md` + `techspec.md` available | `tasks.md` and `[n]_task.md` files created | `tasks.md`, `[n]_task.md` |
| execution | `[pending|in_progress|complete|blocked]` | Task files available | Selected tasks implemented with passing validations | `tasks.md`, `progress.md` |
| closure | `[pending|in_progress|complete|blocked]` | All planned work resolved | Final status and handoff completed | `task_plan.md`, `progress.md` |

## Gate Checklist
| Gate | Status | Evidence |
|---|---|---|
| Planning bundle initialized (`task_plan.md`, `findings.md`, `progress.md`) | `[pass|fail]` | `[PROGRESS_REF]` |
| PRD exists before Tech Spec phase | `[pass|fail|n/a]` | `[FILE_PATH_OR_PROGRESS_REF]` |
| PRD + Tech Spec exist before execution phase | `[pass|fail|n/a]` | `[FILE_PATHS_OR_PROGRESS_REF]` |
| Validation/tests pass before task completion | `[pass|fail|n/a]` | `[PROGRESS_REF_OR_TEST_COMMAND]` |
| Closure includes updated status and logs | `[pass|fail|n/a]` | `[PROGRESS_REF]` |

## Decisions
| Decision | Rationale | Phase | Reference |
|---|---|---|---|
| `[DECISION]` | `[RATIONALE]` | `[PHASE]` | `[findings.md#SECTION_OR_PROGRESS_REF]` |

## Blocker Status
- Current Blocker: `[none|blocked]`
- Blocker Description: `[BLOCKER_DESCRIPTION_OR_NONE]`
- Unblock Condition: `[UNBLOCK_CONDITION_OR_NA]`
- Progress Reference: `[PROGRESS_REF: progress.md#SESSION_OR_ENTRY]`

## Delegate Handoff Context
### Constraints
- `[CONSTRAINT]`

### Acceptance Criteria
- `[ACCEPTANCE_CRITERION]`

### Context Summary
- `[CONTEXT_SUMMARY]`

### Artifact Paths
- `[ARTIFACT_PATH]`
