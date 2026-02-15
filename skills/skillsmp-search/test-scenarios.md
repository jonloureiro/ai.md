# SkillsMP Search - Test Scenarios

## Scenario 1: Happy Path (Browse Recent Skills)

**Input**: `/skillsmp-search` or `/skillsmp-search recent`

**Expected Behavior**:
1. Skill constructs API request to browse endpoint with sortBy=recent
2. Parses JSON response for skill results
3. Formats output with skill name, author, description, stats, and GitHub URL
4. Returns top results sorted by recency

**Success Criteria**:
- Results are displayed in structured markdown format
- Each result includes: name, author, description, stars, forks, GitHub URL
- Sort mode and total count are shown in header

---

## Scenario 2: Happy Path (Browse Popular Skills)

**Input**: `/skillsmp-search popular` or `/skillsmp-search stars`

**Expected Behavior**:
1. Skill detects "popular" or "stars" argument
2. Constructs API request with sortBy=stars
3. Returns results sorted by GitHub star count (most popular first)

**Success Criteria**:
- Results are sorted by popularity
- Output format matches recent skills browse
- High-star projects appear at the top

---

## Scenario 3: Edge Case (Cloudflare Protection)

**Input**: `/skillsmp-search` when API returns Cloudflare HTML

**Expected Behavior**:
1. API returns HTML instead of JSON (Cloudflare challenge)
2. Skill detects non-JSON response
3. Provides helpful message directing user to website
4. Suggests visiting https://skillsmp.com directly

**Success Criteria**:
- Clear error message about API protection
- Directs user to website as fallback
- No crash or raw HTML displayed

---

## Scenario 4: Edge Case (Search Request - Redirect to Website)

**Input**: `/skillsmp-search "git automation"`

**Expected Behavior**:
1. User provides a search query
2. Skill informs user that search requires website visit
3. Provides helpful message with direct link
4. May offer to browse recent/popular skills instead

**Success Criteria**:
- Clear explanation that search is not available via API
- Direct link to https://skillsmp.com with query pre-filled if possible
- Helpful alternative suggestion (browse modes)

---

## Scenario 5: Edge Case (Pagination)

**Input**: `/skillsmp-search popular` (returns many pages)

**Expected Behavior**:
1. API returns first page with pagination metadata
2. Skill shows page 1 results
3. Indicates total pages available
4. Notes that user can browse more on the website

**Success Criteria**:
- Pagination info is displayed (page X of Y)
- User knows there are more results available
- Results are limited to reasonable number (e.g., 10)

---

## Scenario 6: Failure Mode (Network Error)

**Input**: `/skillsmp-search` during network outage

**Expected Behavior**:
1. API request fails (timeout, DNS error, etc.)
2. Skill catches the error gracefully
3. Informs user of the issue
4. Suggests fallback: visit https://skillsmp.com directly

**Success Criteria**:
- Error message is user-friendly (no raw stack traces)
- Clear explanation of what went wrong
- Actionable fallback option provided

---

## Test Execution Checklist

- [ ] Scenario 1: Browse recent skills works
- [ ] Scenario 2: Browse popular skills works
- [ ] Scenario 3: Cloudflare protection handled gracefully
- [ ] Scenario 4: Search requests redirect to website
- [ ] Scenario 5: Pagination displayed correctly
- [ ] Scenario 6: Network errors handled gracefully
