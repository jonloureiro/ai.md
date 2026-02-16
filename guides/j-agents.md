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

Artifacts are written to:

```
tasks/
  prd-[feature-slug]/
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

1. **Detect state** — check which artifacts exist in `tasks/prd-[slug]/`.
2. **Route** — delegate to the appropriate specialist:
   - No PRD → `j-prd`
   - PRD exists, no tech spec → `j-tec`
   - PRD + tech spec, no tasks → `j-exe` (planning mode)
   - Tasks exist → `j-exe` (execution mode)
3. **Gate** — no execution starts before PRD + Tech Spec exist (unless you explicitly override).
4. **Report** — return a structured status block after each phase.

#### Re-entry

If a session is interrupted, invoke `j-orc` again on the same feature. It inspects existing artifacts and resumes from the correct phase.

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

1. **Research** — runs domain research on the problem space.
2. **Clarify** — asks structured questions via `AskUser` (never skipped, even for updates).
3. **Draft** — writes the PRD locked to the template structure.
4. **Save** — outputs to `tasks/prd-[slug]/prd.md`.

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

1. **Read PRD** — fully parses the PRD and its requirements.
2. **Explore project** — maps existing code, modules, dependencies, and integration points.
3. **Research** — investigates technical decisions and library documentation.
4. **Clarify** — asks technical questions via `AskUser`.
5. **Draft** — writes the spec with explicit PRD traceability (`RFxx` → technical decision).
6. **Save** — outputs to `tasks/prd-[slug]/techspec.md`.

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

## Tips

- **Start with `j-orc`** for most feature work. Use individual agents only when you need granular control over a specific phase.
- **Templates matter** — agents are template-locked. Customize templates to match your project standards; the agents will follow them.
- **Incremental updates** — all agents support updating existing artifacts without full rewrites.
- **Phase skipping** — tell `j-orc` which artifacts already exist to skip phases you've handled manually.
