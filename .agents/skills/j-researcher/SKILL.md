---
name: j-researcher
description: Run deep, evidence-only research for j-agents (`j-orc`, `j-prd`, `j-tec`, `j-exe`). Use when you need date-verified, knowledge-first, high-parallel investigation with mandatory persistence of findings into `./knowledge`.
---

# j-researcher

Run reliable deep research for agent architecture, workflow rules, tooling behavior, and implementation constraints.

## Primary Target and Compatibility

- Primary target: Factory Droid workflows.
- Also compatible with Gemini CLI, Claude Code, OpenCode, and similar agent environments.
- Keep methods platform-agnostic; adapt only the tool names, never the quality gates.

## Mandatory Startup Protocol

Before any research operation, execute in this order:

1. **Date Gate (required first)**
   - Verify current date/time from runtime.
   - Record timestamp in notes/output.
   - If date cannot be verified, stop and report `BLOCKED`.

2. **Knowledge Gate**
   - Read `knowledge/INDEX.md`.
   - Read relevant files in `knowledge/research/` before external sources.

3. **Scope Gate**
   - Restate objective and expected deliverable.
   - Define what is in-scope vs out-of-scope.

Do not start external research before all three gates pass.

## Non-Negotiable Rules

1. **Knowledge-first only**
   - Never use unstated "common knowledge" for critical claims.
   - Each important claim must map to explicit evidence.

2. **Evidence-only reasoning**
   - Separate facts from inference.
   - Label uncertain items as `UNKNOWN`.

3. **High-parallel research is mandatory**
   - Use at least two parallel waves:
     - Wave 1: **>= 8** parallel operations.
     - Wave 2: **>= 6** parallel operations.
   - Do not draft final conclusions before both waves complete.

4. **Dual-source j-agent investigation (when j-agents are in scope)**
   - Investigate local `agents/` j-agent specs and an external `j-agents.md` reference in parallel.
   - If external file path is missing/inaccessible, mark `UNKNOWN` and continue with available evidence.

5. **Persistence is required**
   - Save findings to `knowledge/research/<topic>.md`.
   - Update `knowledge/INDEX.md` in the same task.

6. **Strict file-scope guardrail**
   - Never alter source code or implementation files.
   - Allowed writes are limited to:
     - `knowledge/research/*.md`
     - `knowledge/INDEX.md`
   - If a request targets files outside this scope, ask for explicit user confirmation before continuing.
   - If scope is not narrowed back to allowed paths, return `STATUS: blocked` and do not edit.

## Parallel Research Workflow

### Wave 1 — Landscape Mapping (>= 8 parallel ops)

Collect broad evidence in parallel across at least four classes:

- Local architecture specs (`agents/`, `skills/`, workspace policy files)
- Local knowledge base (`knowledge/research/*.md`)
- Platform docs / vendor docs / source references
- Existing deployed prompts/skills/droids related to the topic

### Wave 2 — Gap Closing (>= 6 parallel ops)

Run targeted parallel checks for:

- Contradictions between sources
- Missing constraints or unsupported assumptions
- Version/date freshness issues
- Tool behavior ambiguities

### Wave 3 — Optional Validation Sweep

Use when confidence is low or contradictions remain. Keep it parallel and focused.

## Required Evidence Ledger

Maintain this ledger before final synthesis:

```markdown
| Claim | Evidence (path/url) | Why it matters |
|---|---|---|
| ... | ... | ... |
```

Requirements:
- Minimum 10 rows for non-trivial research.
- Every hard recommendation must map to at least one row.

## Persistence Contract (`./knowledge`)

At the end of each research task:

1. Create/update a file in `knowledge/research/` with:
   - title
   - last updated date
   - status (`Current`, `Aging`, `Stale`)
   - sources
   - key findings
   - implications
2. Add or update corresponding entry in `knowledge/INDEX.md`.
3. Include file paths in final response.

Work is incomplete if persistence is missing.

## Output Contract

Return results in this structure:

```markdown
STATUS: <completed|blocked|partial>
DATE_VERIFIED: <timestamp>
SCOPE:
- ...

SOURCES_USED:
- <path-or-url>

EVIDENCE_SUMMARY:
- <claim -> source -> implication>

KNOWLEDGE_UPDATES:
- <knowledge/research/file.md>
- <knowledge/INDEX.md updated: yes|no>

UNKNOWNS:
- <item or none>

NEXT_ACTIONS:
- <concrete next step>
```

## Completion Gates

- [ ] Date verified before research
- [ ] `knowledge/INDEX.md` reviewed first
- [ ] Two mandatory parallel waves completed (>= 8, >= 6)
- [ ] Evidence ledger complete and internally consistent
- [ ] No unsupported assumptions in conclusions
- [ ] Findings persisted to `knowledge/research/*.md`
- [ ] `knowledge/INDEX.md` updated
- [ ] No code or out-of-scope file edits attempted
- [ ] `UNKNOWN` items explicitly listed with follow-up actions

## Failure Behavior

- If evidence is insufficient, do not fabricate.
- Return `STATUS: blocked` or `STATUS: partial` with exact missing evidence.
- Provide the smallest high-impact next research step to unblock progress.
- If asked to modify code or files outside `knowledge/research/*.md` and `knowledge/INDEX.md`, request explicit confirmation and keep `STATUS: blocked` unless scope is corrected.
