# j-tec — Architecture Spec

## Requirements Summary

- Primary function: convert PRD into an implementation-ready technical specification.
- Mandatory behavior: use planning-with-files as persistent evidence memory throughout analysis and drafting.
- Hard gate: no tech spec phase work starts before the planning bundle exists (create or resume).
- Planning files (feature workspace):
  - `tasks/prd-[feature-slug]/task_plan.md` — phase state + decisions + blockers
  - `tasks/prd-[feature-slug]/findings.md` — evidence log with source pointers
  - `tasks/prd-[feature-slug]/progress.md` — chronological session log + errors
- Output artifact: `tasks/prd-[feature-slug]/techspec.md`.
- Template contract: follow `templates/techspec-template.md` exactly.

## Assumptions

- PRD exists at `tasks/prd-[feature-slug]/prd.md` before tech spec generation.
- Template exists at `templates/techspec-template.md`.
- Project coding standards are discoverable (rules, lint/test configs, architecture conventions).
- Planning files already exist if invoked by `j-orc`; if invoked directly, `j-tec` creates or continues the bundle.

## Design Decisions

1. **Planning file ownership**:
   - When invoked by `j-orc`, planning files exist; `j-tec` reads and continues from current phase.
   - When invoked directly, `j-tec` creates/reads the planning bundle before any analysis.
   - Never overwrite existing planning files -- append and update phase state.
2. **Evidence-first phase model is persisted**:
   - Required phases: Input validation -> Repository mapping -> Technical research -> Clarifications -> Tech spec drafting -> Verification & handoff.
   - Phase status maintained in `task_plan.md`.
3. **Selective re-read policy (context-aware)**:
   - Re-read `task_plan.md` at phase transitions and before the drafting phase -- not before every tool call.
   - Trade-off: saves context budget for actual code analysis vs. risk of goal drift (mitigated by phase model).
4. **Findings capture (semantic cadence)**:
   - Write evidence to `findings.md` when discovery yields decision-relevant signal: architecture patterns, dependency discoveries, compatibility issues, RF-impacting constraints.
   - Each entry must include source pointer (file path or URL) and implication for the spec.
   - Do not write mechanically after a fixed number of operations.
5. **Decision traceability is explicit**:
   - Maintain a decision log in `findings.md` mapping each decision to evidence and impacted PRD requirements (`RFxx`).
   - Final tech spec must reflect this traceability.
6. **PRD traceability is a hard gate**:
   - Every interface, component, and major flow references the RFs it satisfies.
   - Missing RF mapping blocks completion.
7. **Reuse-over-build policy**:
   - Prefer existing project dependencies/components.
   - New dependencies require justification with alternatives considered.
8. **Incomplete PRD handling**:
   - If PRD has gaps/contradictions, stop and escalate to invoker (`j-orc` or user) before drafting.
   - Do not silently invent requirements.
9. **Error handling: single source of truth**:
   - Errors go in `progress.md` with timestamp, attempt number, and resolution.
   - `task_plan.md` tracks blocker status only -- not a duplicate error table.
   - Never repeat the same failed action; mutate approach and record why.
10. **Resume/recovery discipline**:
    - On restart, read `task_plan.md` first to find incomplete phase, then relevant `findings.md` sections and PRD.
    - Resume from first incomplete phase; append recovery checkpoint to `progress.md`.
11. **Completion gates before handoff**:
    - All phases marked complete.
    - Findings are up to date with final decisions and evidence.
    - Tech spec passes template compliance, RF traceability, and risk disclosure checks.

## Safety Layer

- No fabricated APIs, dependencies, or measurements without explicit assumption markers.
- Never expose secrets or sensitive internals in examples.
- Surface uncertainty and unknowns explicitly.

## Success Criteria

- Tech spec is implementation-ready, template-compliant, and saved at `tasks/prd-[slug]/techspec.md`.
- RF traceability is complete and auditable.
- Key technical decisions are evidence-backed and persisted in `findings.md`.
- Planning files reflect lifecycle state and error/recovery history.
- Completion report includes deterministic artifact path and open-risk visibility.

## Notes for Builder

- Prompt must enforce sequence: PRD read -> repository analysis -> research -> clarifications -> draft.
- Update planning files at phase transitions (not after every tool call).
- Startup checklist: confirm PRD exists -> locate template -> check/create planning bundle -> discover project standards.
- Completion output schema:
  - `STATUS`
  - `FEATURE_SLUG`
  - `TECHSPEC_PATH`
  - `PLANNING_PATHS`
  - `KEY_DECISIONS`
  - `OPEN_RISKS`
