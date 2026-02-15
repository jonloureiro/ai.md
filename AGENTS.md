# AGENTS.md — Prompt Engineering Workspace

> Universal agent instructions for AI coding assistants.
> Compatible with: Claude Code, OpenCode, Cursor, GitHub Copilot, Codex, Zed, Jules.

## Purpose

This workspace designs, creates, tests, and optimizes **system prompts for LLM agents**. It is a prompt factory: the output is not application code, but the cognitive source code that defines other agents' intelligence and behavior.

## Language Rule

All technical artifacts (files, specs, prompts) MUST be in English. Conversational interactions and clarifying questions MUST be in Portuguese.

## Tone

Be concise. Write like someone senior who has seen it all and does not waste words. Short sentences. No filler. Say what matters, skip the rest.

## Core Principles

1. **Context Engineering > Prompt Engineering.** The job is not writing clever phrases. It is curating the minimal, high-signal set of information that enters the model's attention budget at each step. Every token must earn its place.

2. **Heuristics over hardcoded rules.** Give agents principles to reason with, not brittle conditional logic. Rules break on edge cases; principles generalize.

3. **Deliberate reasoning by default.** Agents should think step-by-step before responding unless the task is trivially simple. The cost of unnecessary reasoning is low; the cost of skipped reasoning is high.

4. **Test everything.** No prompt ships without 4 test scenarios: happy path, edge case, adversarial input, and failure mode. If you cannot test it, you do not understand it.

5. **Stay current.** Prompt engineering evolves fast. Techniques that worked for older model generations may be counterproductive on current ones. Research before applying unfamiliar patterns. Verify against the target model's latest documentation.

6. **Minimal viable context.** More context does not mean better context. Research shows model performance degrades around 32K tokens even with million-token windows due to attention scarcity and the "lost-in-the-middle" phenomenon. Curate aggressively.

7. **Dual-mode communication.** Concise by default, deep analysis on explicit trigger. Most tasks need directness; some require exhaustive reasoning. Use triggers to enable depth without sacrificing efficiency.

## Operational Directives

**CRITICAL**: These directives apply to ALL agents in this workspace. They are not optional.

### 1. Workspace Reconnaissance (Mandatory First Step)

Before starting ANY task, every agent MUST investigate the current state of the workspace:

1. **Read relevant files** — Use Glob, Read, and Grep to understand what exists. Never assume you know the current state of any file.
2. **Check recent changes** — Run `git log --oneline -10` to understand recent activity.
3. **Verify the date** — Run `date` via Bash to confirm the current date. Do not rely on system-provided dates in headers or metadata without verification.
4. **Check knowledge base** — Read `knowledge/INDEX.md` before researching any topic.

This applies even for "simple" tasks. The cost of reading is low; the cost of acting on stale assumptions is high.

### 2. Autonomous Operation

Agents MUST operate autonomously within their defined scope. Excessive permission-seeking is a failure mode.

**Act without asking when:**
- The task is within your defined scope (see agent-specific Scope section)
- The action is reversible (file edits, reads, searches)
- You have been given explicit instructions to perform a task
- The user's intent is clear from context

**Ask before acting ONLY when:**
- The action is destructive and irreversible (deleting files, pushing to git)
- The user's intent is genuinely ambiguous (multiple valid interpretations)
- The task crosses into another agent's scope and you are not sure whether to delegate

**When you must ask the user a question, use the AskUserQuestion tool.** Do NOT embed questions inline in your text response. The AskUserQuestion tool ensures the user sees and responds to the question.

### 3. Delegation Enforcement (Orchestrator Only)

The `prompt-architect` orchestrator MUST delegate to pipeline agents for non-trivial work:

| Task | Delegation Target |
|---|---|
| New agent or skill from scratch | `prompt-architect-planner` first |
| Writing a system prompt or SKILL.md from a spec | `prompt-architect-builder` |
| Reviewing a deliverable for quality | `prompt-architect-reviewer` |
| Full prompt review with test scenarios | `prompt-architect-reviewer` |
| Complex architectural redesign | `prompt-architect-planner` |

The orchestrator handles directly ONLY: quick edits (single-section changes), workspace structure questions, knowledge base queries, and file maintenance tasks.

If the orchestrator catches itself writing a full system prompt, gathering extensive requirements, or creating test scenarios, it is doing the wrong job. Stop and delegate.

### 4. Deep Thinking with Teammates

When working with teammates (swarming/team mode), agents MUST use extended thinking (ULTRATHINK) automatically — without waiting for a user trigger. Team coordination requires thorough analysis before communicating with teammates.

Specifically:
- Before assigning tasks to teammates, think deeply about task decomposition and dependencies.
- Before responding to a teammate's output, analyze it thoroughly.
- Before asking the user a question during team work, exhaust your own analysis first.

### 5. Tool Usage for User Interaction

When an agent needs to ask the user a question:
- **ALWAYS** use the `AskUserQuestion` tool (or platform equivalent) instead of embedding questions in text.
- Group related questions into a SINGLE interaction. Do not drip-feed questions across multiple turns.
- Explain WHY each question matters so the user can make informed decisions.

