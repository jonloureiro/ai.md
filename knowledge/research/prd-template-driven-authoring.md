# PRD Template-Driven Authoring

> Last updated: 2026-02-16
> Status: Current

## Summary

A reliable PRD workflow should be clarification-first, template-locked, and outcome-focused. The sample project demonstrates a strict sequence: clarify -> plan -> draft -> save. This materially reduces ambiguity before implementation starts.

## Sources

- Local template: `samples/iadevt5_4/templates/prd-template.md`
- Local workflow command: `samples/iadevt5_4/.claude/commands/criar-prd.md`
- Local PRD example: `samples/iadevt5_4/tasks/prd-weather-dashboard/prd.md`
- Atlassian: <https://www.atlassian.com/agile/product-management/requirements>

## Key Findings

### 1) Clarification must happen before drafting

The sample command enforces mandatory clarification questions before generating the PRD. This aligns stakeholders and prevents speculative requirements.

### 2) Template lock improves consistency

Keeping section order and intent fixed (overview, goals, user stories, core features, UX, constraints, out-of-scope) makes PRDs easier to review and operationalize.

### 3) PRD should prioritize what/why

The template explicitly separates high-level technical constraints from implementation details, preserving PRD as a product artifact.

### 4) Requirements should be testable and traceable

Numbered functional requirements (`RFxx`) improve downstream traceability into tech specs, tasks, and tests.

### 5) Out-of-scope is not optional

Explicit exclusions are a practical anti-scope-creep mechanism and improve delivery predictability.

## Implications for This Workspace

- `j-prd` should enforce clarification-first flow.
- `j-prd` should be template-locked to project PRD template.
- Output path convention (`tasks/prd-[slug]/prd.md`) should remain deterministic.
- Completion report should always include path, assumptions, and concise summary.
