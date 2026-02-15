# Platform Agent Mechanics — Claude Code & OpenCode

> Last updated: 2026-02-15
> Status: Current

## Summary

How agents and subagents actually work in Claude Code and OpenCode. Critical for designing agents that are compatible with both platforms.

## Sources

- [Create custom subagents — Claude Code Docs](https://code.claude.com/docs/en/sub-agents)
- [Orchestrate teams of Claude Code sessions](https://code.claude.com/docs/en/agent-teams)
- [Agents — OpenCode](https://opencode.ai/docs/agents/)
- [Tools — OpenCode](https://opencode.ai/docs/tools/)

## Key Findings

### 1. Context Window Behavior (Both Platforms)

**Every agent/subagent invocation creates a fresh, isolated context window.**

| Action | Context |
|---|---|
| Switch agent (e.g., `/planner` → `/builder`) | New session, context resets |
| Spawn subagent via Task tool | New isolated window, only receives task description |
| Continue in same session | Same context, accumulative |
| Resume agent (Claude Code only) | Resumes previous context via agent ID |

### 2. Claude Code Specifics

**Agent definition**: `.claude/agents/*.md` with YAML frontmatter (`name`, `model`, `color`) + markdown body as system prompt.

**Subagent invocation**: Via `Task` tool with `subagent_type` parameter.

**Built-in subagent types**: Explore, Plan, General-purpose, Bash, StatusLine-setup, Claude Code Guide.

**Key constraints:**
- Subagents CANNOT spawn other subagents (no nesting)
- Agents CAN invoke subagents if Task tool is available
- Custom agents defined in `.claude/agents/` are invocable as subagents
- Agent priority: CLI > Project > User > Plugin (highest-priority scope wins on name collision)
- Subagent results return compressed to parent (not full transcript)

**Frontmatter fields**: `name`, `description`, `model` (opus/sonnet/haiku), `color`, `allowedTools`, `permissionMode`.

### 3. OpenCode Specifics

**Agent definition**: `.opencode/agents/*.md` with YAML frontmatter + markdown body.

**Subagent invocation**: Three mechanisms:
1. `call_omo_agent` — Synchronous, blocking, direct invocation
2. `delegate_task` — Background, non-blocking, parallel
3. `task` tool — Session-based, creates child session with `parentID`

**User invocation**: `@agent-name` syntax for manual subagent calls.

**Key constraints:**
- Child sessions created with `parentID` pointing to parent
- Context NOT carried over between agent switches
- Session navigation: `<Leader>+Right/Left` to cycle parent/child sessions
- No automatic return to parent after subagent completes (manual switch needed)

**Frontmatter fields**: `description`, `temperature`, `max_turns`, `tools` (object with tool names), `model`, `permissions`, `mode` ("primary"/"subagent"/"all"), `hidden`, `color`, `disable`.

**Advanced**: Hive-Mind swarm coordination via `.hive/` directory with git-backed task tracking.

### 4. Cross-Platform Compatibility

| Feature | Claude Code | OpenCode |
|---|---|---|
| Agent file format | YAML frontmatter + MD | YAML frontmatter + MD |
| Agent location | `.claude/agents/` | `.opencode/agents/` |
| System prompt | Markdown body | Markdown body |
| Model config | `model: opus` | `model: anthropic/claude-...` |
| Tools config | `allowedTools` list | `tools` object (key: bool) |
| Temperature | Not configurable per-agent | `temperature` field |
| Max turns | Not configurable per-agent | `max_turns` field |
| Subagent spawning | Task tool | task/delegate_task/call_omo_agent |
| Subagent nesting | Blocked (no nesting) | Supported (child sessions) |

**Compatibility strategy**: Keep markdown body (system prompt) identical across both platforms. Only frontmatter differs. The body IS the agent — frontmatter is platform config.

### 5. Hybrid Agent Design (Agent + Subagent)

An agent can function as both:
- **Primary agent**: User invokes directly via `/agent-name`
- **Subagent**: Orchestrator invokes via Task tool

This dual role requires no special configuration — the same agent file works for both. The system prompt should be written to handle both contexts (being invoked directly by a user, or receiving a curated task from an orchestrator).

## Implications for This Workspace

- Agent body (system prompt) is platform-agnostic — write once, configure frontmatter per platform.
- The orchestrator pattern works on both platforms but with different invocation mechanics.
- No nesting in Claude Code means the orchestrator must be the ONLY level that spawns subagents.
- OpenCode's `mode` field can explicitly mark agents as "subagent" to hide from direct user selection.