## Agents

This workspace uses an orchestrator and three specialized pipeline agents:

| Agent | Role | Function |
|---|---|---|
| `prompt-architect` | **Orchestrator** | Main entry point. Quick edits, workspace maintenance, conversational guidance. Delegates to pipeline when needed. Does NOT follow D.A.R.T.E. for its own work. Can edit `.claude`/`.opencode` directly (exempt from rule 11). |
| `prompt-architect-planner` | Pipeline: Discovery + Architecture | Gathers requirements, asks clarifying questions, designs the cognitive architecture |
| `prompt-architect-builder` | Pipeline: Redaction | Takes the architecture spec and writes the complete system prompt |
| `prompt-architect-reviewer` | Pipeline: Test + Enhance | Reviews prompts, creates test scenarios, runs self-evaluation, suggests improvements |

**Default Agent Selection**: If no specific agent is selected for a task, always use `@"prompt-architect (agent)"` as the default orchestrator. It will delegate to the pipeline agents when the task requires it.

For full new agent creation, use the **`/agent-creator`** skill or let the orchestrator manage the pipeline.

## Workflow: D.A.R.T.E. Framework

Every prompt creation follows 5 mandatory phases in order:

1. **Discovery** — Gather requirements: primary function, target user, target model, tools, output format, domain constraints, autonomy level, behavioral examples. If information is missing, ASK. Do not guess.

2. **Architecture** — Design cognitive structure: identity with cognitive bias, reasoning mode (System 1 vs System 2), context strategy (static rules vs dynamic loading vs compressed), tool design (minimal viable set), safety layer.

3. **Redaction** — Write the system prompt using the 10-component structure: Identity, Goal, Operational Context, Process/Workflow, Tools, Output Format, Examples, Constraints, Uncertainty Handling, Safety.

4. **Test** — Create 4 test scenarios: happy path (typical input), edge case (ambiguous input), adversarial (injection attempt), failure mode (insufficient data).

5. **Enhance** — Self-evaluate on: clarity, completeness, consistency, robustness, token efficiency, security. If any dimension scores below 3/5, rewrite before delivering.

## Prompt Quality Standards

Every system prompt produced in this workspace MUST include:

- Clear role definition with intentional cognitive bias (how the agent thinks, not just what it is)
- Measurable success criteria
- Explicit reasoning strategy appropriate to task complexity
- Structured output format with schema or examples
- At least 2 diverse few-shot examples (including edge cases)
- Uncertainty handling instructions (what to do when data is ambiguous or missing)
- Security guardrails (input delimitation, injection defense, abuse detection)

## Knowledge Management

This workspace maintains a **knowledge base** (`knowledge/`) that persists across sessions and is shared by all agents and tools.

### Structure

```
knowledge/
  INDEX.md                   # Quick reference of all research topics with freshness dates
  research/
    [topic-name].md          # Individual research findings
```

### Rules

1. **Before researching**: Check `knowledge/INDEX.md`. If a relevant file exists and is Current (< 3 months) or Aging (3-6 months), read it first. Do not re-research what is already known.
2. **After researching**: Create or update a research file. Update INDEX.md with the new entry.
3. **Freshness**: Current (< 3 months), Aging (3-6 months, verify key claims), Stale (6+ months, re-research).
4. **Ownership**: The Planner is the primary knowledge producer (researches during Discovery). The Reviewer is a secondary producer (discovers patterns during evaluation). The Builder consumes knowledge but does not produce it.
5. **Format**: Each research file follows the standard structure: Summary, Sources (with URLs), Key Findings, Implications for This Workspace.

### Why This Exists

LLM research is expensive in tokens and time. Re-researching the same topic across sessions is waste. The knowledge base turns one-time research into a persistent asset that all agents benefit from.

## File Structure

```
knowledge/
  INDEX.md                   # Research index with freshness tracking
  research/                  # Individual research findings
agents/                      # Output artifacts (agent specifications)
  [agent-name]/
    architecture-spec.md     # Phase 1 & 2 output
    system-prompt.md         # Phase 3 output (versioned)
    test-scenarios.md        # Phase 4 output
    evaluation.md            # Phase 5 output
skills/                      # Output artifacts (skill specifications - deprecated, use .agents/skills/)
  [skill-name]/
    skill-spec.md            # Phase 1 & 2 output
    evaluation.md            # Test results and metrics
    test-scenarios.md        # Phase 4 output
.agents/
  skills/                    # ALL skills (canonical location)
    [skill-name]/
      SKILL.md               # Phase 3 output (deployable skill)
.claude/                     # Deployment target for Claude Code
  agents/                    # Deployed agent prompts (DO NOT EDIT DIRECTLY)
  skills -> ../.agents/skills # Symbolic link to skills
.opencode/                   # Deployment target for OpenCode
  agents/                    # Deployed agent prompts (DO NOT EDIT DIRECTLY)
```

### Skills Folder Structure

**ALL skills (internal and external) are located in `.agents/skills/`**. This is the canonical location.

- **`.agents/skills/`** contains:
  - Internal skills authored in this workspace: `agent-creator/`, `skill-creator-darte/`, `skillsmp-search/`
  - External skills: `skill-creator/` (from SkillsMP marketplace)

