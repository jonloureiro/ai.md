# j-rev and j-qa Architecture Research (Deep)

> Last Updated: 2026-02-17
> Status: Current

## Summary

This research deepens the architecture contract for two new specialist agents in the `j-orc` ecosystem:

- `j-rev`: code review gatekeeper for spec/rules/test compliance.
- `j-qa`: functional QA gatekeeper for E2E, WCAG 2.2 checks, and visual evidence.

The design is evidence-first, planning-with-files aligned, and compatible with two browser-operation paths for QA:

1. Playwright MCP tools (`browser_*`) when available.
2. `agent-browser` skill fallback (Playwright-backed CLI) when MCP tools are unavailable.

## Scope

In scope:

- Operating contracts for `j-rev` and `j-qa`.
- Quality gates and failure behavior.
- Evidence model and report schema.
- Tooling policy for E2E/a11y execution.

Out of scope:

- Modifying `j-orc` routing logic directly.
- Project-specific implementation code.

## Sources

### Local Sources

- `agents/j-orc/architecture-spec.md`
- `agents/j-exe/architecture-spec.md`
- `agents/j-prd/architecture-spec.md`
- `agents/j-tec/architecture-spec.md`
- `guides/j-agents.md`
- `guides/j-agents-extended.md`
- `templates/task_plan.md`
- `templates/findings.md`
- `templates/progress.md`
- `/Users/jon/.agents/skills/agent-browser/SKILL.md`
- `/Users/jon/.agents/skills/agent-browser/references/commands.md`
- `/Users/jon/.agents/skills/agent-browser/references/snapshot-refs.md`

### External Sources

- Playwright best practices: <https://playwright.dev/docs/best-practices>
- Playwright locators: <https://playwright.dev/docs/locators>
- Playwright isolation/browser contexts: <https://playwright.dev/docs/browser-contexts>
- Playwright release notes (accessibility API deprecation): <https://playwright.dev/python/docs/release-notes>
- WCAG 2.2 normative: <https://www.w3.org/TR/WCAG22/>
- WCAG 2.2 Focus Not Obscured (Minimum): <https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-minimum.html>
- WCAG 2.2 Focus Appearance: <https://www.w3.org/WAI/WCAG22/Understanding/focus-appearance.html>
- WCAG 2.2 Labels or Instructions: <https://www.w3.org/WAI/WCAG22/Understanding/labels-or-instructions.html>
- Google review guidance: <https://google.github.io/eng-practices/review/reviewer/looking-for.html>
- GitHub PR review workflow: <https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-proposed-changes-in-a-pull-request>
- OWASP secure code review cheat sheet: <https://cheatsheetseries.owasp.org/cheatsheets/Secure_Code_Review_Cheat_Sheet.html>

## Evidence Ledger

| Claim | Evidence (path/url) | Why it matters |
|---|---|---|
| `j-*` pipeline requires persistent planning state | `agents/j-orc/architecture-spec.md` | `j-rev`/`j-qa` must read/write `task_plan.md`, `findings.md`, `progress.md`. |
| Specialists are phase-gated and evidence-first | `agents/j-exe/architecture-spec.md` | New agents should fail closed when gates are unmet. |
| Test isolation is mandatory for E2E reliability | <https://playwright.dev/docs/best-practices> | `j-qa` should avoid state-leaking flows and re-initialize context between RFs when needed. |
| Locator strategy should prioritize user-facing semantics | <https://playwright.dev/docs/locators> | `j-qa` should prefer roles/labels/text over brittle CSS selectors. |
| Browser context isolation is first-class in Playwright | <https://playwright.dev/docs/browser-contexts> | Supports deterministic retries and multi-user validation patterns. |
| Playwright accessibility API is deprecated in favor of external libs | <https://playwright.dev/python/docs/release-notes> | `j-qa` should rely on a11y tree/snapshots + rule-based checks and optional axe integration where available. |
| WCAG 2.2 is the normative baseline | <https://www.w3.org/TR/WCAG22/> | `j-qa` must enforce WCAG criteria references in findings/report. |
| Focused elements must remain visible/not obscured | <https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-minimum.html> | `j-qa` keyboard testing must include focus-visibility checks under sticky headers/overlays. |
| Focus indicator needs visible area/contrast properties | <https://www.w3.org/WAI/WCAG22/Understanding/focus-appearance.html> | `j-qa` must check focus style quality, not just keyboard traversal. |
| Form controls require labels/instructions | <https://www.w3.org/WAI/WCAG22/Understanding/labels-or-instructions.html> | `j-qa` must verify field-level semantics and error instruction clarity. |
| Code review quality dimensions include design, functionality, complexity, tests | <https://google.github.io/eng-practices/review/reviewer/looking-for.html> | `j-rev` checklist should cover these dimensions explicitly. |
| PR review should be contextual and file-by-file | <https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-proposed-changes-in-a-pull-request> | `j-rev` should enforce full changed-file inspection, not diff fragments only. |
| Security review should include baseline and diff-based methods | <https://cheatsheetseries.owasp.org/cheatsheets/Secure_Code_Review_Cheat_Sheet.html> | `j-rev` must apply security checks to both branch diff and modified-file context. |
| `agent-browser` provides Playwright-backed command workflow with snapshot refs | `/Users/jon/.agents/skills/agent-browser/SKILL.md` | `j-qa` can execute robust E2E even without direct MCP browser tools. |
| Ref invalidation requires re-snapshot after page changes | `/Users/jon/.agents/skills/agent-browser/references/snapshot-refs.md` | `j-qa` interaction loop must force fresh snapshots after navigation and dynamic updates. |

