# SkillsMP Search - Skill Architecture Spec

## Requirements Summary

**Purpose**: Enable users to browse and discover agent skills from the SkillsMP marketplace (skillsmp.com) directly within Claude Code or OpenCode.

**Target Platforms**: Claude Code, OpenCode (cross-compatible)

**Invocation Mode**: User-invocable with optional sort mode argument

**Scope**: Browse and retrieve skill metadata from SkillsMP. Does not install skills - only discovers them. Note: Search APIs require authentication, so browsing by sort order is the primary function.

---

## Naming

**Skill Name**: `skillsmp-search`

**Validation**:
- Pattern: `^[a-z0-9]+(-[a-z0-9]+)*$` - PASSED
- Length: 14 characters (under 64) - PASSED
- Ke bab-case: PASSED

---

## Frontmatter Design

```yaml
---
name: skillsmp-search
description: Browse and discover agent skills from the SkillsMP marketplace (skillsmp.com). Access 117K+ open-source agent skills sorted by popularity or recency. For search, visit the website directly.
---
```

**Rationale**:
- `name` follows kebab-case convention
- `description` clarifies that browsing is supported, search requires website visit
- Includes keywords for agent discovery
- No Claude Code-specific extensions needed for cross-platform compatibility

---

## Content Strategy

**Body Content** (in SKILL.md):
- Skill purpose and invocation patterns
- When to use this skill
- Browse modes (recent vs. popular)
- Output format specification
- Examples of common usage
- API limitations (search requires auth)
- Error handling for Cloudflare protection

**Progressive Disclosure**: Not required - skill is simple enough for body-only approach.

**External References**: None needed.

**Scripts**: None needed - uses Execute tool for curl requests.

---

## Argument Design

**Argument**: Optional sort mode (recent/popular/stars)

**Behavior**:
- **No argument**: Browse recent skills (default)
- **"popular" or "stars"**: Browse by GitHub star count

**Examples**:
- `/skillsmp-search` - Browse recent skills
- `/skillsmp-search popular` - Browse by popularity

---

## Cross-Platform Compatibility Plan

**Platform Considerations**:
- Both Claude Code and OpenCode support user-invocable skills
- No Claude Code-only features are used
- Execute tool (or equivalent shell tool) is available on both platforms

**Deployment**:
- Single SKILL.md file for both platforms
- Copy to both `.claude/skills/` and `.opencode/skills/`

---

## Tool Requirements

**Primary Tool**: Execute (for curl requests to SkillsMP API)

**API Endpoints**:
1. Browse Recent: `https://skillsmp.com/api/skills?sortBy=recent`
2. Browse Popular: `https://skillsmp.com/api/skills?sortBy=stars`
3. Search: REQUIRES API KEY - direct users to website instead

**Parameters**:
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 20, max: 100)
- `sortBy`: Sort by "stars" or "recent"

---

## Output Format

**Structured Output**:
```
## SkillsMP Browse Results

**Sort**: {recent or popular}
**Found**: {total} skills
**Page**: {current}/{total_pages}

### {skill_name}
**Author**: [@{author}]({author_avatar_url})
**Description**: {skill_description}
**Stats**: {stars} | {forks} forks
**Source**: [View on GitHub]({github_url})
**Updated**: {relative_time}

### {skill_name}
...
```

---

## Success Criteria

1. User can browse skills by recency
2. User can browse skills by popularity
3. Results are formatted clearly with relevant metadata
4. Pagination is indicated when results exceed one page
5. API errors are handled gracefully
6. Skill works on both Claude Code and OpenCode
7. Users are directed to website for search functionality

---

## Version

**Version**: v1.0 (initial release)
**Target Model**: Any (tool-using model)
**Estimated Token Count**: ~1200 tokens
