# Tech Spec Template-Driven Authoring

> Last updated: 2026-02-16
> Status: Current

## Summary

High-quality technical specs should be evidence-first: read PRD, inspect codebase, clarify unknowns, and only then draft. A template-locked format ensures architecture decisions, interfaces, testing, sequencing, and risks are explicit and implementable.

## Sources

- Local template: `samples/iadevt5_4/templates/techspec-template.md`
- Local workflow command: `samples/iadevt5_4/.claude/commands/criar-techspec.md`
- Local example: `samples/iadevt5_4/tasks/prd-weather-dashboard/techspec.md`
- Google doc practices: <https://google.github.io/styleguide/docguide/best_practices.html>
- AWS ADR guidance: <https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/welcome.html>

## Key Findings

### 1) Repository analysis before writing is essential

The sample command explicitly requires deep project exploration before clarification and drafting, reducing architecture drift.

### 2) Tech Spec focuses on how

The template separates implementation design from product intent and avoids duplicating PRD content.

### 3) Reuse-first decision policy reduces risk

The sample guidance prefers existing libraries and components before introducing custom solutions.

### 4) Rules compliance should be explicit

Mapping architecture decisions against project rules/policies prevents non-conforming implementations.

### 5) Testing and observability belong in the spec

Including unit/integration/e2e strategy and monitoring expectations early improves delivery quality and reviewability.

## Implications for This Workspace

- `j-tec` should enforce: PRD read -> codebase analysis -> clarifications -> draft.
- `j-tec` should keep strict template adherence.
- `j-tec` outputs should include rationale, trade-offs, risks, and relevant files.
- Tech spec completion should always include path + key decisions + open risks.
