---
name: post-task-review
description: Post-task review workflow that generates git commit messages. Reviews changed files for coherence, verifies documentation alignment (README, CLAUDE.md, AGENTS.md), detects workspace conventions, and generates appropriate commit messages. READ-ONLY: uses git for inspection only, NEVER executes git commit/add/push. Adds (WIP) prefix if issues found.
license: Complete terms in LICENSE.txt
argument-hint: none
---

# Post-Task Review

Review changes after task completion and generate an appropriate git commit message.

## When to Use

Invoke this skill after completing any task that made file changes. The skill will:
- Identify what was changed
- Review changes for coherence and potential issues
- Check if documentation needs updating
- Generate a commit message (following workspace conventions if found)

## Core Constraints

**CRITICAL: This skill is READ-ONLY regarding git.**

Allowed git commands (informational only):
- `git status`
- `git diff`
- `git diff --staged`
- `git log`
- `git config` (for reading user context only)
- `git rev-parse`
- `git branch`
- `git show`

**NEVER execute:**
- `git commit`
- `git add`
- `git push`
- `git stash`
- `git reset`
- Any git command that modifies repository state

**File reading is expected and necessary:** Use the Read tool to examine changed file contents for coherence review. This is not a git operation and is essential for the review workflow.

## Workflow

### Step 1: Detect Workspace Context

First, understand the workspace structure and conventions:

1. **Check for workspace guidelines** in this order:
   - `CONTRIBUTING.md`
   - `CLAUDE.md`
   - `AGENTS.md`
   - `README.md`
   - Look for sections about: commit message format, code style, documentation requirements

2. **Determine if this is a specialized workspace**:
   - Prompt engineering workspace (has `.claude/agents/` or similar)
   - Frontend/backend project
   - Documentation project
   - Multi-language project

3. **Ask the user if uncertain** about guidelines:
   - "I found [X] guidelines. Should I follow them?"
   - "I couldn't find commit message conventions. What format should I use?"

### Step 2: Identify Changed Files

Run git commands to identify what was changed:

```bash
# Get overview of changes
git status

# Get detailed diff of unstaged changes
git diff

# Get detailed diff of staged changes (if any)
git diff --staged

# Get recent commit history (to understand context)
git log --oneline -10
```

**Edge case handling:**
- **Early exit:** If `git status` shows no changes, inform the user and exit.
- **Merge conflict detection:** Look for `CONFLICT` markers in git status output. If present, flag this prominently in issues.
- **Large diffs:** If the diff output is very large (>1000 lines), select representative files to review rather than attempting to read everything.

Parse the output to identify:
- Modified files
- New files
- Deleted files
- Renamed/moved files

### Step 3: Review Changes for Coherence

**Priority:** Check for syntax errors and broken imports first (critical issues), then assess coherence and consistency (secondary concerns).

For each changed file, read the relevant changes and assess:

**Code files** (`.py`, `.js`, `.ts`, `.md`, etc.):
- Do the changes make logical sense?
- Are there obvious bugs or syntax issues?
- Are there debugging artifacts left behind? (check workspace guidelines for what to look for)
- Is file naming consistent with the project?

**Documentation files**:
- `README.md` - Is it still accurate? Were new features/changes documented?
- `CLAUDE.md` / `AGENTS.md` - If they exist, are they still consistent with code?
- **Prompt engineering workspaces:** If `.claude/` and `.opencode/` both exist, check that agent/skill files are synchronized (DRY rule)
- Inline code comments - Are they accurate and helpful?

**Configuration files**:
- `package.json`, `pyproject.toml`, etc. - Are dependencies updated if needed?
- Config files - Are changes consistent?

### Step 4: Check Documentation Alignment

Verify that documentation reflects the changes:

1. **If code was added/modified**:
   - Does `README.md` need updating?
   - Do `CLAUDE.md` / `AGENTS.md` (if they exist) need updates?
   - Are there new files that should be documented?

