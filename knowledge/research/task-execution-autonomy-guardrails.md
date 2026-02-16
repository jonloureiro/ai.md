# Task Execution Autonomy Guardrails

> Last updated: 2026-02-16
> Status: Current

## Summary

Autonomous task execution works best with strict preconditions (read PRD/Tech Spec/task), explicit completion gates (tests pass), and approval gates for irreversible actions. High autonomy should optimize reversible local work, not bypass safety.

## Sources

- Local templates:
  - `samples/iadevt5_4/templates/tasks-template.md`
  - `samples/iadevt5_4/templates/task-template.md`
- Local commands:
  - `samples/iadevt5_4/.claude/commands/criar-tasks.md`
  - `samples/iadevt5_4/.claude/commands/executar-task.md`
- Local task examples:
  - `samples/iadevt5_4/tasks/prd-weather-dashboard/tasks.md`
  - `samples/iadevt5_4/tasks/prd-weather-dashboard/1_task.md`
- Tool approval gates reference: <https://inference.sh/blog/tools/approval-gates>
- Multi-agent orchestration context: `knowledge/research/multi-agent-patterns.md`

## Key Findings

### 1) Read-before-act prevents invalid execution

The sample task template explicitly requires reading `prd.md` and `techspec.md` before execution.

### 2) Planning and execution should be separate modes

Tasks planning (`tasks.md` + `[n]_task.md`) and implementation execution are distinct activities and should not be conflated.

### 3) Test-passing is a hard completion gate

Task completion without successful test execution creates false progress and must be blocked.

### 4) Approval gates should be risk-based

Reversible local actions can be autonomous; irreversible/high-impact actions require explicit user approval.

### 5) Task state updates must be evidence-backed

Marking tasks complete should only happen after criteria and validations are satisfied.

## Implications for This Workspace

- `j-exe` should support planning mode and execution mode.
- `j-exe` should enforce tests-before-completion and deterministic task status updates.
- `j-exe` should use high autonomy only for reversible operations.
- Orchestrator handoffs should include acceptance criteria to reduce execution ambiguity.
