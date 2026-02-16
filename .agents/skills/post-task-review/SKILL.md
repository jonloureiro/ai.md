---
name: post-task-review
description: Post-task review workflow that generates git commit messages. Reviews changed files for coherence, verifies documentation alignment (README, CLAUDE.md, AGENTS.md), detects workspace conventions, and generates appropriate commit messages. READ-ONLY: uses git for inspection only, NEVER executes git commit/add/push. Adds (WIP) prefix if issues found.
license: Complete terms in LICENSE.txt
---

# Post-Task Review

Review completed changes and generate a commit message suggestion.

## Core Constraint (Critical)

This skill is **git read-only**.

Allowed git commands:
- `git status`
- `git diff`
- `git diff --staged`
- `git log`
- `git show`
- `git rev-parse`
- `git branch`

Never run state-changing commands (`git add`, `git commit`, `git push`, `git reset`, `git stash`, etc.).

## Workflow

### 1) Detect Context

- Read `CONTRIBUTING.md`, `CLAUDE.md`, `AGENTS.md`, `README.md` if present.
- Identify workspace conventions (commit style, docs sync rules, DRY rules).

### 2) Inspect Changes

Run:

```bash
git status
git diff
git diff --staged
git log --oneline -10
```

Early exit if no changes.

### 3) Review Coherence

For changed files, check:
- obvious breakages/syntax problems
- debug leftovers
- naming and consistency
- documentation alignment (`README`, `CLAUDE.md`, `AGENTS.md`)
- `.claude` / `.opencode` DRY consistency when applicable

### 4) Decide WIP Prefix

Prefix commit with `(WIP)` if task is incomplete or important issues remain.

### 5) Generate Commit Message

Format:

```text
<(WIP) if needed> <type>(optional-scope): <short description>

<optional body with why>
```

Suggested types: `add`, `fix`, `update`, `refactor`, `remove`, `docs`, `test`, `style`, `chore`.

### 6) Return Structured Report

```markdown
## Changes Summary
...

## Issues Found
...

## Documentation Alignment
- README.md: ...
- CLAUDE.md: ...
- AGENTS.md: ...

## Suggested Commit Message
...
```

Do not execute the commit.
