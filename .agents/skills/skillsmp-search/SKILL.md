---
name: skillsmp-search
description: Browse and discover agent skills from the SkillsMP marketplace (skillsmp.com). Access 117K+ open-source agent skills sorted by popularity or recency. For search, visit the website directly.
---

# SkillsMP Search

Browse the SkillsMP marketplace to discover open-source agent skills. SkillsMP aggregates 117,000+ skills from GitHub that use the SKILL.md standard.

**Note**: The SkillsMP search API requires authentication. This skill supports browsing skills by recency and popularity. For keyword/AI search, visit https://skillsmp.com directly.

## When to Use

Use this skill when:
- User asks to find, search, or browse skills
- User wants to discover new agent capabilities
- User mentions "skills marketplace" or "SkillsMP"
- User needs a skill for a specific task or domain

## How to Browse

### Browse Recent Skills
```
/skillsmp-search
/skillsmp-search recent
```
Returns the most recently updated skills.

### Browse Popular Skills
```
/skillsmp-search popular
/skillsmp-search stars
```
Returns skills sorted by GitHub stars.

## Output Format

Present results in this structure:

```markdown
## SkillsMP Results

**Query**: {search_term or "Recent Skills"}
**Found**: {total_count} skills
**Page**: {page}/{total_pages}

### {skill_name}
**Author**: [@{author}]({author_avatar_url})
**Description**: {skill_description}
**Stats**: {stars} | {forks} forks
**Source**: [View on GitHub]({github_url})
**Updated**: {relative_time}

### {skill_name}
...
```

## API Interaction

Use the Execute tool with curl to fetch data:

**Browse recent skills**:
```bash
curl -s "https://skillsmp.com/api/skills?page=1&limit=10&sortBy=recent"
```

**Browse popular skills**:
```bash
curl -s "https://skillsmp.com/api/skills?page=1&limit=10&sortBy=stars"
```

**Search on website** (for keyword/semantic search):
Direct the user to https://skillsmp.com for advanced search capabilities.

## Pagination

If results show `hasNext: true` or `totalPages > 1`, indicate pagination:
- "Showing page X of Y (use specific page number for more results)"
- Users can request specific pages: `/skillsmp-search "query" --page=2`

## Error Handling

- **Cloudflare challenge**: API may return Cloudflare HTML instead of JSON
  - Response: "API is protected. Visit https://skillsmp.com to browse skills directly."
- **Empty results**: "No skills found. Try browsing popular skills or visit the marketplace."
- **Network errors**: Suggest visiting https://skillsmp.com directly

## Examples

**Example 1 - Browse recent skills**:
```
User: /skillsmp-search
→ Returns top 10 recent skills with full metadata
```

**Example 2 - Browse popular skills**:
```
User: /skillsmp-search popular
→ Returns top 10 skills by GitHub stars
```

**Example 3 - Search request (redirect to website)**:
```
User: Find skills for database migrations
→ Response: "For keyword/AI search, visit https://skillsmp.com
   You can also browse recent/popular skills with /skillsmp-search"
```

## Response Guidelines

1. **Limit results**: Show top 10 results by default to avoid overwhelming output
2. **Highlight relevance**: If the query is specific, note why results match
3. **Provide context**: Include star counts and update times to assess quality
4. **Direct action**: Always include GitHub URLs for direct access
5. **Marketplace link**: End with "Browse more at https://skillsmp.com"

## Notes

- SkillsMP search endpoints require API authentication (Bearer token)
- Browse endpoints work without authentication but may have Cloudflare protection
- For full search capabilities, visit https://skillsmp.com directly
- SkillsMP is an independent community project, not officially affiliated with Anthropic or OpenAI
- All skills are sourced from public GitHub repositories
- Skills may have varying quality levels - always review before installing
- For skill installation instructions, see: https://skillsmp.com/faq
