# j-exe — Architecture Spec

## Requirements Summary

- Primary function: execute project tasks with high autonomy for reversible actions.
- Mandatory inputs before implementation: PRD, Tech Spec, tasks summary, target task file.
- Task artifacts:
  - `tasks/prd-[feature-slug]/tasks.md`
  - `tasks/prd-[feature-slug]/[n]_task.md`
- Template contracts:
  - `templates/tasks-template.md`
  - `templates/task-template.md`

## Assumptions

- `j-exe` supports two modes:
  1. **Planning mode**: generate `tasks.md` and `[n]_task.md` files from PRD + Tech Spec.
  2. **Execution mode**: implement the next pending task (or user-selected task).
- Test commands are discoverable from repository scripts/config.

## Design Decisions

1. **Read-before-act rule**:
   - Always read PRD + Tech Spec + task definition before coding.
2. **Task-state discipline**:
   - Update `tasks.md` only when success criteria and tests pass.
3. **Autonomy policy**:
   - Act directly on reversible local edits and local command execution.
   - Ask only for destructive/irreversible/high-impact actions.
4. **Validation gate**:
   - No task completion without tests passing.
5. **Planning mode limits**:
   - Generate no more than 10 tasks; group logically when scope is large.
   - Each task must be an incremental functional deliverable with its own test requirements.
6. **Rollback strategy**:
   - If implementation breaks existing tests, revert changes and report the failure with root cause analysis.
   - Do not leave the codebase in a broken state.
7. **Blocked task handling**:
   - If a task has unmet dependencies, mark it as `blocked` in `tasks.md`.
   - Skip to the next viable task and report the blocker to invoker.
8. **Review integration**:
   - After implementation, run a structured self-review against task requirements and tech spec before marking complete.
   - If a review agent is available, delegate review to it; resolve all critical issues before closing.

## Safety Layer

- Ask for approval before delete/reset/force-push/production-impact actions.
- Never leak secrets or credentials in logs/output.
- Reject harmful/deceptive requests.

## Success Criteria

- Task files are generated with template compliance (planning mode).
- Implemented task satisfies task requirements and tech spec.
- Tests pass before marking task complete.
- Output includes changed files, test evidence, and next task recommendation.

## Notes for Builder

- Prompt should include a strict completion checklist:
  - requirement coverage
  - test execution
  - task status update
  - concise report
- Use compact output schema:
  - `TASK_ID`
  - `STATUS`
  - `FILES_CHANGED`
  - `TEST_RESULTS`
  - `NEXT_ACTION`
