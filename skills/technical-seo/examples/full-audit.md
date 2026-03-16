# Example: Full Audit — "Audit this crawl"

This walkthrough shows how the technical-seo skill handles a comprehensive audit request. The user asks for a full crawl audit, and the skill exports all relevant data, analyzes every SEO domain, cross-references findings, and delivers a structured report.

---

## User prompt

> Audit this crawl

---

## Step 1: Preflight — Verify Screaming Frog

The skill calls `sf_check` before doing anything else.

**Tool call:**

```
sf_check()
```

**Result:**

```json
{
  "status": "ok",
  "version": "23.2",
  "license": "active",
  "gui_running": false
}
```

Screaming Frog is installed, licensed, and the GUI is closed. Proceed to Step 2.

---

## Step 2: Identify the Crawl

The skill calls `list_crawls` to find available crawl data.

**Tool call:**

```
list_crawls()
```

**Result:**

```json
{
  "crawls": [
    {
      "id": 1,
      "name": "example.com",
      "pages_crawled": 15234,
      "crawl_date": "2026-03-10"
    },
    {
      "id": 2,
      "name": "staging.example.com",
      "pages_crawled": 847,
      "crawl_date": "2026-03-08"
    }
  ]
}
```

Multiple crawls found. Per SKILL.md rules, the skill lists both and asks the user to pick one:

> Two crawls available:
>
> 1. **example.com** — 15,234 pages, crawled March 10
> 2. **staging.example.com** — 847 pages, crawled March 8
>
> Which crawl should I audit?

**User response:**

> 1

The skill proceeds with crawl #1 (example.com, 15,234 pages).

---

## Step 3: Understand the Ask

The user said "Audit this crawl." This is a **full audit** — not an ad-hoc query. The skill will:

1. Export data across all SEO domains
2. Read all reference files
3. Analyze each domain: crawlability, on-page, link structure, performance, structured data
4. Cross-reference findings across domains
5. Produce a structured report per `examples/report-template.md`

---

## Step 4: Export the Right Data

The skill reads `references/export-mappings.md` and looks up the **Full Audit** section. This determines the complete set of export parameters.

**Tool call:**

```
export_crawl(
  crawl_id: 1,
  export_tabs: [
    "Internal:All",
    "Response Codes:All",
    "Page Titles:All",
    "Meta Description:All",
    "H1:All",
    "Canonicals:All",
    "Directives:All",
    "Images:All"
  ],
  bulk_export: [
    "All Inlinks",
    "All Redirect Chains"
  ],
  save_report: [
    "Crawl Overview"
  ]
)
```

All 8 tabs, 2 bulk exports, and 1 report — exactly matching the Full Audit row in export-mappings.md. No more, no less.

**If any export fails:** Log it, skip it, continue with the rest. Include a note in the findings: "Note: [tab name] data was not available in this crawl — [reason if known]." Do not abort the entire audit over one failed export.

---

## Step 5: Read and Analyze

### 5a: Read all reference files

For a full audit, the skill reads every reference file before analyzing:

1. `references/export-mappings.md` (already done in Step 4)
2. `references/crawlability.md`
3. `references/on-page.md`
4. `references/link-structure.md`
5. `references/performance.md`
6. `references/structured-data.md`

These references contain the severity classifications, diagnostic criteria, and recommended fixes that the skill applies during analysis.

### 5b: Read exported data

Use `read_crawl_data` to load each exported CSV. For large exports, use pagination — never attempt to load a full export in a single call.

**Handling large result sets (>500 rows):**

This is a 15,234-page crawl. Several exports will exceed 500 rows. When they do:

1. **Summarize with counts and patterns** — do not list every URL
2. **Group by URL pattern** (e.g., "312 of the 404s are under `/products/discontinued/`")
3. **Show up to 10 sample URLs per issue**, ordered by relevance (e.g., highest inlink count for 404s, highest response time for slow pages)
4. **State the total** and offer to show more: "Showing 10 of 847 — ask me to filter or show more"

### 5c: Analyze each SEO domain

Work through each domain systematically, applying the criteria from the reference files:

**Crawlability** (from `crawlability.md`):
- HTTP status errors: 404s, 5xx, soft 404s
- Redirects: chains (3+ hops), loops, 302s used as permanent
- Orphan pages
- Crawl depth distribution
- Robots directives: blocked pages, noindex/nofollow conflicts
- Sitemap issues: non-200 URLs in sitemap, indexable pages missing from sitemap

**On-page** (from `on-page.md`):
- Page titles: missing, duplicate, length issues
- Meta descriptions: missing, duplicate, length issues
- H1s: missing, multiple, duplicate
- Canonicals: non-self-referencing, canonicals to non-200 pages, conflicts with noindex
- Hreflang: missing return tags, invalid codes, missing x-default

