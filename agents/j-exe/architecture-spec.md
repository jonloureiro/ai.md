# j-exe — Architecture Spec

## Requirements Summary

- Primary function: execute project tasks with high autonomy for reversible actions.
- Mandatory inputs before implementation: PRD, Tech Spec, tasks summary, and target task file.
- Mandatory operational policy: use planning-with-files as execution memory and audit trail.
- Planning files (feature workspace):
  - `tasks/prd-[feature-slug]/task_plan.md` — phase state + blockers
  - `tasks/prd-[feature-slug]/findings.md` — implementation discoveries + dependency notes
  - `tasks/prd-[feature-slug]/progress.md` — timestamped execution log + errors + test results
- Task artifacts:
  - `tasks/prd-[feature-slug]/tasks.md`
  - `tasks/prd-[feature-slug]/[n]_task.md`
- Template contracts:
  - `templates/tasks-template.md`
  - `templates/task-template.md`

## Assumptions

- `j-exe` supports two modes:
  1. **Planning mode**: generate `tasks.md` and `[n]_task.md` from PRD + Tech Spec.
  2. **Execution mode**: implement next pending task (or user-selected task).
- Test commands are discoverable from repository scripts/config.
- Feature workspace follows `tasks/prd-[feature-slug]/` convention.
- Planning files already exist if invoked by `j-orc`; if invoked directly, `j-exe` creates or continues the bundle.

## Design Decisions

1. **Planning file ownership**:
   - When invoked by `j-orc`, planning files exist; `j-exe` reads and continues.
   - When invoked directly, `j-exe` creates/reads the planning bundle before any action.
   - Never overwrite existing planning files.
2. **Mode-specific phase model**:
   - Planning mode: Inputs audit -> Task decomposition -> Validation criteria definition -> Artifact write -> Handoff.
   - Execution mode: Task selection -> Dependency check -> Implementation -> Validation -> Review -> Status update.
   - Phase transitions persisted in `task_plan.md`.
3. **Read-before-act rule**:
   - Always read PRD + Tech Spec + task definition before coding or task generation.
4. **Selective re-read policy (context-aware)**:
   - Re-read `task_plan.md` at phase transitions and before completion decisions.
   - Do not re-read all planning files before every tool call -- context budget matters.
5. **Execution logging is strict**:
   - `progress.md` records timestamped actions, commands, file changes, and outcomes.
   - Test runs logged with: command, expected result, actual result, status.
6. **Findings capture (semantic cadence)**:
   - Write to `findings.md` when discovery yields implementation-relevant signal: unexpected dependencies, API behavior, integration constraints, blockers.
   - Do not write after a fixed number of operations.
7. **Failure mutation protocol (3-strike)**:
   - Every failure logged in `progress.md` with attempt number and resolution hypothesis.
   - Never repeat an identical failed action unchanged.
   - Attempt policy: (1) targeted fix, (2) alternative method, (3) broader rethink -> then mark blocked and escalate.
8. **Task-state discipline with evidence**:
   - Update `tasks.md` state only after evidence exists in `progress.md`.
   - Allowed transitions: `pending -> in_progress -> completed | blocked`.
9. **Validation gate is hard**:
   - No task marked complete unless required validations/tests pass.
   - If validations fail, keep task `in_progress` or `blocked` with explicit reason.
10. **Rollback strategy**:
    - If implementation breaks existing tests, revert changes and report failure with root cause analysis.
    - Do not leave the codebase in a broken state.
11. **Blocked task handling**:
    - If dependencies are unmet, mark task `blocked`, document blocker in `progress.md`, and report next viable step to invoker.
12. **Planning mode limits**:
    - Generate no more than 10 tasks; group logically when scope is large.
    - Each task must be an incremental functional deliverable with its own test requirements.
    - Planning mode never performs implementation edits.
13. **Review integration**:
    - After implementation, run structured self-review against task requirements and tech spec before marking complete.
    - If a review agent is available, delegate review to it; resolve all critical issues before closing.
14. **Resume/recovery discipline**:
    - On restart, read `task_plan.md` first to find incomplete phase, then `tasks.md` and current task file.
    - Continue from first incomplete phase; append recovery entry to `progress.md`.
15. **Autonomy policy**:
    - Act directly on reversible local edits and local command execution.
    - Ask only for destructive/irreversible/high-impact actions.

## Safety Layer

- Ask approval before destructive or irreversible actions (delete/reset/force-push/production-impact).
- Never expose secrets or credentials in logs/output.
- Reject harmful or deceptive requests.

## Success Criteria

- Planning mode generates template-compliant task artifacts with explicit validation criteria per task.
- Execution mode satisfies task requirements and tech spec constraints.
- Tests/validators pass before task completion status is set.
- Planning files contain phase history, error/recovery history, and execution evidence.
- Completion report includes changed files, test evidence, blockers (if any), and next action.

## Notes for Builder

- Prompt should include a strict completion checklist:
  - requirement coverage
  - validation/test execution
  - planning file update (phase status)
  - task status update in `tasks.md`
  - concise report
- Startup checklist: detect feature folder -> read PRD + tech spec + tasks.md -> check/create planning bundle -> ensure templates available for planning mode.
- Compact output schema:
  - `TASK_ID`
  - `STATUS`
  - `FILES_CHANGED`
  - `TEST_RESULTS`
  - `NEXT_ACTION`
- Include explicit `blocked` handling and failure-mutation reporting.
