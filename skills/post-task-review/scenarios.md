# Test Scenarios: post-task-review

## Scenario 1: Simple Code Change with Documentation

**Setup:**
- User modified a skill's SKILL.md
- User updated README.md to reflect the change
- No issues in the code
- Task is complete

**Expected behavior:**
- Skill identifies the modified files
- Reviews changes and finds no issues
- Confirms documentation is aligned
- Generates clean commit message without (WIP) prefix
- Does NOT execute git commit

**Example output:**
```
## Changes Summary
Updated post-task-review/SKILL.md to add workspace-specific checks section

## Issues Found
None detected

## Documentation Alignment
- README.md: Updated, aligned
- CLAUDE.md: Not applicable
- AGENTS.md: Not applicable

## Suggested Commit Message

update: Add workspace-specific checks to post-task-review skill

Enhanced the skill with guidance for reviewing prompt engineering
workspaces, frontend, backend, and documentation projects.
```

---

## Scenario 2: Incomplete Task (WIP)

**Setup:**
- User created a new skill directory
- SKILL.md has TODO placeholders
- User stated task is not finished
- Documentation not yet updated

**Expected behavior:**
- Skill identifies new files
- Detects incomplete state (TODOs, user statement)
- Adds (WIP) prefix to commit message
- Notes what remains to be done

**Example output:**
```
## Changes Summary
Created new image-processor skill with basic structure

## Issues Found
- SKILL.md contains TODO placeholders
- Test scenarios not yet created
- README.md not updated with new skill

## Documentation Alignment
- README.md: Needs update (new skill not listed)
- CLAUDE.md: Not applicable
- AGENTS.md: Not applicable

## Suggested Commit Message

(WIP) add: Create image-processor skill skeleton

Initialized skill directory with SKILL.md template.
TODO: Implement image processing workflows, create test scenarios,
update README.
```

---

## Scenario 3: Debugging Artifacts Left Behind

**Setup:**
- User modified Python code
- Code contains console.log or print statements
- Code contains commented-out debug code
- Task otherwise complete

**Expected behavior:**
- Skill identifies debugging artifacts
- Flags them as issues
- Adds (WIP) prefix or asks user to fix first

**Example output:**
```
## Changes Summary
Added user authentication flow to auth.py

## Issues Found
- Line 42: print(f"Debug: {user_data}") - debugging statement
- Line 55: # TODO: fix this hack - incomplete work
- Line 78: Commented-out debug code block

## Documentation Alignment
- README.md: No auth documentation yet
- CLAUDE.md: Not applicable
- AGENTS.md: Not applicable

## Suggested Commit Message

(WIP) add: Implement user authentication flow

Added login, logout, and session management.
ISSUES: Remove debug prints, address TODO, document auth flow.
```

---

## Scenario 4: Workspace with Commit Conventions

**Setup:**
- Workspace has CONTRIBUTING.md with specific commit format
- Format: `[<type>] <description>` with types: feat, fix, docs, refactor
- User followed these conventions

**Expected behavior:**
- Skill detects CONTRIBUTING.md
- Reads and applies the commit format
- Generates message matching workspace conventions

**Example output:**
```
## Changes Summary
Fixed bug in user session timeout handling

## Issues Found
None detected

## Documentation Alignment
- README.md: No changes needed
- CLAUDE.md: Not applicable
- AGENTS.md: Not applicable

## Suggested Commit Message

[fix] Correct session timeout calculation

Previous logic used milliseconds instead of seconds,
causing immediate session expiration. Updated to use
proper time units.
```

---

## Scenario 5: Prompt Engineering Workspace

**Setup:**
- User modified an agent prompt in `.claude/agents/prompt-architect.md`
- User did NOT update `.opencode/agents/prompt-architect.md` (same content required)
- CLAUDE.md exists and documents DRY rule

**Expected behavior:**
- Skill detects prompt engineering workspace
- Notices agent prompt exists in both locations
- Checks if both files have identical content
- Flags inconsistency as issue

**Example output:**
```
## Changes Summary
Updated prompt-architect agent with improved decision protocol

## Issues Found
- .claude/agents/prompt-architect.md was modified
- .opencode/agents/prompt-architect.md NOT updated
- DRY rule violation: agent prompts must be identical in both locations

## Documentation Alignment
- README.md: No changes needed
- CLAUDE.md: DRY rule documented (agent acknowledges constraint)
- AGENTS.md: Not applicable

## Suggested Commit Message

(WIP) update: Improve decision protocol in prompt-architect

Added clarification on when to use Decision Protocol.
ISSUE: Must also update .opencode/agents/prompt-architect.md
to maintain identical content (DRY rule).
```

---

## Scenario 6: Documentation Out of Sync

**Setup:**
- User added new skill
- README.md not updated to list the skill
- CLAUDE.md should reference the skill but doesn't

**Expected behavior:**
- Skill identifies new skill was created
- Checks if documentation references it
- Asks user if documentation needs updating
- Generates appropriate message

**Example output:**
```
## Changes Summary
Created new commit-message-generator skill

## Issues Found
None detected in code

## Documentation Alignment
- README.md: Does not list new skill
- CLAUDE.md: Does not reference /commit-message-generator
- AGENTS.md: Not applicable

The documentation should be updated to include this new skill.
Should I note this in the commit message?

## Suggested Commit Message

add: Create commit-message-generator skill

New skill for generating conventional commit messages.
TODO: Update README.md and CLAUDE.md to document.
```

---

## Scenario 7: Large Refactor

**Setup:**
- User moved multiple files to new directory structure
- Updated imports across many files
- All tests passing
- Documentation updated

**Expected behavior:**
- Skill handles many changed files
- Summarizes the refactor
- Verifies all imports updated correctly
- Clean commit message

**Example output:**
```
## Changes Summary
Refactored: moved agent-specific skills into dedicated directories

## Issues Found
None detected

## Documentation Alignment
- README.md: Updated with new structure
- CLAUDE.md: Verified, still accurate
- AGENTS.md: Verified, still accurate

## Suggested Commit Message

refactor: Reorganize skill directory structure

Moved agent-specific skills from /skills/ to /skills/agents/
and utility skills to /skills/utils/. Updated all import
statements and documentation to reflect new structure.
```

---

## Scenario 8: Unknown Workspace Type

**Setup:**
- New project without clear workspace guidelines
- No CONTRIBUTING.md, CLAUDE.md, or AGENTS.md
- User makes changes

**Expected behavior:**
- Skill detects no clear workspace type
- Asks user about conventions
- Uses default format with user confirmation

**Example interaction:**
```
Skill: "I couldn't find specific commit message conventions in this workspace.
Should I use standard format (type: description) or does this project have
a specific format I should follow?"

User: "Use conventional commits with format: type(scope): description"

Skill: [Proceeds with specified format]
```

---

## Scenario 9: Empty Git Status

**Setup:**
- User invokes the skill but no files have changed
- Working directory is clean
- No staged or unstaged changes

**Expected behavior:**
- Skill detects `git status` shows no changes
- Informs user immediately that there's nothing to review
- Exits gracefully without attempting to generate commit message

**Example output:**
```
## Changes Summary

No changes detected. The working directory is clean.

## Suggested Commit Message

None - there are no changes to commit.

Run git status to verify the repository state.
```

**Edge case variants:**
- Untracked files present but no actual changes (should note untracked files and ask if they should be added)
- Only `.gitignore` changes (should handle appropriately)