**Link structure** (from `link-structure.md`):
- Low inlink pages (< 3 unique inlinks on indexable pages)
- Link equity distribution across page types
- Broken internal links (4xx destinations with inlinks)
- Excessive outlinks (> 100 per page)
- Anchor text quality on high-value pages
- Link depth analysis

**Performance** (from `performance.md`):
- TTFB: pages > 1s, pages > 2s
- Page size: HTML documents > 3MB, > 5MB
- Large images: > 500KB, > 1MB
- Redirect latency on multi-hop chains

**Structured data** (from `structured-data.md`):
- Missing schema on key page types (products, articles, FAQs, breadcrumbs)
- Invalid schema (syntax errors, wrong types)
- Incomplete schema (missing required/recommended properties)
- Mismatched schema (type doesn't match page content)

**If the `Structured Data:All` export was not included or returned empty:** The crawl was likely not configured to extract structured data. Note this explicitly: "Structured data analysis is not available for this crawl — the crawl was not configured with structured data extraction enabled (Configuration > Spider > Extraction > Structured Data). Recommend enabling it for the next crawl." Do not guess or skip the section silently.

### 5d: Cross-reference findings

This is where the audit becomes more than a checklist. Cross-referencing surfaces issues that only become visible when data from different domains is combined:

| Finding A | + Finding B | = Elevated Severity |
|---|---|---|
| 404 page | Has inlinks from live pages | Critical — link equity flowing to dead ends |
| Orphan page | In the XML sitemap | High — contradictory signals (sitemap says index, no internal links say it's unimportant) |
| Noindex page | Has significant inlinks | High — equity flowing to a page that won't be indexed |
| Noindex page | In the XML sitemap | High — sitemap says index, tag says don't |
| Deep page (depth 5+) | Also has < 3 inlinks | Compounding discoverability problem |
| Slow page (TTFB > 2s) | Is a high-traffic category/product page | Critical — directly impacts user experience and rankings on a key page |
| Redirect chain | Ends in 4xx/5xx | Critical — all inlinks to the chain source effectively point to a dead end |
| Canonical points to non-200 | Target page is a 404 or redirect | High — canonical signal is broken |

### 5e: Prioritize by impact

Not all issues are equal. Prioritize using this logic:

- **One noindexed high-traffic page > 50 long titles.** A single noindex on a page that drives significant organic traffic is more damaging than 50 titles that are 5 characters over the recommended length.
- **404s with inlinks > 404s without inlinks.** A 404 with zero inlinks wastes crawl budget but loses no equity. A 404 with 30 inlinks is actively destroying link value.
- **Template-level issues > one-off issues.** A broken canonical in a page template affecting 2,000 product pages is far more impactful than the same issue on a single blog post.
- **Indexation blockers > optimization opportunities.** Anything that prevents a page from being indexed (noindex, robots block, redirect loop) outranks anything that merely makes an indexed page perform suboptimally (long title, missing meta description).

---

## Step 6: Present Report

The skill produces a structured markdown report following the format defined in `examples/report-template.md`. Key structural elements:

1. **Executive summary** — top-line health assessment with 3-5 bullet points
2. **Issues organized by severity** (Critical > High > Medium > Low), each with:
   - Affected page count and percentage of total crawled pages
   - SEO impact explanation
   - Specific, actionable fix
   - Up to 10 sample URLs in a table, grouped by pattern where applicable
3. **Structured data gap note** — if structured data was not available in the crawl, state this explicitly as a section so the user doesn't assume the site has no issues
4. **Next steps** — prioritized list of top 5-10 actions, ordered by impact-to-effort ratio

The report is presented inline in the conversation. It is not saved to a file unless the user asks.

---

## Key behaviors this workflow demonstrates

### Pagination discipline
A 15,234-page crawl produces large exports. The skill never dumps thousands of rows. For any result set exceeding 500 rows, it summarizes with counts, groups by URL pattern, and shows 10 sample URLs per issue. The user can always ask for more.

### Cross-referencing
The skill doesn't analyze each domain in isolation. It combines data across exports to surface compound issues — 404s with inlinks, orphan pages in the sitemap, noindexed pages receiving link equity. These cross-referenced findings are often the most actionable items in the audit.

### Prioritization by impact
Severity is not assigned by issue type alone. A 404 with 30 inlinks is Critical; a 404 with zero inlinks is Medium. A noindexed page that ranks for high-value keywords is Critical; a noindexed utility page is expected. The skill applies the severity escalation rules from the reference files and weights findings by actual impact on the site.

### Proactive findings
The user said "audit this crawl" — they didn't ask about any specific issue. The skill surfaces everything it finds, including issues the user may not have been thinking about. If the crawl reveals an unexpected robots.txt block on a key section, or a pattern of redirect chains from a past migration, the skill reports it without being asked.

### Structured data gap handling
If the crawl wasn't configured to extract structured data, the skill doesn't silently skip the section. It notes the gap explicitly and recommends enabling structured data extraction for the next crawl. The user should know what the audit could not assess, not just what it found.
