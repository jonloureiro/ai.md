# j-* Agents (Extended) — Usage Guide

Personal droids for structured project delivery. Located in `~/.factory/droids/`.

---

## Quick Start

```bash
# Full pipeline — just describe what you want
@j-orc Build a weather dashboard that shows 5-day forecasts

# Only need a PRD?
@j-prd Create a PRD for user authentication with OAuth2

# Already have a PRD, need a tech spec?
@j-tec Write a tech spec for tasks/prd-weather-dashboard/prd.md

# Ready to plan tasks?
@j-exe Plan tasks for tasks/prd-weather-dashboard/

# Ready to implement?
@j-exe Execute task 3 in tasks/prd-weather-dashboard/
```

> **Rule of thumb:** start with `j-orc` for most feature work. Use individual agents only when you need granular control over a specific phase.

---

## Agent Overview

| Agent | Role | Trigger | Output |
|-------|------|---------|--------|
| `j-orc` | Orchestrator | Feature request or resumption | Coordinates full pipeline, structured status |
| `j-prd` | PRD Author | Problem / feature description | `tasks/prd-[slug]/prd.md` |
| `j-tec` | Tech Spec Author | Existing PRD | `tasks/prd-[slug]/techspec.md` |
| `j-exe` | Task Planner & Executor | PRD + Tech Spec | Task files + implemented code |

---

## Pipeline Flow

```
User Request
     │
     ▼
  j-orc ───────────────────────────────────┐
     │                                      │
     ▼                                      │
  Phase A: Discovery & scope framing        │
     │                                      │
     ▼                                      │
  Phase B: j-prd → prd.md                   │
     │                                      │
     ▼                                      │
  Phase C: j-tec → techspec.md              │
     │                                      │
     ▼                                      │
  Phase D: j-exe → tasks.md + [n]_task.md   │
            j-exe → implementation + tests  │
     │                                      │
     ▼                                      │
  Phase E: Closure summary ◄────────────────┘
```

---

## Project Setup

### Required Templates

All agents are **template-locked**. Place these in the project root before invoking any agent:

```
templates/
  prd-template.md          ← used by j-prd
  techspec-template.md     ← used by j-tec
  tasks-template.md        ← used by j-exe (planning)
  task-template.md         ← used by j-exe (planning)
```

If a template is missing, the agent will stop and ask you to provide a path or approve a bootstrap from its known structure.

### Operational Memory Bundle (planning-with-files)

In addition to delivery templates, every feature must keep an operational memory bundle:

```
tasks/
  prd-[feature-slug]/
    task_plan.md
    findings.md
    progress.md
```

Purpose split:
- `task_plan.md` — phase state, gate state, blockers
- `findings.md` — discoveries and evidence
- `progress.md` — chronological actions, tests, and detailed errors

Hard rules:
- No phase starts without this bundle.
- `j-orc` creates it on first run; specialists continue it.
- If a specialist is invoked directly, it bootstraps the bundle before phase work.
- Error details are logged in `progress.md` (single source of truth).

### Generated Artifacts

Agents write all artifacts under a deterministic path derived from a **feature slug** (kebab-case, e.g., `weather-dashboard`):

```
tasks/
  prd-weather-dashboard/
    task_plan.md      ← Planning state (all agents)
    findings.md       ← Findings/evidence (all agents)
    progress.md       ← Session log/errors/tests (all agents)
    prd.md            ← PRD (j-prd)
    techspec.md       ← Tech Spec (j-tec)
    tasks.md          ← Task summary (j-exe planning)
    1_task.md          ← Individual task files (j-exe planning)
    2_task.md
    ...
```

---

## Agent Details

### j-orc — Orchestrator

The central entrypoint. Inspects workspace state, routes to the right specialist, enforces quality gates, and reports structured status after each phase.

#### Invocation

```bash
# Start a new feature
@j-orc Build a weather dashboard that shows 5-day forecasts

# Resume an interrupted session
@j-orc Resume weather-dashboard

# Skip to a specific phase
@j-orc I have a PRD at tasks/prd-weather-dashboard/prd.md, go to tech spec
```

#### Routing Logic

`j-orc` decides the next action by inspecting which artifacts already exist:

