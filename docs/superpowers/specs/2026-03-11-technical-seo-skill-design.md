# Technical SEO Agent — Skill Design Spec

## Overview

A Claude Code skill that acts as a trained technical SEO specialist. The user (SEO manager) runs crawls in the Screaming Frog GUI, then uses this skill to analyze crawl data via the Screaming Frog MCP server. The skill handles ad-hoc queries ("show me the 404 pages") and comprehensive audits ("audit this crawl"), producing conversational responses or structured markdown reports.

## Architecture

### Approach

**Skill + Knowledge Base** — A core skill file (SKILL.md) handles workflow and persona. Separate reference files provide deep SEO expertise per domain. The skill reads relevant references based on the user's query so it loads only what it needs.

### Why This Approach

- Modular: each SEO domain can be updated independently
- Doesn't bloat context with irrelevant knowledge on ad-hoc queries
- Follows established Claude Code plugin conventions (SKILL.md + references/)

## Data Source

The skill consumes crawl data exclusively through the [Screaming Frog MCP server](https://github.com/bzsasson/screaming-frog-mcp). It does NOT trigger crawls — the user runs crawls in the Screaming Frog GUI beforehand for maximum control (custom extraction, JavaScript rendering, crawl config).

### MCP Tools Used

| Tool | Purpose |
|------|---------|
| `sf_check` | Verify SF installation, version, and license before proceeding |
| `list_crawls` | Show available crawls, let user pick one |
| `export_crawl` | Pull specific data tabs/reports from a crawl |
| `read_crawl_data` | Read exported CSV files with filtering/pagination |

### MCP Tools NOT Used

| Tool | Reason |
|------|--------|
| `crawl_site` | User runs crawls in the GUI for full control over crawl config, custom extraction, and JS rendering. Headless crawls use default settings only. |
| `crawl_status` | Not needed since we don't initiate crawls |
| `delete_crawl` | Destructive action — user manages their own crawl storage |
| `storage_summary` | Informational only, not needed for analysis |

### Prerequisites

1. Screaming Frog SEO Spider installed (v16+, paid license recommended)
2. Screaming Frog MCP server installed and configured in Claude Code
3. A completed crawl saved in SF
4. The Screaming Frog GUI must be **closed** before the skill can access crawl data (the internal database permits only single-process access)

## File Structure

```
technical-seo-agent/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── technical-seo/
│       ├── SKILL.md                  # Core skill — persona, workflow, MCP tool usage
│       ├── references/
│       │   ├── crawlability.md       # 404s, redirects, orphans, crawl depth, robots
│       │   ├── on-page.md            # Titles, metas, H1s, canonicals, hreflang
│       │   ├── link-structure.md     # Internal links, inlink distribution, anchors
│       │   ├── performance.md        # Page size, response time, resources
│       │   ├── structured-data.md    # Schema markup issues
│       │   └── export-mappings.md    # Maps SEO queries → SF export tabs/params
│       └── examples/
│           ├── ad-hoc-query.md       # Example: targeted query workflow
│           ├── full-audit.md         # Example: comprehensive audit workflow
│           └── report-template.md    # Markdown report structure for audits
└── README.md
```

## Core Skill (SKILL.md)

### Persona

A senior technical SEO specialist reporting to the user (SEO manager). Characteristics:

- **Knowledgeable**: Deep understanding of technical SEO across all domains
- **Proactive**: Flags related issues when investigating a specific query
- **Direct**: Assumes the user knows SEO — no explaining basics, focus on findings and actions
- **Data-driven**: Always backs recommendations with specific data from the crawl

### Trigger Description

Auto-invokes on phrases like: "audit this crawl", "check SEO", "show me 404s", "find missing titles", "analyze internal links", "technical SEO", "crawl analysis", "show me pages with", "SEO issues", "broken links", "redirect chains", "orphan pages", "thin content", "missing schema".

### Workflow

When triggered, the skill follows this process:

1. **Preflight check**
   - Call `sf_check` to verify Screaming Frog is installed and licensed
   - If SF is not found or the GUI is still open, tell the user what to fix and stop

2. **Identify the crawl**
   - Call `list_crawls` to show available crawls
   - If no crawls found, tell user to run a crawl in the SF GUI first and stop
   - If only one crawl exists, auto-select it
   - If multiple, present options and let user pick

3. **Understand the ask**
   - **Ad-hoc query**: User asks about a specific issue (e.g., "show me 404s", "which pages have low inlinks")
   - **Full audit**: User asks for a broad analysis (e.g., "audit this crawl", "run a full technical SEO check")

4. **Export the right data**
   - Consult `references/export-mappings.md` to determine which `export_crawl` tabs, bulk exports, and reports to pull
   - Ad-hoc: export only the specific tab(s) needed
   - Full audit: export all relevant tabs across all SEO domains

5. **Read and analyze**
   - Use `read_crawl_data` with appropriate filters to read exported CSVs
   - Read the relevant `references/*.md` file(s) for domain expertise
   - Apply SEO knowledge to interpret the data — identify issues, assess severity, determine root causes

6. **Present findings**
   - **Ad-hoc queries**: Conversational response with data tables, interpretation, and recommended actions
   - **Full audits**: Structured markdown report (see Report Format below)

### Error Handling

| Scenario | Behavior |
|----------|----------|
| SF not installed / `sf_check` fails | Tell user to install Screaming Frog and the MCP server. Stop. |
| SF GUI still open (database locked) | Tell user to close the Screaming Frog application. Stop. |
| No crawls found | Tell user to run a crawl in the SF GUI first. Stop. |
| `export_crawl` fails (invalid tab name) | Log the error, skip that tab, continue with remaining exports. Note the skipped data in output. |
| `read_crawl_data` returns empty results | Report "no issues found" for that category — this is a valid result, not an error. |
| MCP server not configured | Tell user to install and configure the screaming-frog-mcp server. Stop. |
| Very large result set (>500 rows) | Summarize with counts and patterns, show top 20 sample URLs, offer to show more on request. |

### Pagination Strategy

For large crawls, the skill handles data volume as follows:

- **Ad-hoc queries**: Use `read_crawl_data` filtering to pull only relevant rows. Show up to 20 sample URLs in the response. If more exist, state the total count and offer to show more.
- **Full audits**: For each issue category, report total affected page count and percentage. Show up to 10 sample URLs per issue in the report. Group by URL pattern when possible (e.g., "42 URLs under `/blog/2023/`").
- **Pagination**: Use `read_crawl_data`'s page/row parameters to read data in chunks if needed. Never attempt to load an entire large export into a single response.

### Smart Query Handling

The skill should interpret natural language queries and map them to the right data. Examples:

| User says | Skill does |
|-----------|-----------|
| "show me the 404 pages" | Export `Response Codes:Client Error (4xx)`, read and display |
| "which pages have low inlinks" | Export `bulk_export: All Inlinks`, aggregate inlink counts, filter for pages <3 inlinks |
| "find redirect chains" | Export `Response Codes:Redirection (3xx)` + `bulk_export: All Redirect Chains` |
| "are there duplicate titles" | Export `Page Titles:Duplicate` |
| "audit this crawl" | Export all relevant tabs, analyze each domain, produce report |

## Knowledge Base (references/)

### crawlability.md

Covers issues related to search engine access and crawling:

- **HTTP status errors**: 404s, 5xx server errors, soft 404s (200 status but error content)
- **Redirects**: Chains (3+ hops), loops, temporary (302) used where permanent (301) should be
- **Orphan pages**: Pages with zero internal inlinks — discoverable only via sitemap or external links
- **Crawl depth**: Pages requiring >3 clicks from homepage to reach
- **Robots directives**: Pages blocked by robots.txt, noindex/nofollow conflicts, meta robots vs X-Robots-Tag conflicts
- **Sitemap issues**: URLs in sitemap returning non-200 status, crawled URLs missing from sitemap
- **Pagination**: Incorrect or missing rel=prev/next, paginated series with noindex

For each issue: what it is, why it matters for SEO, severity guidance, and recommended fix.

### on-page.md

Covers on-page HTML element issues:

- **Page titles**: Missing, duplicate, too long (>60 chars), too short (<30 chars), same as H1
- **Meta descriptions**: Missing, duplicate, too long (>160 chars), too short (<70 chars)
- **Headings**: Missing H1, multiple H1s, empty H1, H1 duplicated across pages
- **Canonicals**: Missing, self-referencing issues, pointing to non-200 pages, conflicting with other signals (noindex + canonical to different page)
- **Hreflang**: Missing return tags, language code errors, self-referencing hreflang missing, x-default missing
- **Content**: Thin pages (low word count), near-duplicate content
- **Open Graph**: Missing og:title, og:description, og:image for social sharing

### link-structure.md

Covers internal linking health:

- **Low inlink pages**: Pages with fewer than 3 internal links pointing to them — signals low importance to search engines
- **Link equity distribution**: Are important pages getting the most internal links?
- **Broken internal links**: Links pointing to 404/5xx pages
- **Excessive outlinks**: Pages with >100 outgoing links diluting link equity
- **Anchor text**: Diversity and relevance of internal link anchor text
- **Link depth**: Average clicks from homepage to reach key pages
- **Opportunities**: Pages that should be linked to from high-authority internal pages but aren't

### performance.md

Covers page performance signals from crawl data:

- **Page size**: Pages >3MB total size (HTML + resources)
- **Response time**: TTFB >1 second
- **Large images**: Uncompressed or oversized image resources
- **Render-blocking resources**: CSS/JS files that delay rendering
- **Redirect latency**: Multiple redirects adding cumulative delay

### structured-data.md

Covers schema/structured data issues:

- **Missing schema**: Key page types (products, articles, FAQs, local business) without appropriate schema markup
- **Invalid schema**: Markup that won't pass Google's Rich Results Test
- **Incomplete schema**: Required properties missing for specific schema types
- **Mismatched schema**: Schema type doesn't match page intent (e.g., Article schema on a product page)

> Note: Structured data analysis depends on whether Screaming Frog's "Structured Data" tab is available in the crawl export. If the crawl was not configured to extract structured data, the skill should note this gap and recommend enabling it for future crawls rather than silently skipping it.

### export-mappings.md

The critical mapping file that translates SEO queries into Screaming Frog MCP parameters. Structure:

```markdown
## Query → Export Mapping

### Crawlability
| Query pattern | export_tabs | bulk_export | save_report |
|---------------|------------|-------------|-------------|
| 404s, client errors | Response Codes:Client Error (4xx) | — | — |
| Server errors | Response Codes:Server Error (5xx) | — | — |
| Redirects, redirect chains | Response Codes:Redirection (3xx) | All Redirect Chains | — |
| Orphan pages | — | Orphan Pages | — |

### On-Page
| Query pattern | export_tabs | bulk_export | save_report |
|---------------|------------|-------------|-------------|
| Missing titles | Page Titles:Missing | — | — |
| Duplicate titles | Page Titles:Duplicate | — | — |
| Long titles | Page Titles:Over 60 Characters | — | — |
| Missing meta descriptions | Meta Description:Missing | — | — |
| Duplicate meta descriptions | Meta Description:Duplicate | — | — |
| Missing H1 | H1:Missing | — | — |
| Multiple H1s | H1:Multiple | — | — |
| Duplicate H1s | H1:Duplicate | — | — |
| Canonicals | Canonicals:All | — | — |
| Directives, noindex | Directives:All | — | — |
| Hreflang | Hreflang:All | — | — |
| Images, alt text | Images:All | — | — |

### Link Structure
| Query pattern | export_tabs | bulk_export | save_report |
|---------------|------------|-------------|-------------|
| Internal links, inlinks | — | All Inlinks | — |
| Outlinks | — | All Outlinks | — |
| Broken links | Response Codes:Client Error (4xx) | All Inlinks | — |

### Performance
| Query pattern | export_tabs | bulk_export | save_report |
|---------------|------------|-------------|-------------|
| Response time, slow pages | Internal:All | — | — |
| Page size, large pages | Internal:All | — | — |

### Structured Data
| Query pattern | export_tabs | bulk_export | save_report |
|---------------|------------|-------------|-------------|
| Schema, structured data | Structured Data:All | — | — |

> Note: Structured data export availability depends on SF version and crawl config. If the tab is not available, the skill should report that structured data was not captured in this crawl and suggest enabling it in SF settings.

### Full Audit
| export_tabs | bulk_export | save_report |
|-------------|-------------|-------------|
| Internal:All, Response Codes:All, Page Titles:All, Meta Description:All, H1:All, Canonicals:All, Directives:All, Images:All | All Inlinks, All Redirect Chains | Crawl Overview |
```

Note: Exact Screaming Frog export tab names and bulk export names will need to be verified against the MCP server's accepted values during implementation.

## Output Formats

### Ad-hoc Query Response (Conversational)

For targeted questions, the skill responds conversationally:

- Present the data in a clear table or list
- Highlight the most important findings
- Group by pattern when useful (e.g., "42 of the 67 404s are under `/blog/2023/`")
- Provide actionable recommendations inline
- Flag related issues proactively (e.g., when showing 404s, mention if any have significant inlinks)

### Full Audit Report (Structured Markdown)

For comprehensive audits, generate a markdown report:

```markdown
# Technical SEO Audit — [domain] — [date]

## Summary
- Pages crawled: X
- Critical issues: X | High: X | Medium: X | Low: X
- Top priority: [one-line summary of the most impactful issue]

## Critical Issues

### [Issue Name]
- **Affected pages**: X ([X]% of crawled pages)
- **Impact**: [Why this matters for SEO — rankings, crawl budget, indexation]
- **Action**: [Specific steps to fix]
- **Sample URLs**:
  | URL | Details |
  |-----|---------|
  | /path/to/page | [relevant detail] |

## High Priority
### [Issue Name]
...

## Medium Priority
### [Issue Name]
...

## Low Priority
### [Issue Name]
...

## Next Steps
1. [Highest priority action]
2. [Second priority action]
3. [Third priority action]
...
```

### Severity Classification

| Severity | Criteria |
|----------|----------|
| **Critical** | Directly prevents indexation or causes major crawl issues. Examples: widespread 404s with inlinks, redirect loops, noindex on important pages |
| **High** | Significantly impacts rankings or user experience. Examples: missing titles on key pages, redirect chains, orphan pages |
| **Medium** | Affects SEO performance but not blocking. Examples: duplicate meta descriptions, suboptimal canonicals, low inlink counts |
| **Low** | Minor optimizations. Examples: title length, missing Open Graph tags, anchor text improvements |

## Integration

### Plugin Registration (plugin.json)

```json
{
  "name": "technical-seo",
  "description": "Technical SEO specialist skill for analyzing Screaming Frog crawl data",
  "version": "1.0.0",
  "skills": "./skills"
}
```

### Complementary Skills

If the `google-ranking-signals` plugin is installed, it provides complementary context on how Google evaluates and ranks pages. The technical-seo skill focuses on identifying issues from crawl data; google-ranking-signals provides the "why it matters" from Google's perspective. They are independent — neither requires the other.

## Success Criteria

1. User can say "show me 404 pages" and get a clear, actionable list from their crawl data
2. User can say "audit this crawl" and get a structured report covering all SEO domains
3. The skill correctly maps queries to the right Screaming Frog export tabs
4. Recommendations are specific and actionable, not generic SEO advice
5. The skill handles large crawls gracefully (pagination, filtering, summarization)

## Open Questions (to resolve during implementation)

- **Export tab names**: Exact Screaming Frog export tab/bulk export names need verification against the MCP server. Resolved by: testing each mapping with a real crawl during implementation.
- **Report file saving**: Audit reports are presented inline. Saving to file only when the user asks (e.g., "save this report"). No auto-save.
- **Crawl comparison**: Deferred to a future version. V1 analyzes a single crawl only.