2. **If documentation was modified**:
   - Is it consistent with the actual code?
   - Are all references accurate?

3. **Ask the user** if documentation seems out of sync:
   - "I noticed [X] changed but [Y documentation] wasn't updated. Should it be?"

### Step 5: Determine WIP Status

Add `(WIP)` prefix to the commit message if ANY of the following:

**Task incomplete:**
- User explicitly stated the task is not finished
- Obvious next steps remain
- Tests are failing (if applicable)

**Issues found:**
- Syntax errors in changed code
- Debugging artifacts present (based on workspace guidelines)
- Broken references or imports
- Inconsistencies between code and documentation

**Documentation missing:**
- Significant changes made but documentation not updated
- User confirms documentation needs updating but hasn't done so yet

### Step 6: Generate Commit Message

Follow this structure, adapting to workspace conventions if found:

```
<WIP prefix if applicable> <Type><Optional Scope>: <Description>

<Optional detailed body explaining what changed and why>
```

**If workspace has specific conventions, follow them.**

Otherwise, use this format:

**Type** (choose one):
- `add` - New feature, new file, new capability
- `fix` - Bug fix, error correction
- `update` - Enhancement to existing feature
- `refactor` - Code restructuring without behavior change
- `remove` - Remove feature, file, or code
- `docs` - Documentation changes only
- `test` - Test additions or modifications
- `style` - Code style changes (formatting, naming)
- `chore` - Maintenance, dependencies, configuration

**Scope** (optional): Brief context like `auth`, `docs`, `skill-name`

**Description**: Concise summary (50 chars or less) in present tense, imperative mood

**Examples:**

```
Add skill for post-task review workflow

Create /post-task-review skill that reviews changed files, checks
documentation alignment, and generates commit messages following
workspace conventions.
```

```
fix(docs): Correct agent symlink instructions in README

The previous instructions described the wrong symlink direction.
Updated to reflect actual workspace structure.
```

```
(WIP) refactor: Extract validation logic into shared module

Extracted validation code from auth and payment modules into
common/utils/validation.py. TODO: Add unit tests.
```

### Step 7: Present Findings and Message

Present the results to the user in this order:

1. **Summary of changes**: Brief overview of what was changed
2. **Issues found** (if any): List coherence problems, debugging artifacts, documentation gaps
3. **Documentation check**: Status of README, CLAUDE.md, AGENTS.md alignment
4. **Commit message**: The generated message

Format:

```
## Changes Summary
[Brief description of what was changed]

## Issues Found
- [List any issues, or "None detected"]

## Documentation Alignment
- README.md: [Status]
- CLAUDE.md: [Exists/Not applicable]
- AGENTS.md: [Exists/Not applicable]

## Suggested Commit Message

[Commit message here]
```

**Important**: Do NOT execute the commit. Present the message for the user to use.

## Workspace-Specific Guidelines

When reviewing, look for and apply workspace-specific patterns:

**Prompt engineering workspaces** (`.claude/` or agents structure):
- **DRY rule check:** If agent prompts exist in both `.claude/` and `.opencode/`, verify content is identical
- Verify SKILL.md frontmatter is complete (name, description required)
- Check for test scenarios in skill specs (`scenarios.md`)

**Frontend projects**:
- Check for console.log, debugger statements
- Verify component props are documented
- Check if package.json versions updated

**Backend projects**:
- Check for print statements, debug flags
- Verify API documentation
- Check migration files if database changed

**Documentation projects**:
- Verify all links work
- Check heading hierarchy
- Verify code examples are current

If unsure what workspace-specific rules to apply, ask the user:
"What workspace-specific checks should I perform? I detected this is a [X] type of project."

## Important Notes

- This skill generates commit messages; it does NOT create commits
- Always ask the user if workspace guidelines are unclear
- The (WIP) prefix signals "work in progress" and is appropriate for incomplete tasks
- When in doubt, be verbose in the commit body rather than vague in the subject
