# j-prd — Architecture Spec

## Requirements Summary

- Primary function: produce a high-quality PRD using the project PRD template.
- Mandatory behavior: run domain research and ask clarification questions before drafting.
- Output artifact: `tasks/prd-[feature-slug]/prd.md`.
- Focus: problem, outcomes, requirements, scope boundaries (what/why; not implementation).

## Assumptions

- Template exists at `templates/prd-template.md`.
- If missing, user provides path or approves bootstrap from known structure.
- Conversation can happen in the user language; written artifact is English unless project policy says otherwise.

## Design Decisions

1. **Research-then-clarify workflow**:
   - Before clarifications, run web research on the problem domain and business rules to ask informed questions.
   - Collect problem statement, goals, users, scope boundaries, constraints via `AskUser`.
2. **Template-locked writing**:
   - Preserve section order and intent from PRD template.
   - Number functional requirements (`RF01`, `RF02`, ...).
3. **Output discipline**:
   - Generate slug in kebab-case derived from the feature name (e.g., `weather-dashboard`).
   - Create deterministic output path: `tasks/prd-[slug]/prd.md`.
   - Return concise summary + path + assumptions.
4. **Update/iteration flow**:
   - If `tasks/prd-[slug]/prd.md` already exists, read current version first.
   - Propose incremental edits instead of full rewrite.
   - Preserve existing numbered requirements when adding new ones.
5. **Startup checklist discipline**:
   - Locate `templates/prd-template.md` before authoring.
   - Generate feature slug in kebab-case.
   - Detect whether an existing PRD already exists for incremental update mode.
6. **Execution sequence**:
   - Research -> Clarify -> Plan -> Draft -> Save -> Report.
   - Never skip clarify stage even when updating an existing PRD.

## Safety Layer

- Reject requests for deceptive/harmful product requirements.
- Avoid embedding sensitive data in PRD examples.
- Do not prescribe implementation details beyond high-level technical constraints.

## Success Criteria

- PRD follows template structure exactly.
- Functional requirements are clear, testable, and numbered.
- Out-of-scope section prevents scope creep.
- File saved to expected path with explicit confirmation.

## Notes for Builder

- Prompt should include a hard rule: no PRD generation before clarifying questions.
- Include a short output format for completion reports:
  - `STATUS`
  - `PRD_PATH`
  - `FEATURE_SLUG`
  - `ASSUMPTIONS`
  - `SUMMARY`
