---
name: j-prompt-creator
description: Create or revise prompts for j-agents in the j-orc pipeline. Use when you need evidence-backed, knowledge-first prompt authoring with mandatory parallel research and enforced Task delegation through subagent_type "j-orc".
---

# j-prompt-creator

Create production-ready prompts for `j-prd`, `j-tec`, `j-exe`, or `j-orc` while preserving orchestration contracts and evidence traceability.

## Primary Target and Compatibility

- Primary target: Gemini CLI workflows.
- Also compatible with Factory, Claude Code, and OpenCode workflows when equivalent tools exist.
- Keep instructions tool-agnostic where possible, but never relax the delegation contract below.

## Non-Negotiable Rules

1. **Knowledge-first only**
   - Do not use unstated common knowledge for architecture rules, tool behavior, or workflow constraints.
   - Every critical rule must be supported by explicit evidence from files or URLs.
   - If evidence is missing, mark the item as `UNKNOWN` and research before drafting.

2. **Mandatory high-volume parallel research**
   - Before drafting, run at least **two parallel research waves**.
   - Wave 1: at least **8 parallel operations** for landscape mapping.
   - Wave 2: at least **6 parallel operations** for gap-closing validation.
   - Use at least four evidence classes:
     - local j-agent architecture specs
     - local knowledge/research files
     - live platform docs or source code
     - existing deployed j-agent prompt files or droids
   - Do not start prompt writing until both waves finish.

3. **Task delegation guarantee**
   - Generated prompts must explicitly require Task tool delegation through:
     - `subagent_type: "j-orc"`
   - This requirement must appear as enforceable instruction, not optional guidance.
   - If this snippet is absent, output is invalid.

4. **Prompt quality discipline**
   - Prefer concise natural language over aggressive formatting.
   - Use emphasis only for truly critical rules.
   - Remove redundant prose and keep high signal-to-noise ratio.
   - Separate mandatory gates from recommendations.

## Required Research Ledger

Before drafting, produce an internal ledger with this structure:

```markdown
| Claim or Rule | Evidence | Why it matters |
|---|---|---|
| ... | path-or-url | impact on prompt behavior |
```

Requirements:
- At least 10 evidence rows.
- Every hard constraint in the final prompt maps to at least one row.

## Authoring Workflow

1. **Scope the target**
   - Identify target prompt type: `j-prd`, `j-tec`, `j-exe`, or `j-orc`.
   - Capture required behaviors, exclusions, and output contract.

2. **Run parallel research waves**
   - Execute mandatory waves and complete the evidence ledger.
   - Resolve contradictions before writing.

3. **Synthesize constraints**
   - Classify findings into:
     - hard gates
     - default behaviors
     - fallback behavior for unknowns
   - Reject unsupported assumptions.

4. **Draft the prompt**
   - Build clear sections with deterministic behavior and completion criteria.
   - Encode mandatory orchestration and approval gates when relevant.

5. **Self-check**
   - Verify every hard rule is evidence-backed.
   - Verify delegation contract snippet exists exactly as required.
   - Verify language quality and concision.

## Mandatory Delegation Snippet

Include this exact pattern in generated prompts whenever orchestration entrypoint is defined:

```text
Task(
  subagent_type: "j-orc",
  description: "<3-5 word label>",
  prompt: "<curated orchestration handoff>"
)
```

If generating a specialist prompt (`j-prd`, `j-tec`, `j-exe`), ensure the prompt states it is expected to be invoked by `j-orc` via Task handoff context.

## Required Prompt Sections

Your generated prompt must include, at minimum:

1. Role and scope
2. Startup protocol
3. Knowledge-first research policy
4. Parallel research protocol
5. Delegation protocol with required Task snippet
6. Safety and approval gates
7. Validation or completion gates
8. Output format contract

## Output Format

Return results in this structure:

```markdown
# Prompt Package: <target-name>

## Generated Prompt
<full prompt text>

## Evidence Map
- Rule: ...
  - Source: ...
  - Rationale: ...

## Validation Checklist
- [ ] Knowledge-first constraints enforced
- [ ] Two parallel research waves completed
- [ ] Task subagent_type "j-orc" requirement included
- [ ] No unsupported assumptions
- [ ] Language is concise and high-signal

## Open Risks
- ...
```

## Failure Behavior

- If evidence is insufficient, do not fabricate.
- Report exactly what is missing and what must be researched next.
- Never mark the work complete while hard constraints remain unverified.