## Key Findings

### 1) `j-rev` should act as a hard quality gate, not a passive reviewer

- Required inputs: `prd.md`, `techspec.md`, `tasks.md`, `task_plan.md`, project rules file(s), git diff.
- Required reject conditions:
  - Any failing required validator.
  - Any critical compliance/security issue.
  - Missing mandatory inputs.

### 2) `j-rev` must combine three perspectives

1. **Spec fidelity** (PRD + Tech Spec + Tasks).
2. **Repository rules compliance** (`.claude/rules` or equivalent policy files).
3. **Execution proof** (tests/lint/typecheck where available).

### 3) `j-qa` should validate by requirement coverage, not by ad-hoc UI exploration

- Every PRD RF must map to at least one executable flow and one evidence item.
- Final status cannot be `APPROVED` unless RF coverage is complete and critical bugs are zero.

### 4) WCAG 2.2 checks should be explicit and reportable

- Focus visibility/not obscured.
- Labels/instructions for form controls.
- Keyboard operability and predictable focus order.
- Clear evidence (snapshot/screenshot + step trace).

### 5) Tooling policy should support both MCP and `agent-browser`

- Primary mode: `browser_*` MCP calls when available.
- Fallback mode: invoke `Skill("agent-browser")` and use snapshot-ref workflow.
- Both modes require identical evidence standards in findings/report artifacts.

### 6) Planning-with-files discipline is mandatory for both agents

- `task_plan.md`: phase + gate + blocker state.
- `findings.md`: requirement-level evidence and bug logs.
- `progress.md`: timestamped action + command history + failures/retries.

### 7) Structured output contracts improve orchestration reliability

- `j-rev` status block should include: phase, issues count, test status, next action.
- `j-qa` status block should include: current RF, bugs count, mode (MCP vs agent-browser), next action.

## Recommended Agent Contracts

### `j-rev`

- Status states: `analyzing | testing | reporting | completed | blocked`
- Final verdicts: `PASS | REQUEST_CHANGES`
- Mandatory report artifact: `tasks/prd-[slug]/review-report.md`

### `j-qa`

- Status states: `planning | testing | accessibility | reporting | completed | blocked`
- Final verdicts: `APPROVED | REJECTED`
- Mandatory report artifact: `tasks/prd-[slug]/qa-report.md`

## Unknowns

- `UNKNOWN`: Some target repositories may not expose `.claude/rules`; a fallback hierarchy is needed (`.claude/rules` -> `AGENTS.md` -> `CLAUDE.md` -> repo lint/test configs).
- `UNKNOWN`: Playwright MCP availability differs by runtime; runtime capability detection is required before starting E2E.

## Implications for This Workspace

- Create two new agent artifacts: `agents/j-rev/architecture-spec.md` and `agents/j-qa/architecture-spec.md`.
- Ensure both define planning-with-files persistence and strict completion gates.
- Ensure `j-qa` explicitly documents `agent-browser` fallback behavior with equivalent evidence requirements.
