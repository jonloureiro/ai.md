# j-* Agents — Usage Guide

Personal droids for structured project delivery. Located in `~/.factory/droids/`.

## Overview

| Agent | Role | Input | Output |
|---|---|---|---|
| `j-orc` | Orchestrator | Feature request | Coordinates full pipeline |
| `j-prd` | PRD Author | Problem/feature description | `tasks/prd-[slug]/prd.md` |
| `j-tec` | Tech Spec Author | PRD | `tasks/prd-[slug]/techspec.md` |
| `j-exe` | Task Planner & Executor | PRD + Tech Spec | Task files + implemented code |

## Pipeline Flow

```
User Request
     │
     ▼
  j-orc  ──────────────────────────────────┐
     │                                      │
     ▼                                      │
  j-prd  → prd.md                           │
     │                                      │
     ▼                                      │
  j-tec  → techspec.md                      │
     │                                      │
     ▼                                      │
  j-exe  → tasks.md + [n]_task.md → code    │
     │                                      │
     ▼                                      │
  Closure summary ◄────────────────────────┘
```

## Prerequisites

All agents expect a `templates/` directory in the project root:

```
templates/
  prd-template.md
  techspec-template.md
  tasks-template.md
  task-template.md
```

Agents also require a **planning-with-files bundle** inside each feature workspace:

```
tasks/
  prd-[feature-slug]/
    task_plan.md
    findings.md
    progress.md
```

Hard rules:
- No phase starts without the bundle.
- `j-orc` creates the bundle on first run; specialists continue it.
- If a specialist is invoked directly, it bootstraps the bundle before proceeding.
- Error details are logged in `progress.md` (single source of truth).

Artifacts are written to:

```
tasks/
  prd-[feature-slug]/
    task_plan.md
    findings.md
    progress.md
    prd.md
    techspec.md
    tasks.md
    1_task.md
    2_task.md
    ...
```

## Usage

### Full Pipeline (j-orc)

Use `j-orc` when you want end-to-end delivery from a feature request. It routes work automatically based on which artifacts already exist.

```
@j-orc Build a weather dashboard that shows 5-day forecasts
```

`j-orc` will:

1. **Bootstrap memory** — create or resume `task_plan.md`, `findings.md`, and `progress.md`.
2. **Detect state** — check which artifacts exist in `tasks/prd-[slug]/`.
3. **Route** — delegate to the appropriate specialist:
   - No PRD → `j-prd`
   - PRD exists, no tech spec → `j-tec`
   - PRD + tech spec, no tasks → `j-exe` (planning mode)
   - Tasks exist → `j-exe` (execution mode)
4. **Gate** — no execution starts before PRD + Tech Spec exist (unless you explicitly override).
5. **Report** — return a structured status block after each phase.

#### Re-entry

If a session is interrupted, invoke `j-orc` again on the same feature. It reads `task_plan.md` first, reconciles artifact state, and resumes from the correct phase.

#### Skipping Phases

If you already have a PRD:

```
@j-orc I have a PRD at tasks/prd-weather-dashboard/prd.md, go to tech spec
```

`j-orc` validates the artifact exists before skipping.

### PRD Only (j-prd)

Use `j-prd` directly when you only need a PRD and want fine-grained control over the clarification process.

```
@j-prd Create a PRD for user authentication with OAuth2
```

Workflow:

1. **Bootstrap/Resume memory** — ensure the planning bundle exists and current phase is set.
2. **Research** — runs domain research on the problem space.
3. **Clarify** — asks structured questions via `AskUser` (never skipped, even for updates).
4. **Draft** — writes the PRD locked to the template structure.
5. **Save** — outputs to `tasks/prd-[slug]/prd.md`.

To update an existing PRD:

```
@j-prd Update the PRD at tasks/prd-auth-oauth2/prd.md to add MFA requirements
```

It reads the current version and proposes incremental edits, preserving existing requirement numbering (`RF01`, `RF02`, ...).

### Tech Spec Only (j-tec)

Use `j-tec` when you have a PRD and need a technical specification.

```
@j-tec Write a tech spec for tasks/prd-weather-dashboard/prd.md
```

Workflow:

1. **Bootstrap/Resume memory** — ensure the planning bundle exists and current phase is set.
2. **Read PRD** — fully parses the PRD and its requirements.
3. **Explore project** — maps existing code, modules, dependencies, and integration points.
4. **Research** — investigates technical decisions and library documentation.
5. **Clarify** — asks technical questions via `AskUser`.
6. **Draft** — writes the spec with explicit PRD traceability (`RFxx` → technical decision).
7. **Save** — outputs to `tasks/prd-[slug]/techspec.md`.

Key behaviors:

- **Reuse over build** — prefers existing project libraries and components.
- **Gap detection** — if the PRD has contradictions or missing info, it stops and escalates instead of assuming.
- **Rules mapping** — lists relevant project rules (lint, test configs) and compliance decisions.

### Task Planning & Execution (j-exe)

`j-exe` has two modes:

#### Planning Mode

Generate task breakdown from PRD + Tech Spec:

```
@j-exe Plan tasks for tasks/prd-weather-dashboard/
```

- Reads PRD, tech spec, and templates.
- Reads `task_plan.md` first, then PRD, tech spec, and templates.
- Generates `tasks.md` (summary) and individual `[n]_task.md` files.
- Maximum 10 tasks; groups logically when scope is large.
- Each task is an incremental functional deliverable with test requirements.
- **Does not write code** — planning mode only creates task files.

#### Execution Mode

Implement a specific task:

```
@j-exe Execute task 3 in tasks/prd-weather-dashboard/
```

- Reads the task file, PRD, and tech spec before coding.
- Reads `task_plan.md` first to recover phase and blockers before coding.
- Implements the task, runs tests, and self-reviews.
- Updates `tasks.md` status only when tests pass.
- If implementation breaks existing tests, reverts changes and reports the failure.
- If a task has unmet dependencies, marks it `blocked` and moves to the next viable task.

## Status Reporting

`j-orc` and `j-exe` use structured status blocks:

```
STATUS: active|blocked|completed
PHASE: discovery|prd|techspec|tasks|execution|closure
ARTIFACTS:
- tasks/prd-weather-dashboard/prd.md
RISKS:
- none
NEXT_ACTION: delegate to j-tec for tech spec
```

## Safety & Guardrails

All agents share these rules:

- **No destructive actions** without explicit approval (delete, push, force-update).
- **No secrets** in outputs or logs.
- **Validation gates** — `j-exe` never marks a task complete without passing tests.
- **Rollback** — `j-exe` reverts broken implementations automatically.
- **Retry policy** — `j-orc` retries failed delegations once, then marks the phase as blocked and asks for guidance.
- **Error log source** — detailed errors and attempts live in `progress.md`; `task_plan.md` tracks phase/blocker state.

## Tips

- **Start with `j-orc`** for most feature work. Use individual agents only when you need granular control over a specific phase.
- **Two layers, two purposes** — delivery templates (`prd/techspec/tasks/task`) define outputs; planning bundle (`task_plan/findings/progress`) defines agent memory and control flow.
- **Templates matter** — agents are template-locked for delivery artifacts. Customize them to match your project standards.
- **Incremental updates** — all agents support updating existing artifacts without full rewrites.
- **Phase skipping** — tell `j-orc` which artifacts already exist to skip phases you've handled manually.
