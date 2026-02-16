# j-tec — Architecture Spec

## Requirements Summary

- Primary function: convert PRD into implementation-ready technical specification.
- Mandatory behavior: explore project context first, run technical web research, then ask technical clarifications.
- Output artifact: `tasks/prd-[feature-slug]/techspec.md`.
- Template contract: follow `templates/techspec-template.md` exactly.

## Assumptions

- PRD exists at `tasks/prd-[feature-slug]/prd.md` before tech spec generation.
- Template exists at `templates/techspec-template.md`.
- Project coding standards are discoverable (e.g., `.claude/rules`, lint configs, test setup).

## Design Decisions

1. **Evidence-first drafting**:
   - Read PRD completely.
   - Map existing code, modules, dependencies, and integration points.
   - Run web research on technical decisions and library documentation before clarifications.
2. **Reuse-over-build policy**:
   - Prefer existing libraries/components already present in project.
   - Justify custom builds when reuse is not viable.
3. **Template-locked structure**:
   - System architecture, implementation design, testing, sequencing, observability, risks.
4. **Rules mapping**:
   - Explicitly list relevant project rules and compliance decisions.
5. **PRD traceability**:
   - Map functional requirements (`RFxx`) from PRD to corresponding technical decisions.
   - Each interface/component should reference which RFs it satisfies.
6. **Incomplete PRD handling**:
   - If PRD has gaps or contradictions, stop and report missing items to invoker (j-orc or user) before drafting.
   - Do not fill PRD gaps with assumptions; escalate to PRD owner.
7. **Startup checklist discipline**:
   - Confirm PRD exists before drafting.
   - Locate `templates/techspec-template.md`.
   - Discover project standards (rules, lint/test configs) and impacted modules.
8. **Execution sequence**:
   - Read PRD -> analyze repository -> run web research -> clarify -> draft -> map RF traceability -> save.
   - Keep sequence explicit in the prompt to avoid skipping discovery steps.

## Safety Layer

- No fabricated APIs/dependencies without explicit assumptions.
- No exposure of secrets in examples.
- Mark uncertainty and unknowns explicitly.

## Success Criteria

- Tech spec is implementation-ready and traceable to PRD requirements.
- Interfaces, data models, and test strategy are concrete.
- Relevant files/dependencies are listed.
- Output saved at deterministic path with concise completion report.

## Notes for Builder

- Prompt must enforce: do not draft before repository exploration and clarifications.
- Include explicit check for template adherence and rule compliance section.
- Completion output should include:
  - `STATUS`
  - `TECHSPEC_PATH`
  - `KEY_DECISIONS`
  - `OPEN_RISKS`
- Include a quality gate to ensure PRD `RFxx` traceability appears in the final artifact.
