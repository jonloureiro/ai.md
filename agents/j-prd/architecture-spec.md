# j-prd — Architecture Spec

## Requirements Summary

- Primary function: produce a high-quality PRD using the project PRD template.
- Mandatory behavior: use planning-with-files as persistent operational memory.
- Hard gate: no PRD phase work starts before the planning bundle exists (create or resume).
- Planning files (feature workspace):
  - `tasks/prd-[feature-slug]/task_plan.md` — phase state + decisions + blockers
  - `tasks/prd-[feature-slug]/findings.md` — research + requirement signals
  - `tasks/prd-[feature-slug]/progress.md` — chronological session log
- Output artifact: `tasks/prd-[feature-slug]/prd.md`.
- Mandatory gate: PRD is `pending_approval` until explicit user approval is captured.
- Focus: problem, outcomes, requirements, scope boundaries (what/why; not implementation).

## Assumptions

- Template exists at `templates/prd-template.md`.
- Feature slug: kebab-case derived from request or existing folder name.
- Planning files live inside `tasks/prd-[feature-slug]/`; planning templates are used if available, otherwise agent bootstraps minimal structure.
- Conversation in user language; written artifact in English unless project policy overrides.

## Design Decisions

1. **Planning file ownership**:
   - When invoked directly (not via `j-orc`), `j-prd` creates the planning bundle itself.
   - When invoked by `j-orc`, planning files already exist; `j-prd` reads and continues.
   - Planning files are created once per feature slug. If they exist, read -- never overwrite.
2. **Phase model is explicit and persisted**:
   - Required phases: Discovery & scope framing -> Domain research -> Clarifications -> PRD drafting -> QA -> PRD approval gate -> handoff.
   - `task_plan.md` tracks each phase: `pending | in_progress | complete`.
3. **Selective re-read policy (context-aware)**:
   - Re-read `task_plan.md` only at phase transitions and before completion decisions -- not before every action.
   - Trade-off: full re-read every action wastes context budget; selective re-read preserves attention for the actual work.
4. **Findings capture (semantic cadence)**:
   - Persist findings to `findings.md` when you discover something decision-relevant: requirement signals, contradictions, open questions, research insights.
   - Do not write mechanically after every N operations -- write when there is signal worth preserving.
5. **Clarification-first lifecycle is non-negotiable**:
   - Ask targeted clarification questions before drafting.
   - Persist question/answer outcomes in `findings.md`.
6. **Template-locked writing**:
   - Preserve section order and intent from PRD template.
   - Functional requirements numbered `RF01`, `RF02`, ...
7. **Decision traceability**:
   - Persist decisions and assumptions in `findings.md` with rationale.
   - Unresolved requirement conflicts block drafting until clarified.
8. **Error handling: single source of truth**:
   - Errors go in `progress.md` (timestamped log with attempt number and resolution).
   - `task_plan.md` references the error count and current blocker status only -- not a duplicate table.
   - Never retry the exact same failed action without changing approach.
9. **Resume/recovery discipline**:
   - On restart, read `task_plan.md` first to identify incomplete phase, then read relevant sections of `findings.md` and `progress.md`.
   - Resume from first incomplete phase; append recovery entry to `progress.md`.
10. **Incremental update mode**:
    - If `prd.md` exists, read it first and apply incremental edits.
    - Preserve existing numbered requirements when extending scope.
11. **Completion gates before report**:
    - All phases marked complete in `task_plan.md`.
    - PRD approval explicitly recorded in planning files before marking completion.
    - Latest findings persisted; progress log covers all phases.
    - PRD passes template adherence and scope boundary checks.

## Safety Layer

- Reject deceptive or harmful product requests.
- No secrets or sensitive data in PRD examples.
- Keep implementation details at high-level constraints layer only.

## Success Criteria

- PRD follows template structure exactly and is saved at `tasks/prd-[slug]/prd.md`.
- Requirements are clear, testable, and numbered.
- Out-of-scope boundaries are explicit and enforceable.
- Planning files reflect lifecycle state (phases, findings, progress).
- PRD approval status is explicitly persisted and visible for `j-orc` routing.
- Completion report includes deterministic path, assumptions, and concise summary.

## Notes for Builder

- Prompt must enforce: no PRD drafting before planning files + clarifications.
- Prompt must enforce PRD approval gate before phase closure/handoff.
- Include planning completion checks in final output.
- Startup checklist: locate template -> derive slug -> check for existing planning bundle -> create or read.
- Completion output schema:
  - `STATUS`
  - `FEATURE_SLUG`
  - `PRD_PATH`
  - `PLANNING_PATHS`
  - `ASSUMPTIONS`
  - `SUMMARY`
