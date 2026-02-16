---
name: skillsmp-search
description: Browse and discover agent skills from the SkillsMP marketplace (skillsmp.com). Access 117K+ open-source agent skills sorted by popularity or recency. For search, visit the website directly.
---

# SkillsMP Search

Browse public skills from SkillsMP (`skillsmp.com`).

## Use When

- User wants to discover skills
- User asks for "skills marketplace" / "SkillsMP"
- You need references before creating a new skill

## Supported Commands

```text
/skillsmp-search
/skillsmp-search recent
/skillsmp-search popular
/skillsmp-search stars
```

- `recent`: latest updates
- `popular` / `stars`: ranked by GitHub stars

For keyword/semantic search, send users to https://skillsmp.com.

## Data Fetch

Use `curl` via Execute:

```bash
curl -s "https://skillsmp.com/api/skills?page=1&limit=10&sortBy=recent"
curl -s "https://skillsmp.com/api/skills?page=1&limit=10&sortBy=stars"
```

## Output Format

```markdown
## SkillsMP Results

**Query**: {Recent|Popular}
**Found**: {count}
**Page**: {page}/{total_pages}

### {skill_name}
**Author**: {author}
**Description**: {description}
**Stats**: {stars} stars | {forks} forks
**Source**: {github_url}
**Updated**: {relative_time}
```

Default to top 10 results.

## Error Handling

- If API returns HTML/Cloudflare challenge: redirect to https://skillsmp.com.
- If empty results: suggest `popular` browsing.
- If request fails: explain failure and provide website fallback.

## Response Rules

- Keep output concise and scannable.
- Include GitHub links for direct action.
- End with: `Browse more at https://skillsmp.com`.
