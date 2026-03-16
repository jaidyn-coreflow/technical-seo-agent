# Report Template — Full Audit

This is the template for full audit reports. When the user requests a comprehensive audit ("audit this crawl", "run a full technical SEO audit", "check everything"), produce the report in this format. Fill in every section with real data from the crawl. Remove any section that has zero issues — do not include empty severity groups.

---

## Template

```markdown
# Technical SEO Audit — [domain] — [date]

## Summary

| Metric | Value |
|--------|-------|
| Pages crawled | [total] |
| Critical issues | [count] |
| High issues | [count] |
| Medium issues | [count] |
| Low issues | [count] |
| **Top priority** | [One-liner describing the single most impactful issue — e.g., "156 inlinks pointing at 42 deleted blog posts returning 404"] |

---

## Critical Issues

### [Issue Name]

- **Category:** [Crawlability / On-Page / Link Structure / Performance / Structured Data]
- **Affected pages:** [count] ([percentage]% of crawled pages)
- **Impact:** [Why this matters — what is being lost or broken. Be specific: indexation loss, equity waste, crawl budget drain, broken user journeys.]
- **Action:** [Specific steps to fix. Not "add redirects" but "301 redirect the 42 deleted blog posts to their closest topical match; update the Related Posts widget to exclude non-200 URLs."]

**Sample URLs:**

| URL | [Relevant Column] | [Relevant Column] |
|-----|-------------------|-------------------|
| /example/page-1 | ... | ... |
| /example/page-2 | ... | ... |
| ... | ... | ... |

_Showing [X] of [total]. [Pattern note if applicable — e.g., "38 of 42 are under `/blog/2023/*`"]_

---

## High Priority

### [Issue Name]

- **Category:** [Crawlability / On-Page / Link Structure / Performance / Structured Data]
- **Affected pages:** [count] ([percentage]% of crawled pages)
- **Impact:** [Why this matters]
- **Action:** [Specific steps to fix]

**Sample URLs:**

| URL | [Relevant Column] | [Relevant Column] |
|-----|-------------------|-------------------|
| /example/page-1 | ... | ... |
| ... | ... | ... |

_Showing [X] of [total]. [Pattern note if applicable]_

---

## Medium Priority

### [Issue Name]

- **Category:** [Crawlability / On-Page / Link Structure / Performance / Structured Data]
- **Affected pages:** [count] ([percentage]% of crawled pages)
- **Impact:** [Why this matters]
- **Action:** [Specific steps to fix]

**Sample URLs:**

| URL | [Relevant Column] | [Relevant Column] |
|-----|-------------------|-------------------|
| /example/page-1 | ... | ... |
| ... | ... | ... |

_Showing [X] of [total]. [Pattern note if applicable]_

---

## Low Priority

### [Issue Name]

- **Category:** [Crawlability / On-Page / Link Structure / Performance / Structured Data]
- **Affected pages:** [count] ([percentage]%)
- **Action:** [Brief fix — these are minor, so keep it concise]

_[Sample URLs optional for low-priority issues. Include a brief table only if the pattern is not obvious from the description.]_

---

## Next Steps

Prioritized actions ordered by impact. Address these in order:

1. **[Action]** — [Why first: brief rationale tied to severity and scope]
2. **[Action]** — [Rationale]
3. **[Action]** — [Rationale]
4. **[Action]** — [Rationale]
5. **[Action]** — [Rationale]
```

---

## Severity Classification Guide

Use this table to assign severity to each issue found during the audit. Severity is based on the type of impact, not just the number of pages affected — though scale can elevate severity (e.g., a template-level title issue affecting 2,000 pages is more severe than the same issue on 3 pages).

| Severity | Criteria | Examples |
|----------|----------|----------|
| **Critical** | Directly prevents indexation or causes major crawl/ranking damage. These issues mean pages that should be indexed are not, or significant link equity is being destroyed. | 404s with inlinks, redirect loops, noindex on important pages, canonical pointing to non-200, robots.txt blocking key sections, redirect chains ending in 4xx/5xx |
| **High** | Significantly impacts rankings or crawl efficiency but does not completely block indexation. Left unfixed, these will measurably hurt organic performance. | Missing titles on key page types, redirect chains (3+ hops), orphan pages on important content, broken internal links, noindexed pages receiving significant inlinks, orphan pages present in sitemap |
| **Medium** | Affects SEO performance but is not urgent. These are real issues that should be fixed, but they are not actively destroying value — they are leaving value on the table. | Duplicate meta descriptions, suboptimal canonical configuration, low-inlink pages that matter, multiple H1s, title length issues at scale, pages at crawl depth 5+ |
| **Low** | Minor optimizations with marginal individual impact. Fix these when the higher-priority items are resolved, or batch them into routine maintenance. | Title character count edge cases, missing Open Graph tags, anchor text improvements, missing hreflang x-default, non-critical image alt text, minor page size optimizations |

### Severity escalation

An issue's severity can be elevated when cross-referencing reveals compounding factors:

- A 404 with zero inlinks is **Medium**. A 404 with 30 inlinks is **Critical**.
- An orphan page on its own is **High**. An orphan page that is also in the XML sitemap is **High** with an explicit note about the contradictory signal.
- A noindex page is expected on utility pages. A noindex page receiving significant inlinks from key pages is **High**.
- A slow page (TTFB > 2s) is **Medium** in general. A slow page that is a high-traffic category page is **Critical**.

Always apply judgment based on the actual data, not just the issue type.

---

## Report Guidelines

Follow these rules when producing an audit report:

1. **Always include the Summary table.** Every report starts with the summary showing total pages crawled and issue counts broken out by severity. This gives the reader immediate context before they see any details.

2. **Group by severity, not by SEO domain.** The report sections are Critical, High, Medium, Low — not Crawlability, On-Page, Performance. A reader should be able to read top-down and encounter the most important issues first, regardless of which SEO domain they fall into.

3. **Show count AND percentage for every issue.** "147 pages" means nothing without context. "147 pages (0.96% of 15,234 crawled)" tells the reader whether this is a widespread problem or a contained one.

4. **Up to 10 sample URLs per issue.** Do not dump hundreds of rows. Show the top 10 most relevant examples (sorted by impact — e.g., highest inlink count for 404s, longest response time for slow pages). If a clear URL pattern exists, call it out: "38 of 42 are under `/blog/2023/*`". Offer to show more if needed.

5. **Next Steps: 5-7 actions maximum.** The reader needs a prioritized punch list, not a laundry list. Each action should be specific and tied to a finding in the report. Order by impact-to-effort ratio. "Implement 301 redirects for the 42 deleted blog posts" is good. "Fix your redirects" is not.

6. **Omit empty severity groups.** If the crawl has zero critical issues, do not include a "Critical Issues" section that says "None found." Just skip it and start at the highest severity that has findings.

7. **Note gaps in crawl coverage.** If a data source was unavailable (e.g., structured data extraction was not enabled in the crawl), include a brief note so the reader knows that domain was not assessed. Do not silently skip it.