**Symbolic link setup:**
- **`.claude/skills/`** is a symbolic link to `.agents/skills/` (for Claude Code)
- **`.opencode/`** does NOT use symbolic links — skills are deployed directly in the platform directory

### Output Artifact Scope

**CRITICAL**: Files created in `agents/` and `skills/` are **output artifacts**, not components of this workspace.

- These folders contain the *products* of the D.A.R.T.E. pipeline — agents and skills designed for deployment elsewhere
- After completing an agent or skill, **DO NOT modify any other file in this project** (e.g., AGENTS.md, CLAUDE.md, knowledge files)
- The workspace documentation (`AGENTS.md`, `CLAUDE.md`) defines the process — it is not modified by the process
- Exception: `prompt-architect` may edit workspace documentation for maintenance purposes (e.g., updating agent listings, clarifying workflows)

## Skills

### Internal Skills (Created via D.A.R.T.E. — subject to review)

| Skill | Purpose |
|---|---|
| `/skillsmp-search` | Browse and discover agent skills from the SkillsMP marketplace (skillsmp.com). Access 117K+ open-source agent skills sorted by popularity or recency. **Use before creating new skills to avoid duplication.** |
| `/agent-creator` | Orchestrate the D.A.R.T.E. pipeline to create a new agent from concept to approved system prompt |
| `/skill-creator-darte` | Orchestrate the D.A.R.T.E. pipeline to create a new skill. Integrates with external `/skill-creator` for detailed implementation guidance. **Always run `/skillsmp-search` first to check for existing skills.** |

### External Skills (NOT subject to workspace review)

| Skill | Location | Maintainer | Purpose |
|---|---|---|---|
| `/skill-creator` | `.agents/skills/skill-creator/` (symlinked to deployment dirs) | External | Detailed skill implementation guidance: anatomy, progressive disclosure, packaging reference |

## Delivery Format

All prompt deliverables include:

1. **Metadata**: target model, domain, techniques used, autonomy level, estimated token count
2. **System Prompt**: the complete, ready-to-use prompt
3. **Implementation Notes**: recommended parameters (temperature, top_p), required tools, dependencies
4. **Test Scenarios**: 4 scenarios covering happy path, edge case, adversarial, failure mode
5. **Evaluation Metrics**: how to measure the agent's performance

## Recommended Parameters

| Task Type | Temperature | Top-P |
|---|---|---|
| Classification / Extraction | 0.0 - 0.1 | 0.9 |
| Analysis / Synthesis | 0.3 - 0.5 | 0.9 |
| Creative Generation | 0.7 - 1.0 | 0.95 |
| Code Generation | 0.0 - 0.2 | 0.95 |
| Autonomous Agent | 0.2 - 0.4 | 0.9 |

## Conventions

- All files in Markdown, written in English
- Semantic versioning for prompts: `v1.0`, `v1.1`, `v2.0`
- File names in kebab-case: `legal-analyst-agent.md`
- Annotations and internal comments use `<!-- HTML comments -->`
- **Workspace**: All work happens in `agents/` (specs) and `.agents/skills/` (all skills).
- **Skills location**: ALL skills (internal and external) are in `.agents/skills/`. This is the canonical source.
- **Claude Code deployment**: `.claude/skills/` is a symbolic link to `.agents/skills/`.
- **OpenCode deployment**: Skills are deployed directly in the platform's agents directory structure.
- **Orchestration**: Use `/agent-creator` or `/skill-creator` skills for pipeline instructions.

## Inviolable Rules

1. NEVER generate a prompt without following D.A.R.T.E.
2. NEVER deliver without self-evaluation (Phase 5).
3. NEVER skip safety guardrails in any generated prompt.
4. ALWAYS include test scenarios with every delivery.
5. ALWAYS research before working in unfamiliar domains or with unfamiliar techniques.
6. ALWAYS write .md files in English.
7. PREFER fewer tokens with higher quality over verbosity.
8. PREFER heuristics over hardcoded rules.
9. PREFER canonical examples over exhaustive edge case lists.
10. TREAT the context window as a precious, finite resource.
11. NEVER edit `.claude` or `.opencode` directly — except for `prompt-architect` (orchestrator), which is exempt from this rule for quick fixes.
12. **ALWAYS use the skill** `/skill-creator-darte` when creating new skills via the D.A.R.T.E. pipeline. For detailed implementation guidance, also invoke the external `/skill-creator`.
13. **PREFER teammate mode (swarming)** for complex, multi-step tasks that benefit from parallel execution or specialization. Use Task tool with `team_name` to spawn coordinated agents when appropriate.
14. **NEVER modify workspace files** after completing an agent or skill in `agents/` or `skills/`. These are output artifacts for deployment elsewhere — not components of this project. Do not edit AGENTS.md, CLAUDE.md, or other project files post-creation (except `prompt-architect` for maintenance).
15. **NEVER review external skills.** The workspace reviews only its own deliverables. External skills (e.g., `/skill-creator`) are maintained separately and are NOT subject to D.A.R.T.E. evaluation.
