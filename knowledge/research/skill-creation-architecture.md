# Skill Creation Architecture — Research & Plan

> Last updated: 2026-02-14
> Status: Current
> **NOTE (2026-02-15)**: File structure references in this document use the old `prompts/agents/` and `prompts/skills/` convention. The current workspace structure uses `agents/` and `skills/` directly (no `prompts/` prefix). See `AGENTS.md` for the current structure.

## Summary

This document contains deep research findings on the anatomy of skills for Claude Code and OpenCode, a gap analysis of the current PromptArchitect pipeline, and a detailed architecture plan for adding skill creation capability to the existing agents without modifying any existing files.

## Sources

- [Extend Claude with skills — Claude Code Docs](https://code.claude.com/docs/en/skills)
- [Agent Skills Open Standard — Specification](https://agentskills.io/specification)
- [Agent Skills GitHub — anthropics/skills](https://github.com/anthropics/skills/blob/main/spec/agent-skills-spec.md)
- [OpenCode Skills Documentation](https://opencode.ai/docs/skills)
- [OpenCode Commands Documentation](https://opencode.ai/docs/commands/)
- [OpenCode Agents Documentation](https://opencode.ai/docs/agents/)
- [Claude Code Customization Guide](https://alexop.dev/posts/claude-code-customization-guide-claudemd-skills-subagents/)
- [Claude Code Merges Slash Commands Into Skills](https://medium.com/@joe.njenga/claude-code-merges-slash-commands-into-skills-dont-miss-your-update-8296f3989697)

---

## Part 1: Research Findings

### 1.1 Agent Skills Open Standard (agentskills.io)

Both Claude Code and OpenCode implement the **Agent Skills** open standard. This is the baseline format.

**Directory structure:**
```
skill-name/
├── SKILL.md           # Required — main instructions
├── scripts/           # Optional — executable code
├── references/        # Optional — additional documentation
└── assets/            # Optional — templates, images, data files
```

**SKILL.md format — Frontmatter (standard fields):**

| Field | Required | Constraints |
|---|---|---|
| `name` | Yes | Max 64 chars. Lowercase letters, numbers, hyphens only. No leading/trailing/consecutive hyphens. Must match directory name. Pattern: `^[a-z0-9]+(-[a-z0-9]+)*$` |
| `description` | Yes | 1-1024 chars. Should describe what the skill does AND when to use it. Include keywords that help agents identify relevant tasks. |
| `license` | No | License name or reference to bundled license file. |
| `compatibility` | No | Max 500 chars. Environment requirements (product, system packages, network). |
| `metadata` | No | Arbitrary string-to-string key-value mapping. |
| `allowed-tools` | No | Space-delimited list of pre-approved tools. Experimental. |

**SKILL.md format — Body:**
- Markdown content after frontmatter
- No format restrictions
- Recommended sections: step-by-step instructions, examples of inputs/outputs, common edge cases
- Keep under 500 lines; move detailed content to referenced files

**Progressive disclosure model:**
1. **Metadata** (~100 tokens): `name` and `description` loaded at startup for all skills
2. **Instructions** (< 5000 tokens recommended): Full `SKILL.md` body loaded when skill is activated
3. **Resources** (as needed): Files in `scripts/`, `references/`, `assets/` loaded only when required

### 1.2 Claude Code Skills — Platform Extensions

Claude Code extends the standard with additional features.

**Storage locations (priority order):**

| Level | Path | Scope |
|---|---|---|
| Enterprise | Managed settings | All users in org |
| Personal | `~/.claude/skills/<skill-name>/SKILL.md` | All your projects |
| Project | `.claude/skills/<skill-name>/SKILL.md` | This project only |
| Plugin | `<plugin>/skills/<skill-name>/SKILL.md` | Where plugin is enabled |

Higher-priority locations win when names conflict. Plugin skills use `plugin-name:skill-name` namespace.

**Claude Code extended frontmatter fields:**

| Field | Default | Description |
|---|---|---|
| `name` | directory name | Display name and slash-command name |
| `description` | first paragraph | What the skill does and when to use it |
| `argument-hint` | none | Hint shown during autocomplete (e.g., `[issue-number]`) |
| `disable-model-invocation` | `false` | If `true`, only user can invoke via `/name` |
| `user-invocable` | `true` | If `false`, hidden from `/` menu; only Claude can invoke |
| `allowed-tools` | none | Tools Claude can use without permission when skill is active |
| `model` | none | Model to use when skill is active |
| `context` | none | Set to `fork` to run in forked subagent context |
| `agent` | `general-purpose` | Which subagent type for `context: fork` |
| `hooks` | none | Lifecycle hooks scoped to skill |

**Invocation control matrix:**

| Frontmatter | User can invoke | Claude can invoke | Context loading |
|---|---|---|---|
| (default) | Yes | Yes | Description always in context; full skill on invoke |
| `disable-model-invocation: true` | Yes | No | Description NOT in context; loads on user invoke |
| `user-invocable: false` | No | Yes | Description always in context; loads on Claude invoke |

**String substitutions in skill content:**
- `$ARGUMENTS` — All arguments passed
- `$ARGUMENTS[N]` or `$N` — Specific argument by 0-based index
- `${CLAUDE_SESSION_ID}` — Current session ID

**Dynamic context injection:**
- `!`command`` syntax runs shell commands before skill content is sent to Claude
- Output replaces the placeholder; Claude receives fully-rendered content

**Skill content types:**
- **Reference content**: Knowledge Claude applies to current work (conventions, patterns, style guides). Runs inline.
- **Task content**: Step-by-step instructions for specific actions. Often `disable-model-invocation: true`.

**Context budget:** Skill descriptions loaded into context budget of 2% of context window (fallback: 16,000 chars). Override via `SLASH_COMMAND_TOOL_CHAR_BUDGET` env var.

**Backward compatibility:** `.claude/commands/` files still work with the same frontmatter support. Skills take precedence when names conflict.

### 1.3 OpenCode Skills — Platform Extensions

OpenCode also implements the Agent Skills standard with its own extensions.

**Storage locations (searched by walking up from cwd to git worktree root):**

| Path | Scope |
|---|---|
| `.opencode/skills/<name>/SKILL.md` | Project-local |
| `~/.config/opencode/skills/<name>/SKILL.md` | Global |
| `.claude/skills/<name>/SKILL.md` | Cross-compatible (project) |
| `~/.claude/skills/<name>/SKILL.md` | Cross-compatible (global) |
| `.agents/skills/<name>/SKILL.md` | Generic (project) |
| `~/.agents/skills/<name>/SKILL.md` | Generic (global) |

**OpenCode frontmatter fields (standard only):**
- `name` (required)
- `description` (required)
- `license` (optional)
- `compatibility` (optional)
- `metadata` (optional)

OpenCode does NOT extend the frontmatter with `disable-model-invocation`, `context`, `agent`, `hooks`, `argument-hint`, `user-invocable`, or `model` fields.

**Invocation mechanism:** Agents access skills through the native `skill` tool by calling `skill({ name: "skill-name" })`. Available skills appear in the tool description with name and description pairs.

**Permission control (in `opencode.json`):**
```json
{
  "permission": {
    "skill": {
      "*": "allow",
      "pattern-*": "deny"
    }
  }
}
```

Behaviors: `allow` (immediate), `deny` (hidden), `ask` (user approval required).

**Per-agent skill permissions (in agent frontmatter):**
```yaml
permission:
  skill:
    "documents-*": "allow"
```

### 1.4 OpenCode Custom Commands

OpenCode has a separate "commands" concept distinct from skills.

**Storage locations:**
- Global: `~/.config/opencode/commands/`
- Per-project: `.opencode/commands/`

**File format:** Markdown file with YAML frontmatter. Filename (minus `.md`) becomes the command name.

**Frontmatter fields:**

| Field | Required | Description |
|---|---|---|
| `description` | No | Brief description shown in TUI |
| `agent` | No | Which agent executes the command |
| `model` | No | Override model for this command |
| `subtask` | No | Forces subagent invocation, isolating context |

**Template features:**
- `$ARGUMENTS`, `$1`, `$2`, `$3` — argument substitution
- `!`command`` — shell output injection
- `@filename` — file content inclusion

### 1.5 Key Differences: System Prompt/Agent vs. Skill

| Dimension | System Prompt / Agent | Skill |
|---|---|---|
| **Purpose** | Defines the agent's core identity, reasoning strategy, and behavioral constraints | Extends an agent's capabilities for a specific task or domain |
| **Persistence** | Always loaded into context | Loaded on demand (description always; full content on invoke) |
| **Scope** | Governs all of the agent's behavior | Governs behavior only when the skill is active |
| **Size** | Can be large (thousands of tokens) | Should be small (< 500 lines SKILL.md; < 5000 tokens recommended) |
| **Invocation** | Selected at session start; user switches between agents | Invoked mid-conversation by user (`/skill-name`) or by agent automatically |
| **Identity** | Defines WHO the agent is | Defines WHAT the agent can do in a specific situation |
| **File format** | Varies by platform (Claude: `.claude/agents/`, OpenCode: `.opencode/agents/`) | Standard: `<location>/skills/<name>/SKILL.md` |
| **Standard** | Platform-specific | Agent Skills open standard (cross-platform) |
| **Composability** | One agent active at a time | Multiple skills can activate per session |
| **Output** | The agent's responses | Task-specific outputs within the agent's session |

### 1.6 What Makes a Good Skill vs. What Makes a Good Agent

**Good candidates for skills:**
- Repeatable workflows (deploy, release, review PR)
- Domain knowledge injection (API conventions, coding standards)
- Template-driven tasks (create component, scaffold module)
- Research/exploration tasks (deep-dive into codebase)
- Tool-specific procedures (git workflows, CI/CD)

**Good candidates for agents:**
- Distinct personas with different cognitive biases
- Fundamentally different reasoning strategies
- Different tool access requirements
- Different autonomy levels
- Long-running, identity-dependent workflows

---

## Part 2: Gap Analysis — Current Pipeline vs. Skill Creation

### 2.1 What the Current Pipeline Supports

The current D.A.R.T.E. pipeline produces **system prompts for LLM agents**. The output is:
- Architecture spec (Planner)
- System prompt in 10-component structure (Builder)
- Test scenarios + evaluation (Reviewer)
- Agent files for `.claude/agents/` and `.opencode/agents/`

The pipeline is optimized for creating agents with:
- Identity with cognitive bias
- Complex reasoning strategies
- Full tool design
- Multi-component output format
- Safety layers

### 2.2 What Is Missing for Skill Creation

**The Planner needs:**
1. A different discovery questionnaire — skills need different requirements than agents
2. Knowledge of the Agent Skills standard (frontmatter fields, naming constraints, directory structure)
3. A different architecture spec template — skills do not have identity, cognitive bias, or reasoning strategy
4. Understanding of skill-specific design decisions: invocation control, context strategy (fork vs. inline), tool restrictions, argument design

**The Builder needs:**
1. A different output format — SKILL.md instead of 10-component system prompt
2. Knowledge of YAML frontmatter schema for both Claude Code and OpenCode
3. Understanding of progressive disclosure (what goes in SKILL.md vs. references/)
4. String substitution syntax ($ARGUMENTS, $N, !`command`)
5. A different "writing principles" set for skills (concise, task-focused, no identity)

**The Reviewer needs:**
1. A different evaluation rubric — skills have different quality dimensions than agents
2. Different test scenarios — skill invocation, argument handling, context budget, cross-platform compatibility
3. Understanding of what makes a skill good vs. bad (specificity of description, progressive disclosure, naming)

**The pipeline needs:**
1. A meta-task template for skill creation (parallel to `meta-task-create-agent.md`)
2. Reference documentation on skill anatomy for all agents to consume
3. A file structure convention for skill deliverables

### 2.3 What Does NOT Need to Change

- The D.A.R.T.E. framework itself (5 phases still apply)
- The 3-agent pipeline structure (Planner -> Builder -> Reviewer)
- The knowledge management system
- The existing agent files (constraint: additive-only)
- The existing AGENTS.md and CLAUDE.md

---

## Part 3: Architecture Plan — New Files

### 3.1 File Plan Overview

All new files are additive. No existing files are modified.

```
NEW FILES TO CREATE:
├── knowledge/
│   └── research/
│       └── skill-creation-architecture.md      # THIS FILE (research + plan)
├── prompts/
│   └── templates/
│       └── meta-task-create-skill.md           # Pipeline orchestration template for skills
├── .claude/
│   └── skills/
│       └── prompt-architect-skill-planner/
│           └── SKILL.md                         # Skill-creation overlay for planner agent
│       └── prompt-architect-skill-builder/
│           └── SKILL.md                         # Skill-creation overlay for builder agent
│       └── prompt-architect-skill-reviewer/
│           └── SKILL.md                         # Skill-creation overlay for reviewer agent
│       └── skill-anatomy-reference/
│           ├── SKILL.md                         # Reference skill for anatomy knowledge
│           └── references/
│               └── platform-comparison.md       # Claude Code vs OpenCode differences
└── .opencode/
    └── skills/
        └── prompt-architect-skill-planner/
            └── SKILL.md                         # Skill-creation overlay for planner agent
        └── prompt-architect-skill-builder/
            └── SKILL.md                         # Skill-creation overlay for builder agent
        └── prompt-architect-skill-reviewer/
            └── SKILL.md                         # Skill-creation overlay for reviewer agent
        └── skill-anatomy-reference/
            ├── SKILL.md                         # Reference skill for anatomy knowledge
            └── references/
                └── platform-comparison.md       # Claude Code vs OpenCode differences
```

**Total new files: 14** (including this research document, which already exists at this path)

### 3.2 Design Rationale: Skills as Overlays

The key design decision: rather than modifying the existing agent files, we create **skills that overlay skill-creation knowledge** onto the existing agents. When a user wants to create a skill instead of an agent, they invoke the appropriate skill overlay, which loads skill-specific instructions into the agent's context.

This approach:
- Preserves all existing files unchanged
- Follows the skill system's own design philosophy (composable, on-demand context)
- Keeps the agent files focused on their core competency (agent creation)
- Adds skill-creation as a capability that loads only when needed
- Works natively in both Claude Code and OpenCode

**How it works in practice:**
1. User invokes `/prompt-architect-skill-planner` (or the agent auto-loads it when the user mentions creating a skill)
2. The existing `prompt-architect-planner` agent receives skill-specific discovery questions and architecture spec template as additional context
3. The Planner produces a skill architecture spec instead of an agent architecture spec
4. Same flow continues with Builder and Reviewer skill overlays

### 3.3 Detailed File Descriptions

#### File 1: `knowledge/research/skill-creation-architecture.md`
**Purpose:** This file. Research findings and architecture plan. Used by all agents as reference.
**Status:** Already created (this document).

#### File 2: `prompts/templates/meta-task-create-skill.md`
**Purpose:** Pipeline orchestration template for creating a new skill. Parallel to `meta-task-create-agent.md`.
**Content outline:**
- Prerequisites (what you need before starting)
- Step 1: Plan — use planner agent WITH skill overlay, produce skill architecture spec
- Step 2: Build — use builder agent WITH skill overlay, produce SKILL.md and supporting files
- Step 3: Review — use reviewer agent WITH skill overlay, evaluate and test
- File save locations (different from agent deliverables)
- Final checklist for skill completion
- Notes on cross-platform compatibility (Claude Code vs OpenCode)

#### File 3: `.claude/skills/prompt-architect-skill-planner/SKILL.md`
**Purpose:** Overlays skill-creation discovery and architecture process onto the Planner agent.
**Frontmatter:**
- `name: prompt-architect-skill-planner`
- `description`: Activates skill-creation mode for the Planner. Use when designing a new skill (not an agent). Modifies the discovery questionnaire and architecture spec template to focus on skill anatomy.
- `disable-model-invocation: true` (user should explicitly invoke this)
**Content outline:**
- Skill-specific discovery questionnaire (replaces agent questionnaire):
  1. Skill purpose: What task or knowledge does this skill provide?
  2. Invocation mode: User-only, agent-only, or both?
  3. Target platforms: Claude Code, OpenCode, or both?
  4. Arguments: Does the skill accept arguments? What format?
  5. Context strategy: Inline or forked subagent?
  6. Tool restrictions: Should the skill limit available tools?
  7. Supporting files: Does the skill need scripts, references, or assets?
  8. Dynamic context: Does the skill need shell command output injection?
  9. Scope: Project-only, personal, or distributable?
  10. Naming: What name fits the `^[a-z0-9]+(-[a-z0-9]+)*$` pattern?
- Skill architecture spec template (replaces agent architecture spec):
  - Requirements summary (purpose, platforms, invocation mode, scope)
  - Naming validation
  - Frontmatter design (platform-specific fields)
  - Content strategy (what goes in SKILL.md vs. references/)
  - Argument design
  - Progressive disclosure plan
  - Cross-platform compatibility notes

#### File 4: `.claude/skills/prompt-architect-skill-builder/SKILL.md`
**Purpose:** Overlays skill-writing process onto the Builder agent.
**Frontmatter:**
- `name: prompt-architect-skill-builder`
- `description`: Activates skill-creation mode for the Builder. Use when writing a SKILL.md file from a skill architecture spec.
- `disable-model-invocation: true`
**Content outline:**
- Skill construction process (replaces 10-component agent structure):
  1. Validate architecture spec contains skill-specific requirements
  2. Compose YAML frontmatter (platform-appropriate fields)
  3. Write skill body content (instructions, not identity)
  4. Design supporting files if needed (references/, scripts/, assets/)
  5. Validate naming constraints
  6. Check progressive disclosure (SKILL.md under 500 lines, body under 5000 tokens)
  7. Add string substitutions where appropriate ($ARGUMENTS, !`command`)
- Skill writing principles (different from agent writing):
  - Task-focused, not identity-focused
  - Concise instructions, not behavioral frameworks
  - Show expected inputs/outputs
  - Include edge case handling
  - Respect context budget constraints
- Output format: complete SKILL.md file(s) plus supporting files
- Platform-specific notes (what to include for Claude Code vs. OpenCode)

#### File 5: `.claude/skills/prompt-architect-skill-reviewer/SKILL.md`
**Purpose:** Overlays skill-review process onto the Reviewer agent.
**Frontmatter:**
- `name: prompt-architect-skill-reviewer`
- `description`: Activates skill-review mode for the Reviewer. Use when reviewing a SKILL.md for quality, testing invocation patterns, and evaluating cross-platform compatibility.
- `disable-model-invocation: true`
**Content outline:**
- Skill-specific test scenarios (replaces agent test scenarios):
  1. Happy path: Typical invocation with expected arguments
  2. Edge case: Missing arguments, unusual input, boundary conditions
  3. Cross-platform: Does the skill work on both Claude Code and OpenCode?
  4. Context budget: Is the description efficient? Is the body under recommended limits?
- Skill-specific evaluation rubric (replaces 7-dimension agent rubric):
  | Dimension | Question |
  |---|---|
  | Description quality | Does the description help agents decide when to load this skill? |
  | Naming | Does the name follow the pattern and clearly indicate purpose? |
  | Conciseness | Is the SKILL.md under 500 lines / 5000 tokens? |
  | Progressive disclosure | Is content split appropriately between SKILL.md and references? |
  | Argument handling | Are arguments well-documented and edge cases handled? |
  | Cross-platform | Does it work on all target platforms without platform-specific breakage? |
  | Invocation control | Is the invocation mode (user/agent/both) appropriate for the skill's purpose? |
- Decision: APPROVE / APPROVE WITH NOTES / REJECT (same as agent reviews)

#### File 6: `.claude/skills/skill-anatomy-reference/SKILL.md`
**Purpose:** Reference skill that provides on-demand knowledge about skill anatomy. Agents can load this when they need platform-specific details.
**Frontmatter:**
- `name: skill-anatomy-reference`
- `description`: Reference documentation on the Agent Skills standard and platform-specific extensions. Use when creating, reviewing, or modifying skills to ensure compliance with the standard.
- `user-invocable: false` (only agents should load this)
**Content outline:**
- Quick reference: YAML frontmatter schema
- Quick reference: naming constraints
- Quick reference: directory structure
- Quick reference: progressive disclosure model
- Link to `references/platform-comparison.md` for detailed platform differences

#### File 7: `.claude/skills/skill-anatomy-reference/references/platform-comparison.md`
**Purpose:** Detailed comparison of Claude Code vs. OpenCode skill support.
**Content:** The platform comparison table from Part 1 of this document, formatted as a quick-reference for agents.

#### Files 8-13: `.opencode/skills/` mirrors
**Purpose:** Identical copies of files 3-7 for OpenCode compatibility. OpenCode searches `.opencode/skills/` natively. Since the Agent Skills standard is shared, the SKILL.md content is identical. The only differences are in frontmatter: OpenCode does not support Claude Code extensions like `disable-model-invocation`, `context`, `agent`, `hooks`, `user-invocable`, or `model`. The OpenCode versions use only standard frontmatter fields.

**Important note:** Because OpenCode also searches `.claude/skills/`, the OpenCode copies could theoretically be omitted. However, for clarity and to follow the convention established by the existing workspace (which maintains separate `.claude/agents/` and `.opencode/agents/` files), we should create explicit OpenCode copies with OpenCode-appropriate frontmatter.

### 3.4 Pipeline Flow for Skill Creation

```
User has an idea for a skill
        │
        v
Step 1: PLAN (prompt-architect-planner + /prompt-architect-skill-planner)
        │  Discovery: skill-specific questionnaire (10 questions)
        │  Architecture: skill architecture spec (not agent architecture spec)
        │  Output: skill-architecture-spec.md
        │
        v
Step 2: BUILD (prompt-architect-builder + /prompt-architect-skill-builder)
        │  Input: skill architecture spec
        │  Construction: SKILL.md + supporting files
        │  Output: complete skill directory
        │
        v
Step 3: REVIEW (prompt-architect-reviewer + /prompt-architect-skill-reviewer)
        │  Test: 4 skill-specific test scenarios
        │  Evaluate: 7 skill-specific dimensions
        │  Output: test-scenarios.md + evaluation.md
        │  Decision: APPROVE / APPROVE WITH NOTES / REJECT
        │
        v
Skill deliverables saved to:
  prompts/skills/[skill-name]/skill-architecture-spec.md
  prompts/skills/[skill-name]/test-scenarios.md
  prompts/skills/[skill-name]/evaluation.md
  .claude/skills/[skill-name]/SKILL.md (+ supporting files)
  .opencode/skills/[skill-name]/SKILL.md (+ supporting files)
```

### 3.5 Deliverable File Structure for Created Skills

When the pipeline creates a new skill, the output goes to:

```
prompts/
  skills/                              # NEW directory (parallel to prompts/agents/)
    [skill-name]/
      skill-architecture-spec.md       # From Planner
      test-scenarios.md                # From Reviewer
      evaluation.md                    # From Reviewer
.claude/
  skills/
    [skill-name]/
      SKILL.md                         # From Builder (production artifact)
      references/                      # Optional, from Builder
      scripts/                         # Optional, from Builder
      assets/                          # Optional, from Builder
.opencode/
  skills/
    [skill-name]/
      SKILL.md                         # From Builder (production artifact)
      references/                      # Optional, from Builder
      scripts/                         # Optional, from Builder
      assets/                          # Optional, from Builder
```

---

## Part 4: Open Questions and Decisions

### 4.1 Resolved Decisions

1. **Approach: skill overlays vs. new agents.** Decision: skill overlays. Rationale: no file modifications, composable, follows the skill system's own philosophy, minimal token overhead (skills only load when needed).

2. **OpenCode copies: separate or shared?** Decision: separate files with OpenCode-appropriate frontmatter. Rationale: consistency with existing workspace convention (separate agent files per platform), and OpenCode does not support Claude Code extended frontmatter fields.

3. **Skill deliverable location.** Decision: `prompts/skills/[name]/` for architecture/test/eval, `.claude/skills/[name]/` and `.opencode/skills/[name]/` for production SKILL.md. Rationale: parallels the existing `prompts/agents/[name]/` convention while keeping production artifacts in their platform-native locations.

### 4.2 Open Questions

1. **Should OpenCode commands also be supported?** OpenCode has a separate "commands" concept (`.opencode/commands/`) with different frontmatter (`agent`, `model`, `subtask`). Skills and commands serve overlapping but distinct purposes. The current plan focuses on skills only. Commands could be added later as a separate overlay.

2. **Should the `skill-anatomy-reference` skill also be available as a knowledge base file?** Currently the research is in `knowledge/research/skill-creation-architecture.md` (this file). The `skill-anatomy-reference` skill provides a subset optimized for agent consumption. There is deliberate overlap — the knowledge file is comprehensive and human-readable; the skill is concise and agent-optimized.

3. **Should the meta-task template reference existing agent meta-task or be standalone?** Recommendation: standalone, but noting the parallel with `meta-task-create-agent.md` for users familiar with the agent pipeline.

4. **Should the AGENTS.md or CLAUDE.md be updated to mention skill creation?** The constraint says NO modifications to existing files. However, the user may want to add skill-related slash commands to CLAUDE.md in a future step. This plan does not include those modifications.

---

## Part 5: Skill Architecture Spec Template

This is the template the Planner will use instead of the agent architecture spec when creating skills.

```markdown
# Skill Architecture Spec: [Skill Name]

## Requirements Summary
- **Purpose**: [1 sentence — what task or knowledge this skill provides]
- **Target Platforms**: [Claude Code / OpenCode / Both]
- **Invocation Mode**: [User-only / Agent-only / Both]
- **Scope**: [Project / Personal / Distributable]
- **Arguments**: [None / Description of expected arguments]

## Assumptions
<!-- List any decisions made due to missing information -->

## Naming
- **Skill name**: [must match ^[a-z0-9]+(-[a-z0-9]+)*$, max 64 chars]
- **Directory name**: [must match skill name]
- **Validation**: [confirm name is valid]

## Frontmatter Design
### Standard Fields (all platforms)
- **name**: [skill name]
- **description**: [1-1024 chars, specific keywords for agent discovery]
- **license**: [if applicable]
- **compatibility**: [if applicable]
- **metadata**: [if applicable]

### Claude Code Extensions (if targeting Claude Code)
- **disable-model-invocation**: [true/false and rationale]
- **user-invocable**: [true/false and rationale]
- **allowed-tools**: [list or none]
- **model**: [override or none]
- **context**: [fork or inline and rationale]
- **agent**: [subagent type if context: fork]
- **argument-hint**: [hint text if arguments expected]

## Content Strategy
- **Content type**: [Reference / Task / Hybrid]
- **SKILL.md scope**: [what goes in the main file]
- **Supporting files**: [what goes in references/, scripts/, assets/]
- **Estimated tokens**: [target for SKILL.md body]

## Argument Design
- **Arguments accepted**: [list with descriptions]
- **Positional mapping**: [$0 = ..., $1 = ...]
- **Default behavior**: [what happens with no arguments]
- **Edge cases**: [missing args, too many args, malformed args]

## Dynamic Context (if applicable)
- **Shell commands**: [list of !`command` injections]
- **Rationale**: [why dynamic data is needed]

## Progressive Disclosure Plan
- **Tier 1 (description)**: [what the description conveys — ~100 tokens]
- **Tier 2 (SKILL.md body)**: [main instructions — target < 5000 tokens]
- **Tier 3 (references)**: [detailed docs loaded on demand]

## Cross-Platform Compatibility
- **Claude Code specifics**: [features used that are CC-only]
- **OpenCode specifics**: [any OpenCode-only considerations]
- **Fallback strategy**: [how the skill degrades on platforms that lack extended features]

## Success Criteria
<!-- How do we know the resulting skill works? -->

## Notes for Builder
<!-- Anything the Builder needs to know that does not fit above -->
```

---

## Part 6: Skill Evaluation Rubric Template

This is the rubric the Reviewer will use instead of the 7-dimension agent rubric when reviewing skills.

| Dimension | Question | Score |
|---|---|---|
| **Description Quality** | Does the description clearly convey what the skill does and when to use it? Would an agent correctly decide to load this skill based on the description alone? | /5 |
| **Naming** | Does the name follow the `^[a-z0-9]+(-[a-z0-9]+)*$` pattern? Is it intuitive and discoverable? | /5 |
| **Conciseness** | Is the SKILL.md under 500 lines? Is the body under 5000 tokens? Is every instruction earning its place? | /5 |
| **Progressive Disclosure** | Is content split appropriately? Does SKILL.md focus on essentials while references/ holds details? | /5 |
| **Argument Handling** | Are arguments well-documented? Are edge cases (missing, extra, malformed) handled? | /5 |
| **Cross-Platform** | Does the skill work on all target platforms? Are platform-specific features gracefully handled? | /5 |
| **Invocation Design** | Is the invocation mode (user/agent/both) appropriate? Is the description optimized for the chosen mode? | /5 |

**Decision thresholds:** Same as agent reviews (all >= 4: APPROVE; any 3: APPROVE WITH NOTES; any <= 2: REJECT).

---

## Implications for This Workspace

1. The PromptArchitect pipeline can support skill creation without any modifications to existing files.
2. Skills-as-overlays is the natural extension mechanism: the existing agents gain new capabilities through the very system they will help users build.
3. The Agent Skills open standard provides a stable foundation that works across Claude Code and OpenCode, reducing platform-specific complexity.
4. The main complexity lies in Claude Code's extended frontmatter fields, which need to be understood by the Planner (for architecture decisions) and the Builder (for correct YAML generation).
5. The knowledge base should be updated with this research (this file) and the INDEX.md should be updated to reference it.