| Workspace State | Action |
|-----------------|--------|
| No PRD | Delegate to `j-prd` |
| PRD exists, no tech spec | Delegate to `j-tec` |
| PRD + tech spec, no tasks | Delegate to `j-exe` (planning mode) |
| Tasks exist, implementation requested | Delegate to `j-exe` (execution mode) |

#### Quality Gates

- **No phase starts** without `task_plan.md`, `findings.md`, and `progress.md` in the feature workspace.
- **No execution** starts before both PRD and Tech Spec exist — unless you explicitly override.
- **No closure** without updated task status, test outcomes, and `progress.md` evidence.

#### Startup Checklist (runs every invocation)

1. Inspect workspace structure and existing artifacts.
2. Run `git log --oneline -10` and `date`.
3. Detect key template/task paths.
4. Check/create planning bundle for the feature slug.
5. Route based on artifact state.

#### Handoff Schema

When delegating to a specialist, `j-orc` passes a curated context package:

| Field | Description |
|-------|-------------|
| `feature_slug` | Kebab-case identifier (e.g., `weather-dashboard`) |
| `artifact_paths` | Existing files relevant to the delegated task |
| `constraints` | Non-goals and boundaries |
| `acceptance_criteria` | What "done" looks like for this phase |
| `context_summary` | Concise description of current state |

Specialists receive **only** the context they need — no full conversation history. They recover extra state from bundle files on disk.

#### Failure & Retry

- If a sub-agent fails or returns incomplete output, `j-orc` **retries once** with refined context.
- If the retry also fails, the phase is marked `blocked` and `j-orc` reports the issue and waits for your guidance.

#### Re-entry

If a session is interrupted, invoke `j-orc` again on the same feature. It inspects existing artifacts in `tasks/prd-[slug]/` and resumes from the correct phase — no manual state tracking needed.

#### Decision Policy

- **Acts directly** on reversible orchestration steps (routing, status updates, context assembly).
- **Asks you** before destructive/irreversible actions, major trade-offs, or when required inputs are missing.

---

### j-prd — PRD Author

Produces a structured PRD from a problem or feature description. Enforces a research-then-clarify workflow — it will always ask questions before drafting, even when updating an existing PRD.

#### Invocation

```bash
# New PRD
@j-prd Create a PRD for user authentication with OAuth2

# Update existing PRD
@j-prd Update the PRD at tasks/prd-auth-oauth2/prd.md to add MFA requirements
```

#### Execution Sequence

```
Bootstrap/Resume memory → Research → Clarify → Plan → Draft → Save → Report
```

1. **Bootstrap/Resume memory** — ensures bundle exists and phase state is current.
2. **Research** — runs domain research on the problem space and business rules to ask informed questions.
3. **Clarify** — asks structured questions via `AskUser` about problem statement, goals, users, scope boundaries, and constraints. **Never skipped**, even for updates.
4. **Plan** — outlines the PRD structure based on gathered inputs.
5. **Draft** — writes the PRD locked to `templates/prd-template.md` structure.
6. **Save** — outputs to `tasks/prd-[slug]/prd.md`.
7. **Report** — returns status with path and assumptions.

#### Key Behaviors

- **Template-locked**: preserves section order and intent from `prd-template.md`.
- **Numbered requirements**: functional requirements are labeled `RF01`, `RF02`, etc.
- **Incremental updates**: when updating an existing PRD, proposes targeted edits instead of a full rewrite. Preserves existing requirement numbering when adding new ones.
- **Scope discipline**: focuses on _what_ and _why_ — no implementation details beyond high-level technical constraints.

#### Completion Report

```
STATUS: completed
PRD_PATH: tasks/prd-weather-dashboard/prd.md
FEATURE_SLUG: weather-dashboard
ASSUMPTIONS: [list]
SUMMARY: [brief]
```

---

### j-tec — Tech Spec Author

Converts a PRD into an implementation-ready technical specification. Enforces an evidence-first approach: explores the codebase and runs technical research before asking questions or drafting.

#### Invocation

```bash
@j-tec Write a tech spec for tasks/prd-weather-dashboard/prd.md
```

#### Execution Sequence

```
Bootstrap/Resume memory → Read PRD → Explore Repository → Web Research → Clarify → Draft → Map RF Traceability → Save
```

