# Implementation Task Summary for `[FEATURE_NAME]`

## Inputs
- PRD: `tasks/prd-[FEATURE_SLUG]/prd.md`
- Tech Spec: `tasks/prd-[FEATURE_SLUG]/techspec.md`
- Generated At: `[YYYY-MM-DD HH:MM TZ]`

## Task Status Rules
- Allowed Status Values: `pending | in_progress | completed | blocked`
- Transition Rule: `pending -> in_progress -> completed|blocked`
- A task can be marked `completed` only with validation evidence.

## High-Level Tasks (Max 10)
| Task ID | Title | Status | Deliverable | Dependencies | Parallelizable | Validation Requirements | Evidence Reference |
|---|---|---|---|---|---|---|---|
| `1.0` | `[TASK_TITLE]` | `[pending|in_progress|completed|blocked]` | `[INCREMENTAL_FUNCTIONAL_DELIVERABLE]` | `[NONE|TASK_IDS]` | `[YES|NO]` | `[UNIT|INTEGRATION|E2E|LINT|TYPECHECK]` | `[progress.md#REF]` |
| `2.0` | `[TASK_TITLE]` | `[pending|in_progress|completed|blocked]` | `[INCREMENTAL_FUNCTIONAL_DELIVERABLE]` | `[NONE|TASK_IDS]` | `[YES|NO]` | `[UNIT|INTEGRATION|E2E|LINT|TYPECHECK]` | `[progress.md#REF]` |

## Sequencing Notes
- `[WHY_THIS_ORDER]`
- `[WHAT_CAN_RUN_IN_PARALLEL]`

## Task-to-Requirement Coverage
| Task ID | Covered RFs | Coverage Notes |
|---|---|---|
| `1.0` | `[RF01, RF02]` | `[NOTES]` |

## Completion Checklist
- [ ] Every task is an incremental, functional deliverable.
- [ ] Every task defines required validations/tests.
- [ ] Dependencies are explicit and enforceable.
- [ ] No task exceeds agreed scope.