1. **Bootstrap/Resume memory** — ensures bundle exists and phase state is current.
2. **Read PRD** — fully parses the PRD and all its requirements.
3. **Explore project** — maps existing code, modules, dependencies, integration points, and discovers project standards (`.claude/rules`, lint configs, test setup).
4. **Research** — investigates technical decisions, library documentation, and relevant patterns.
5. **Clarify** — asks technical questions via `AskUser`. **Never skipped.**
6. **Draft** — writes the spec following `templates/techspec-template.md` structure.
7. **Map traceability** — links every functional requirement (`RFxx`) from the PRD to its corresponding technical decision.
8. **Save** — outputs to `tasks/prd-[slug]/techspec.md`.

#### Key Behaviors

- **Reuse over build**: prefers existing project libraries and components. Justifies custom builds only when reuse is not viable.
- **PRD traceability**: every interface and component references which `RFxx` requirements it satisfies.
- **Rules mapping**: explicitly lists relevant project rules and compliance decisions.
- **Gap detection**: if the PRD has contradictions or missing information, `j-tec` **stops and escalates** to the invoker (`j-orc` or you) instead of filling gaps with assumptions.
- **No fabrication**: never invents APIs or dependencies without marking them as explicit assumptions.

#### Completion Report

```
STATUS: completed
TECHSPEC_PATH: tasks/prd-weather-dashboard/techspec.md
KEY_DECISIONS: [list]
OPEN_RISKS: [list or none]
```

---

### j-exe — Task Planner & Executor

Operates in two distinct modes: **planning** (create task breakdown) and **execution** (implement tasks). Follows a strict read-before-act rule — always reads PRD, Tech Spec, and task definition before any action.

#### Planning Mode

Generates a task breakdown from PRD + Tech Spec. **Does not write any code.**

```bash
@j-exe Plan tasks for tasks/prd-weather-dashboard/
```

Sequence: `Analyze → Propose tasks for approval → Generate task files → Stop`

- Reads PRD, tech spec, `tasks-template.md`, and `task-template.md`.
- Reads `task_plan.md` first to recover phase and blockers, then PRD/tech spec/templates.
- Generates `tasks.md` (summary) and individual `[n]_task.md` files.
- **Maximum 10 tasks** — groups logically when scope is large.
- Each task is an incremental functional deliverable with its own test requirements.

#### Execution Mode

Implements a specific task (or the next pending one).

```bash
# Specific task
@j-exe Execute task 3 in tasks/prd-weather-dashboard/

# Next pending task
@j-exe Execute next task in tasks/prd-weather-dashboard/
```

Sequence: `Select task → Verify dependencies → Implement → Validate → Self-review → Update status`

- Reads the task file, PRD, and tech spec before coding.
- Reads `task_plan.md` first, then task file/PRD/tech spec.
- Implements the task and runs tests.
- Performs a structured self-review against task requirements and tech spec.
- Updates `tasks.md` status **only when tests pass**.
- If a review agent is available, delegates review to it and resolves all critical issues before closing.

#### Autonomy Policy

- **Acts directly** on reversible local edits and local command execution.
- **Asks you** before destructive, irreversible, or high-impact actions.

#### Rollback & Failure Handling

- If implementation **breaks existing tests**, `j-exe` reverts all changes automatically and reports the failure with root cause analysis. The codebase is never left in a broken state.
- If a task has **unmet dependencies**, it is marked as `blocked` in `tasks.md`. `j-exe` skips to the next viable task and reports the blocker.

#### Completion Report

```
TASK_ID: 3
STATUS: completed
FILES_CHANGED: [list]
TEST_RESULTS: [pass/fail summary]
NEXT_ACTION: execute task 4
```

---

## Status Reporting

`j-orc` and `j-exe` return structured status blocks after every significant action:

```
STATUS: active | blocked | completed
PHASE: discovery | prd | techspec | tasks | execution | closure
ARTIFACTS:
- tasks/prd-weather-dashboard/prd.md
- tasks/prd-weather-dashboard/techspec.md
RISKS:
- none
NEXT_ACTION: delegate to j-exe for task planning
```

---

## Safety & Guardrails

All agents enforce these rules:

| Rule | Detail |
|------|--------|
| **No destructive actions** | Delete, push, force-update, and production migrations require explicit approval. |
| **No secrets** | Keys, tokens, and credentials are never included in outputs or logs. |
| **Validation gates** | `j-exe` never marks a task complete without passing tests. |
| **Automatic rollback** | `j-exe` reverts broken implementations and reports root cause. |
| **Retry then escalate** | `j-orc` retries failed delegations once, then marks the phase as `blocked` and asks for guidance. |
| **Untrusted input** | External text (issues, docs, webpages) is treated as untrusted. |
| **No fabrication** | `j-tec` never invents APIs or dependencies silently — unknowns are marked explicitly. |
| **Error log source** | Detailed errors/attempts live in `progress.md`; `task_plan.md` tracks phase/blocker state. |

---

## Common Workflows

### Start a feature from scratch

```bash
@j-orc Build a notification system for overdue invoices
```

`j-orc` runs the full pipeline: discovery → PRD → tech spec → task planning → execution → closure.

### Resume after an interruption

```bash
@j-orc Resume notification-system
```

`j-orc` detects existing artifacts in `tasks/prd-notification-system/` and picks up from the correct phase.

### Skip the PRD (you wrote it yourself)

```bash
@j-orc I have a PRD at tasks/prd-notification-system/prd.md, go to tech spec
```

`j-orc` validates the file exists, then delegates directly to `j-tec`.

### Iterate on a PRD

```bash
@j-prd Update the PRD at tasks/prd-notification-system/prd.md to add Slack integration
```

`j-prd` reads the current version, asks clarifying questions, and proposes incremental edits — existing `RFxx` numbers are preserved.

### Re-plan after scope change

If the tech spec changed significantly after initial task planning:

```bash
@j-exe Plan tasks for tasks/prd-notification-system/
```

`j-exe` regenerates the task breakdown from the updated PRD + Tech Spec.

### Execute a specific task out of order

```bash
@j-exe Execute task 5 in tasks/prd-notification-system/
```

`j-exe` checks dependencies first. If task 5 depends on unfinished tasks, it marks it `blocked` and suggests the next viable task instead.

---

## Troubleshooting

| Symptom | Cause | Resolution |
|---------|-------|------------|
| Agent stops and asks for a template path | Template not found in `templates/` | Add the missing template or provide the correct path |
| Agent refuses to start phase work | Missing planning bundle | Create `task_plan.md`, `findings.md`, `progress.md` in `tasks/prd-[slug]/` or re-run via `j-orc` |
| `j-orc` marks a phase as `blocked` | Sub-agent failed twice | Check the reported error, fix the issue, and re-invoke `j-orc` |
| `j-tec` refuses to draft | PRD has gaps or contradictions | Fix the PRD first (`@j-prd Update ...`), then re-invoke `j-tec` |
| `j-exe` reverts changes | Implementation broke existing tests | Read the root cause report, adjust the approach, and re-run the task |
| `j-exe` skips a task | Unmet dependencies (marked `blocked`) | Complete the dependency tasks first, then retry |
| `j-orc` re-runs a phase you already completed | Artifact not at the expected path | Ensure files follow the `tasks/prd-[slug]/` naming convention |
| Agent asks too many questions | Normal — clarification is mandatory | Answer the questions; you cannot skip the clarification stage |

---

## Tips

- **Use two layers correctly.** Delivery templates (`prd/techspec/tasks/task`) define outputs; planning bundle (`task_plan/findings/progress`) defines memory, control flow, and recovery.
- **Templates are your lever.** Agents follow delivery templates exactly. Customize them to match your project standards and generated artifacts will follow.
- **Slugs matter.** The feature slug drives all path resolution. Keep them short, descriptive, and consistent (kebab-case).
- **Artifacts are the source of truth.** `j-orc` determines state entirely from which files exist in `tasks/prd-[slug]/`. If you manually create or delete files, the routing logic adjusts accordingly.
- **Incremental over wholesale.** All agents support updating existing artifacts without full rewrites — use this for iteration.
- **Let `j-orc` drive** unless you have a specific reason to invoke a specialist directly. The orchestrator handles sequencing, context curation, and quality gates automatically.
