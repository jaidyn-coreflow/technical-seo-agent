# Technical SEO Skill Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a Claude Code plugin skill that acts as a technical SEO specialist, analyzing Screaming Frog crawl data via MCP and producing actionable findings.

**Architecture:** Single Claude Code plugin with one skill (technical-seo). Core SKILL.md defines persona and workflow. Reference files provide domain-specific SEO expertise. Example files demonstrate usage patterns. All files are markdown — no code.

**Tech Stack:** Claude Code plugin system (SKILL.md + references/ + examples/), Screaming Frog MCP server

**Spec:** `docs/superpowers/specs/2026-03-11-technical-seo-skill-design.md`

---

## File Map

| File | Responsibility |
|------|---------------|
| `.claude-plugin/plugin.json` | Plugin registration metadata |
| `skills/technical-seo/SKILL.md` | Core skill: persona, trigger, workflow, error handling, output formats |
| `skills/technical-seo/references/export-mappings.md` | Maps SEO queries to Screaming Frog export parameters |
| `skills/technical-seo/references/crawlability.md` | SEO knowledge: crawl errors, redirects, orphans, robots |
| `skills/technical-seo/references/on-page.md` | SEO knowledge: titles, metas, H1s, canonicals, hreflang |
| `skills/technical-seo/references/link-structure.md` | SEO knowledge: inlinks, outlinks, broken links, anchor text |
| `skills/technical-seo/references/performance.md` | SEO knowledge: page size, TTFB, images, render-blocking |
| `skills/technical-seo/references/structured-data.md` | SEO knowledge: schema markup issues |
| `skills/technical-seo/examples/ad-hoc-query.md` | Example: handling targeted SEO queries |
| `skills/technical-seo/examples/full-audit.md` | Example: comprehensive crawl audit workflow |
| `skills/technical-seo/examples/report-template.md` | Markdown template for audit reports |

---

## Chunk 1: Plugin Scaffolding and Core Skill

### Task 1: Create plugin scaffolding

**Files:**
- Create: `.claude-plugin/plugin.json`

- [ ] **Step 1: Create the plugin.json file**

```json
{
  "name": "technical-seo",
  "description": "Technical SEO specialist skill for analyzing Screaming Frog crawl data",
  "version": "1.0.0",
  "skills": "./skills"
}
```

- [ ] **Step 2: Create the directory structure**

```bash
mkdir -p skills/technical-seo/references
mkdir -p skills/technical-seo/examples
```

- [ ] **Step 3: Commit**

```bash
git add .claude-plugin/plugin.json
git commit -m "feat: add plugin scaffolding for technical-seo skill"
```

---

### Task 2: Write the core SKILL.md

This is the most important file — it defines who the skill is, when it triggers, and how it operates.

**Files:**
- Create: `skills/technical-seo/SKILL.md`

- [ ] **Step 1: Write SKILL.md with frontmatter**

The frontmatter must include a comprehensive `description` field for auto-invocation triggers.

```yaml
---
name: technical-seo
description: >
  Use when the user asks to audit a crawl, analyze SEO issues, check technical SEO,
  show 404 pages, find missing titles, analyze internal links, check redirect chains,
  find orphan pages, check crawl data, show broken links, analyze page titles,
  check meta descriptions, find thin content, check schema markup, review canonicals,
  analyze crawl depth, check hreflang, find duplicate content, review directives,
  or any Screaming Frog crawl analysis task.
user-invokable: true
args:
  - name: query
    description: "The SEO question or 'audit' for a full crawl audit (optional)"
    required: false
---
```

- [ ] **Step 2: Write the persona section**

Immediately after frontmatter. Defines who the skill acts as and how it communicates.

```markdown
# Technical SEO Specialist

You are a senior technical SEO specialist reporting to the user, who is your SEO manager. You have deep expertise across all technical SEO domains.

## How You Communicate

- **Direct**: The user knows SEO. Don't explain basics. Focus on findings, severity, and actions.
- **Data-driven**: Back every recommendation with specific data from the crawl.
- **Proactive**: When investigating one issue, flag related problems you notice.
- **Actionable**: Every finding comes with a specific fix, not generic advice.
```

- [ ] **Step 3: Write the prerequisites and preflight section**

```markdown
## Prerequisites

Before doing anything, verify the environment is ready:

1. Call `sf_check` to verify Screaming Frog is installed and licensed.
   - If SF is not found: tell the user to install Screaming Frog SEO Spider (v16+) and the screaming-frog-mcp server.
   - If the GUI is still open / database is locked: tell the user to close the Screaming Frog application (the database only allows single-process access).
   - If the MCP server is not responding: tell the user to install and configure the screaming-frog-mcp server in Claude Code.
   - On any preflight failure: **stop**. Do not attempt to continue.

2. Call `list_crawls` to find available crawls.
   - If no crawls are found: tell the user to run a crawl in the Screaming Frog GUI first, then close the app. **Stop.**
   - If one crawl exists: auto-select it and confirm to the user which crawl you're using.
   - If multiple crawls exist: present the list with Database IDs and let the user pick.
```

- [ ] **Step 4: Write the workflow section**

```markdown
## Workflow

After preflight passes and a crawl is selected:

### 1. Understand the Ask

Classify the user's request:

- **Ad-hoc query**: A specific question about the crawl data (e.g., "show me 404s", "which pages have low inlinks", "are there duplicate titles").
- **Full audit**: A broad request for comprehensive analysis (e.g., "audit this crawl", "run a full technical SEO check", "what are the issues with this site").

### 2. Export the Right Data

Read `references/export-mappings.md` to determine which `export_crawl` parameters to use.

- **Ad-hoc**: Export only the specific tab(s) or bulk exports needed for the query.
- **Full audit**: Export all tabs listed in the "Full Audit" row of export-mappings.md.

Call `export_crawl` with the crawl's Database ID and the determined parameters.

If an export fails (e.g., invalid tab name): log the error, skip that tab, continue with remaining exports. Note the skipped data in your output.

### 3. Read and Analyze

Use `read_crawl_data` to read the exported CSV files.

Read the relevant `references/*.md` file(s) for the SEO domain(s) being analyzed. These contain:
- What each issue is and why it matters
- Severity guidance (critical/high/medium/low)
- Recommended fixes

Apply this knowledge to interpret the crawl data. Identify issues, assess their severity, and determine root causes.

**Handling large data sets:**
- If a result set has >500 rows: summarize with counts and URL patterns, show top 20 sample URLs, offer to show more.
- Use `read_crawl_data` pagination (page/row parameters) to read in chunks. Never load an entire large export at once.
- If an exported CSV is not found (auto-deleted after 1 hour): re-export and retry.

### 4. Present Findings

**For ad-hoc queries:**
- Respond conversationally with data tables, interpretation, and recommended actions.
- Group by URL pattern when useful (e.g., "42 of the 67 404s are under `/blog/2023/`").
- Flag related issues proactively (e.g., when showing 404s, mention if any have significant inlinks).

**For full audits:**
- Generate a structured report following `examples/report-template.md`.
- Organize by severity: Critical > High > Medium > Low.
- For each issue: count, percentage of total pages, impact statement, specific action, and sample URLs (up to 10 per issue).
- End with a prioritized "Next Steps" section.
- Present the report inline. Only save to file if the user asks.
```

- [ ] **Step 5: Write the reference file loading guidance**

```markdown
## Reference Files

When analyzing crawl data, read the relevant reference file(s) from `references/` to inform your analysis:

| SEO Domain | Reference File | When to Read |
|-----------|---------------|-------------|
| Crawl errors, redirects, orphans, robots, sitemaps | `references/crawlability.md` | Queries about 404s, 5xx, redirects, orphans, crawl depth, robots.txt, sitemaps |
| Titles, metas, H1s, canonicals, hreflang | `references/on-page.md` | Queries about page titles, meta descriptions, headings, canonicals, hreflang |
| Internal links, inlinks, outlinks, anchors | `references/link-structure.md` | Queries about internal links, link equity, broken links, anchor text |
| Page size, response time, images | `references/performance.md` | Queries about page speed, large pages, slow response times |
| Schema markup | `references/structured-data.md` | Queries about schema, structured data, rich results |
| Query-to-export parameter mapping | `references/export-mappings.md` | **Always** — needed to determine what to export |

For a full audit, read all reference files.
```

- [ ] **Step 6: Commit**

```bash
git add skills/technical-seo/SKILL.md
git commit -m "feat: add core SKILL.md with persona, workflow, and reference guidance"
```

---

### Task 3: Write export-mappings.md

The critical reference that maps user queries to Screaming Frog MCP export parameters.

**Files:**
- Create: `skills/technical-seo/references/export-mappings.md`

- [ ] **Step 1: Write export-mappings.md**

```markdown
# Export Mappings

Maps SEO analysis queries to Screaming Frog MCP `export_crawl` parameters.

## How to Use

1. Match the user's query to a query pattern below.
2. Call `export_crawl` with the crawl's Database ID and the listed parameters.
3. If a query spans multiple domains, combine the exports from each relevant section.

## Crawlability

| Query Pattern | export_tabs | bulk_export | save_report |
|---------------|-------------|-------------|-------------|
| 404s, client errors, broken pages | Response Codes:Client Error (4xx) | — | — |
| Server errors, 5xx | Response Codes:Server Error (5xx) | — | — |
| Redirects, redirect chains, 301s, 302s | Response Codes:Redirection (3xx) | All Redirect Chains | — |
| Orphan pages | — | Orphan Pages | — |
| Crawl overview, summary | — | — | Crawl Overview |

## On-Page

| Query Pattern | export_tabs | bulk_export | save_report |
|---------------|-------------|-------------|-------------|
| Missing titles | Page Titles:Missing | — | — |
| Duplicate titles | Page Titles:Duplicate | — | — |
| Long titles, title length | Page Titles:Over 60 Characters | — | — |
| All title issues | Page Titles:All | — | — |
| Missing meta descriptions | Meta Description:Missing | — | — |
| Duplicate meta descriptions | Meta Description:Duplicate | — | — |
| All meta description issues | Meta Description:All | — | — |
| Missing H1 | H1:Missing | — | — |
| Multiple H1s | H1:Multiple | — | — |
| Duplicate H1s | H1:Duplicate | — | — |
| All H1 issues | H1:All | — | — |
| Canonicals | Canonicals:All | — | — |
| Directives, noindex, nofollow | Directives:All | — | — |
| Hreflang | Hreflang:All | — | — |
| Images, alt text, missing alt | Images:All | — | — |

## Link Structure

| Query Pattern | export_tabs | bulk_export | save_report |
|---------------|-------------|-------------|-------------|
| Internal links, inlinks, low inlinks | — | All Inlinks | — |
| Outlinks, external links | — | All Outlinks | — |
| Broken internal links | Response Codes:Client Error (4xx) | All Inlinks | — |

## Performance

| Query Pattern | export_tabs | bulk_export | save_report |
|---------------|-------------|-------------|-------------|
| Response time, slow pages, TTFB | Internal:All | — | — |
| Page size, large pages | Internal:All | — | — |

## Structured Data

| Query Pattern | export_tabs | bulk_export | save_report |
|---------------|-------------|-------------|-------------|
| Schema, structured data, rich results | Structured Data:All | — | — |

> **Note:** Structured data export availability depends on SF version and crawl configuration. If the export fails, report that structured data was not captured in this crawl and recommend enabling "Structured Data" extraction in Screaming Frog's configuration for future crawls.

## Full Audit

For a comprehensive audit, export all of these:

| export_tabs | bulk_export | save_report |
|-------------|-------------|-------------|
| Internal:All, Response Codes:All, Page Titles:All, Meta Description:All, H1:All, Canonicals:All, Directives:All, Images:All | All Inlinks, All Redirect Chains | Crawl Overview |

> **Note:** Export tab and bulk export names are based on Screaming Frog SEO Spider conventions. If an export fails with an unrecognized name, try the closest alternative or check SF's export documentation.
```

- [ ] **Step 2: Commit**

```bash
git add skills/technical-seo/references/export-mappings.md
git commit -m "feat: add export-mappings reference for SF MCP parameter mapping"
```

---

## Chunk 2: Knowledge Base Reference Files

### Task 4: Write crawlability.md

**Files:**
- Create: `skills/technical-seo/references/crawlability.md`

- [ ] **Step 1: Write crawlability.md**

```markdown
# Crawlability & Indexation

Technical SEO issues that affect how search engines discover, crawl, and index pages.

---

## HTTP Status Errors

### 404 (Not Found)
- **What**: Pages returning 404 status code — the URL exists in the site's link graph but the page doesn't exist.
- **Why it matters**: Wastes crawl budget. If other pages link to 404 URLs, the link equity is lost. Users hitting 404s have a poor experience and may leave the site.
- **Severity**: **Critical** if pages have inlinks (internal or external). **Medium** if no inlinks point to them.
- **How to identify**: Export `Response Codes:Client Error (4xx)`. Cross-reference with inlink data to prioritize.
- **Fix**: Redirect (301) to the most relevant existing page. If no relevant page exists, ensure the 404 returns a proper 404 status (not a soft 404) and remove internal links pointing to it.

### 5xx (Server Errors)
- **What**: Pages returning 500, 502, 503, or other server error codes.
- **Why it matters**: Search engines cannot index these pages. Persistent 5xx errors signal site reliability issues to Google.
- **Severity**: **Critical** — server errors prevent indexation entirely.
- **How to identify**: Export `Response Codes:Server Error (5xx)`.
- **Fix**: Investigate server-side cause (application errors, timeout, resource limits). These are typically development/infrastructure issues to escalate.

### Soft 404s
- **What**: Pages that return a 200 status code but display error content (e.g., "page not found" message on a 200 page).
- **Why it matters**: Google detects these and treats them as 404s anyway, but the mixed signal wastes crawl budget and can cause indexing confusion.
- **Severity**: **High**
- **How to identify**: Look for 200-status pages with very low word count or generic error-like titles.
- **Fix**: Return a proper 404 status code for pages that don't exist.

---

## Redirects

### Redirect Chains (3+ hops)
- **What**: A URL redirects to another URL, which redirects to another, forming a chain of 3 or more hops before reaching the final destination.
- **Why it matters**: Each hop adds latency and risks losing link equity. Google follows up to 10 redirects but strongly prefers direct redirects. Long chains may cause Googlebot to give up.
- **Severity**: **High**
- **How to identify**: Export `All Redirect Chains` bulk export. Look for chains with 3+ hops.
- **Fix**: Update each redirect in the chain to point directly to the final destination URL.

### Redirect Loops
- **What**: URL A redirects to URL B, which redirects back to URL A (or a cycle of any length).
- **Why it matters**: Creates an infinite loop — the page can never be reached or indexed.
- **Severity**: **Critical**
- **How to identify**: Export `Response Codes:Redirection (3xx)`. Look for URLs that appear as both source and destination.
- **Fix**: Break the loop by updating one of the redirects to point to the correct final destination.

### Temporary Redirects (302) Used as Permanent
- **What**: A 302 (temporary) redirect used where a 301 (permanent) would be appropriate — the old URL is never coming back.
- **Why it matters**: 302s tell search engines to keep the old URL indexed. If the move is permanent, link equity may not fully transfer.
- **Severity**: **Medium**
- **How to identify**: Export `Response Codes:Redirection (3xx)`. Filter for 302 status codes. Review whether each is truly temporary.
- **Fix**: Change 302 redirects to 301 where the move is permanent.

---

## Orphan Pages

- **What**: Pages with zero internal inlinks — no other page on the site links to them. They're only discoverable via sitemap, external links, or direct URL entry.
- **Why it matters**: Search engines primarily discover pages by following links. Orphan pages signal low importance and may not get crawled frequently or at all.
- **Severity**: **High** for important pages (product pages, key content). **Low** for intentionally unlisted pages (thank-you pages, campaign landing pages).
- **How to identify**: Export `Orphan Pages` bulk export, or cross-reference `All Inlinks` to find pages with zero internal inlinks.
- **Fix**: Add internal links from relevant, authoritative pages on the site. Ensure the page is in the XML sitemap as a secondary discovery path.

---

## Crawl Depth

- **What**: The number of clicks required to reach a page from the homepage.
- **Why it matters**: Pages deeper than 3 clicks from the homepage are crawled less frequently and perceived as less important by search engines.
- **Severity**: **Medium** for depths of 4-5. **High** for depths of 6+.
- **How to identify**: Export `Internal:All`. Check the "Crawl Depth" column.
- **Fix**: Improve site architecture and internal linking to bring important pages within 3 clicks of the homepage. Consider hub pages, breadcrumbs, or related content modules.

---

## Robots Directives

### Blocked by robots.txt
- **What**: Pages disallowed in robots.txt but linked to or expected to be indexed.
- **Why it matters**: Googlebot cannot crawl blocked pages, so they won't appear in search results. If other pages link to them, the link equity is wasted.
- **Severity**: **Critical** if the page should be indexed. **Low** if intentionally blocked (admin pages, staging).
- **How to identify**: Export `Directives:All`. Filter for "Blocked by Robots.txt".
- **Fix**: Remove the disallow rule if the page should be indexed. If intentionally blocked, remove internal links pointing to it.

### Noindex/Nofollow Conflicts
- **What**: A page has conflicting directives — e.g., noindex in meta robots but the page is in the sitemap, or noindex combined with a canonical pointing to a different page.
- **Why it matters**: Conflicting signals confuse search engines and may lead to unexpected indexing behavior.
- **Severity**: **High**
- **How to identify**: Export `Directives:All`. Look for pages with noindex that are also in the sitemap or have canonical tags pointing elsewhere.
- **Fix**: Align all signals: if a page should not be indexed, remove it from the sitemap and remove canonical tags. If it should be indexed, remove the noindex directive.

---

## Sitemap Issues

### Non-200 URLs in Sitemap
- **What**: URLs listed in the XML sitemap that return non-200 status codes (404, 301, 5xx).
- **Why it matters**: Sitemaps should only contain indexable, 200-status URLs. Non-200 URLs waste crawl budget and signal poor site maintenance to search engines.
- **Severity**: **High**
- **How to identify**: Cross-reference sitemap URLs with response code data.
- **Fix**: Remove non-200 URLs from the sitemap. For redirected URLs, update the sitemap to include the final destination URL.

### Indexable URLs Missing from Sitemap
- **What**: Pages that return 200, are not noindexed, and should be indexed — but aren't in the XML sitemap.
- **Why it matters**: While not strictly required, the sitemap helps search engines discover and prioritize pages. Missing important pages can slow their discovery and indexation.
- **Severity**: **Low** for well-linked pages. **Medium** for deep or poorly-linked pages.
- **How to identify**: Compare crawled indexable URLs against sitemap contents.
- **Fix**: Add missing indexable URLs to the sitemap.

---

## Pagination

### Missing or Incorrect rel=prev/next
- **What**: Paginated series (e.g., /blog/page/1, /blog/page/2) without proper rel=prev/next link elements.
- **Why it matters**: While Google has deprecated rel=prev/next as a ranking signal, it still helps other search engines understand paginated content structure.
- **Severity**: **Low**
- **Fix**: Implement rel=prev/next across paginated series, or consider alternative pagination strategies (load-more, infinite scroll with proper SEO handling).

### Noindexed Pagination Pages
- **What**: Pages in a paginated series with noindex directives.
- **Why it matters**: Can prevent content on paginated pages from being discovered and indexed. If the content only exists on paginated pages, it becomes invisible to search engines.
- **Severity**: **Medium** if unique content exists on paginated pages. **Low** if paginated pages only duplicate the first page's listings.
- **Fix**: Remove noindex if the paginated pages contain unique content. If using noindex intentionally, ensure all content is accessible via other paths.
```

- [ ] **Step 2: Commit**

```bash
git add skills/technical-seo/references/crawlability.md
git commit -m "feat: add crawlability reference — status errors, redirects, orphans, robots"
```

---

### Task 5: Write on-page.md

**Files:**
- Create: `skills/technical-seo/references/on-page.md`

- [ ] **Step 1: Write on-page.md**

```markdown
# On-Page Elements

Technical SEO issues related to HTML elements that search engines use to understand and display pages.

---

## Page Titles (<title>)

### Missing Title
- **What**: Page has no `<title>` element.
- **Why it matters**: The title is the most important on-page ranking signal and appears as the clickable headline in search results. Missing titles mean Google will generate one — often poorly.
- **Severity**: **Critical**
- **How to identify**: Export `Page Titles:Missing`.
- **Fix**: Add a unique, descriptive `<title>` tag to every indexable page. Include the primary keyword naturally.

### Duplicate Titles
- **What**: Multiple pages share the same `<title>` content.
- **Why it matters**: Duplicate titles make it harder for search engines to determine which page is most relevant for a query. Causes keyword cannibalization.
- **Severity**: **High**
- **How to identify**: Export `Page Titles:Duplicate`. Group by title text to see which pages share titles.
- **Fix**: Write unique titles for each page that reflect the page's specific content and target keyword.

### Title Too Long (>60 characters)
- **What**: Page title exceeds ~60 characters and will be truncated in search results.
- **Why it matters**: Truncated titles lose their call-to-action or key information. Users see "..." instead of the full title.
- **Severity**: **Low**
- **How to identify**: Export `Page Titles:Over 60 Characters`.
- **Fix**: Rewrite titles to fit within 50-60 characters. Front-load important keywords.

### Title Too Short (<30 characters)
- **What**: Page title is under 30 characters — likely too generic or missing descriptive detail.
- **Why it matters**: Short titles miss opportunities to include relevant keywords and compelling descriptions.
- **Severity**: **Low**
- **How to identify**: Export `Page Titles:All`. Filter for titles under 30 characters.
- **Fix**: Expand titles to be more descriptive while staying under 60 characters.

### Title Same as H1
- **What**: The `<title>` tag and `<h1>` contain identical text.
- **Why it matters**: While not inherently harmful, it's a missed opportunity. The title (shown in SERPs) and H1 (shown on page) serve different contexts and can target different variations of the keyword.
- **Severity**: **Low**
- **How to identify**: Export `Page Titles:All` and `H1:All`, compare.
- **Fix**: Differentiate the title and H1 slightly — the title can be more keyword-focused for search, the H1 more user-focused for the page.

---

## Meta Descriptions

### Missing Meta Description
- **What**: Page has no `<meta name="description">` tag.
- **Why it matters**: Google may generate a snippet from page content, which is often less compelling than a crafted description. Meta descriptions affect click-through rate (CTR), not rankings directly.
- **Severity**: **Medium**
- **How to identify**: Export `Meta Description:Missing`.
- **Fix**: Write a unique, compelling meta description (120-160 characters) that summarizes the page and includes a call to action.

### Duplicate Meta Descriptions
- **What**: Multiple pages share the same meta description.
- **Why it matters**: Signals to search engines that pages are similar or templated. Reduces the value of the description for differentiating pages in results.
- **Severity**: **Medium**
- **How to identify**: Export `Meta Description:Duplicate`.
- **Fix**: Write unique descriptions for each page. For large sites, prioritize high-traffic pages.

### Meta Description Too Long (>160 characters)
- **What**: Description exceeds ~160 characters and will be truncated.
- **Severity**: **Low**
- **Fix**: Trim to 120-160 characters. Put the most important information first.

### Meta Description Too Short (<70 characters)
- **What**: Description is under 70 characters — likely too brief to be useful.
- **Severity**: **Low**
- **Fix**: Expand to provide more context and a call to action.

---

## Headings (H1)

### Missing H1
- **What**: Page has no `<h1>` element.
- **Why it matters**: The H1 is the primary on-page heading and a strong signal for page topic. Missing H1 weakens topical relevance.
- **Severity**: **High**
- **How to identify**: Export `H1:Missing`.
- **Fix**: Add a single, descriptive H1 to every page that clearly states the page's topic.

### Multiple H1s
- **What**: Page has more than one `<h1>` element.
- **Why it matters**: While HTML5 technically allows multiple H1s in sectioning elements, best practice is a single H1 per page for clear topical signaling.
- **Severity**: **Medium**
- **How to identify**: Export `H1:Multiple`.
- **Fix**: Keep one H1 as the primary heading. Change others to H2 or appropriate heading level.

### Duplicate H1s Across Pages
- **What**: Multiple pages share the same H1 text.
- **Why it matters**: Similar to duplicate titles — signals to search engines that pages may be duplicative.
- **Severity**: **Medium**
- **How to identify**: Export `H1:Duplicate`.
- **Fix**: Write unique H1s that reflect each page's specific content.

---

## Canonicals

### Missing Canonical
- **What**: Page lacks a `<link rel="canonical">` tag.
- **Why it matters**: Without a canonical, search engines must guess which version of a page to index (especially with URL parameters, trailing slashes, http/https variants).
- **Severity**: **Medium**
- **How to identify**: Export `Canonicals:All`. Filter for missing canonicals.
- **Fix**: Add self-referencing canonical tags to all indexable pages.

### Canonical Pointing to Non-200 Page
- **What**: The canonical URL returns a non-200 status code (404, 301, 5xx).
- **Why it matters**: Tells search engines to index a page that doesn't exist or redirects. Causes indexing confusion.
- **Severity**: **Critical**
- **How to identify**: Export `Canonicals:All`. Cross-reference canonical URLs with response codes.
- **Fix**: Update the canonical to point to the correct, live (200-status) URL.

### Conflicting Canonical and Noindex
- **What**: A page has both a noindex directive AND a canonical pointing to a different URL.
- **Why it matters**: Noindex says "don't index this page" while the canonical says "the canonical version is over here." Google has to choose which to follow — the result is unpredictable.
- **Severity**: **High**
- **How to identify**: Export `Canonicals:All` and `Directives:All`. Cross-reference pages with noindex that also have non-self-referencing canonicals.
- **Fix**: Choose one signal. If the page shouldn't be indexed, use noindex and remove the canonical (or set it to self-referencing). If traffic should consolidate to another page, use the canonical and remove noindex.

---

## Hreflang

### Missing Return Tags
- **What**: Page A has an hreflang pointing to Page B, but Page B doesn't have a return hreflang pointing back to Page A.
- **Why it matters**: Hreflang requires bidirectional confirmation. Without return tags, search engines may ignore the hreflang entirely.
- **Severity**: **High**
- **How to identify**: Export `Hreflang:All`. Look for unreciprocated hreflang entries.
- **Fix**: Add return hreflang tags on every page referenced by another page's hreflang.

### Missing x-default
- **What**: Hreflang implementation lacks an `x-default` tag for the fallback/default language version.
- **Why it matters**: x-default tells search engines which page to show when no language-specific version matches the user's locale.
- **Severity**: **Medium**
- **How to identify**: Export `Hreflang:All`. Check for presence of x-default.
- **Fix**: Add `hreflang="x-default"` pointing to the most appropriate default page (usually English or the primary language).

### Invalid Language Codes
- **What**: Hreflang tags use incorrect ISO 639-1 language codes or ISO 3166-1 Alpha-2 country codes.
- **Why it matters**: Invalid codes are ignored by search engines.
- **Severity**: **High**
- **How to identify**: Export `Hreflang:All`. Validate language/country codes.
- **Fix**: Use valid ISO codes (e.g., `en-US`, `fr-FR`, `de`).

---

## Content

### Thin Content
- **What**: Pages with very low word count (typically under 200 words of body content).
- **Why it matters**: Thin pages provide little value to users and may be classified as low-quality by Google's helpful content signals.
- **Severity**: **Medium** for content pages (articles, product pages). **Low** for functional pages (contact, login).
- **How to identify**: Export `Internal:All`. Filter by word count.
- **Fix**: Expand content to provide comprehensive, useful information. If the page serves no purpose, consider consolidating it with another page or removing it.

---

## Open Graph

### Missing OG Tags
- **What**: Page lacks Open Graph tags (og:title, og:description, og:image).
- **Why it matters**: When pages are shared on social media, OG tags control how they appear. Without them, platforms auto-generate previews that are often unappealing.
- **Severity**: **Low** — does not affect search rankings, but affects social sharing and click-through.
- **How to identify**: Export `Internal:All`. Check for OG tag presence.
- **Fix**: Add og:title, og:description, and og:image to all pages, especially content pages likely to be shared.
```

- [ ] **Step 2: Commit**

```bash
git add skills/technical-seo/references/on-page.md
git commit -m "feat: add on-page reference — titles, metas, H1s, canonicals, hreflang"
```

---

### Task 6: Write link-structure.md

**Files:**
- Create: `skills/technical-seo/references/link-structure.md`

- [ ] **Step 1: Write link-structure.md**

```markdown
# Link Structure

Technical SEO issues related to internal linking — how pages connect to each other within the site.

---

## Low Inlink Pages

- **What**: Pages with fewer than 3 internal links pointing to them.
- **Why it matters**: Internal links are the primary way search engines discover pages and assess their relative importance. Pages with few inlinks receive less crawl attention and are perceived as less important.
- **Severity**: **High** for important content/product pages. **Low** for utility pages.
- **How to identify**: Export `All Inlinks` bulk export. Aggregate by target URL. Filter for pages with fewer than 3 unique inlinks.
- **Fix**: Add internal links from relevant, high-authority pages. Good opportunities include:
  - Related content sections
  - Contextual links within body copy
  - Navigation/footer links for key pages
  - Breadcrumbs
  - Hub/category pages

---

## Link Equity Distribution

- **What**: Whether the site's most important pages receive the most internal links, and whether link equity is spread effectively rather than concentrated or wasted.
- **Why it matters**: Internal link structure directly influences which pages rank. If important pages have few links while unimportant pages have many, the site's ranking potential is misallocated.
- **Severity**: **Medium** — this is an optimization opportunity, not a blocking issue.
- **How to identify**: Export `All Inlinks`. Compare inlink counts against page importance (defined by the user — typically high-value pages like product categories, service pages, cornerstone content).
- **Fix**: Restructure internal linking to prioritize important pages. Consider:
  - Which pages generate revenue or leads? Do they have strong internal links?
  - Are navigation links pointing to low-value pages that dilute equity?
  - Can hub/category pages be restructured to better distribute links?

---

## Broken Internal Links

- **What**: Internal links that point to pages returning 404, 410, or 5xx status codes.
- **Why it matters**: Broken links waste link equity, harm user experience, and signal poor site maintenance. Every broken link is a dead end for both users and crawlers.
- **Severity**: **High**
- **How to identify**: Export `Response Codes:Client Error (4xx)` and `All Inlinks`. Cross-reference to find internal links pointing to error pages. Focus on: which pages are linking to broken URLs and how many inlinks the broken pages receive.
- **Fix**: Either redirect the broken URL to the correct page (301), or update the source pages to link to the correct URL directly. Prefer updating the link over adding a redirect when possible.

---

## Excessive Outlinks

- **What**: Pages with more than 100 outgoing internal links.
- **Why it matters**: While Google can follow many links per page, excessive outlinks dilute the link equity passed to each linked page. Navigation-heavy pages with hundreds of links pass very little equity per link.
- **Severity**: **Low** for navigation/sitemap pages (expected). **Medium** for content pages (suggests poor link architecture).
- **How to identify**: Export `Internal:All`. Check outlink counts. Also export `All Outlinks` for detailed link-level data.
- **Fix**: For content pages: reduce unnecessary links, consolidate navigation, use nofollow on low-priority links (e.g., login, terms). For navigation: consider a more focused navigation structure.

---

## Anchor Text

- **What**: The clickable text of internal links.
- **Why it matters**: Anchor text is a strong relevance signal. Descriptive, keyword-relevant anchors help search engines understand what the linked page is about. Generic anchors ("click here", "read more", "learn more") waste this signal.
- **Severity**: **Low** — optimization opportunity.
- **How to identify**: Export `All Inlinks`. Review anchor text distribution for key pages. Look for:
  - Generic anchors ("click here", "learn more", "read more")
  - URL-as-anchor (using the raw URL as the link text)
  - Overly repetitive anchors (exact same text from every link)
- **Fix**: Use descriptive, varied anchor text that naturally includes relevant keywords. Different pages linking to the same target should use slightly different anchor text.

---

## Link Depth

- **What**: The minimum number of clicks required to reach a page from the homepage.
- **Why it matters**: Pages closer to the homepage are crawled more frequently and perceived as more important. Key pages should be within 3 clicks.
- **Severity**: **Medium** for depth 4-5. **High** for depth 6+.
- **How to identify**: Export `Internal:All`. Check the "Crawl Depth" column. Sort by depth to find the deepest pages.
- **Fix**: Reduce depth by:
  - Adding links from higher-level pages (homepage, category pages)
  - Improving site architecture (flatten deep hierarchies)
  - Adding breadcrumbs to expose the hierarchy
  - Using related content modules to cross-link
```

- [ ] **Step 2: Commit**

```bash
git add skills/technical-seo/references/link-structure.md
git commit -m "feat: add link-structure reference — inlinks, broken links, equity, anchors"
```

---

### Task 7: Write performance.md

**Files:**
- Create: `skills/technical-seo/references/performance.md`

- [ ] **Step 1: Write performance.md**

```markdown
# Performance

Page performance issues identifiable from Screaming Frog crawl data. Note: SF provides server-side metrics (response time, page size). For full Core Web Vitals (LCP, CLS, FID/INP), use Google Search Console or PageSpeed Insights.

---

## Response Time (TTFB)

- **What**: Time to First Byte — how long the server takes to respond to a request. Measured by Screaming Frog during the crawl.
- **Why it matters**: TTFB is a component of page speed, which is a Google ranking signal. Slow TTFB affects all pages on a site and indicates server-side performance issues.
- **Severity**: **High** for TTFB >2 seconds. **Medium** for TTFB 1-2 seconds. Acceptable: <500ms.
- **How to identify**: Export `Internal:All`. Sort by "Response Time" column (in milliseconds). Look for patterns — are slow pages on a specific subdirectory? Specific server?
- **Fix**: Investigate server-side causes:
  - Database query optimization
  - Server-side caching (page cache, object cache)
  - CDN implementation
  - Server resource limits (CPU, memory)
  - Hosting infrastructure upgrades
  - Reduce heavy server-side processing

---

## Page Size

- **What**: Total download size of the HTML document (and potentially associated resources depending on SF crawl config).
- **Why it matters**: Large pages take longer to download and render, especially on mobile connections. Excessive page size can slow crawling.
- **Severity**: **High** for pages >5MB. **Medium** for pages 3-5MB. Acceptable: <3MB.
- **How to identify**: Export `Internal:All`. Sort by "Size" column.
- **Fix**: Common causes and fixes:
  - Inline CSS/JS that should be external and cached
  - Excessive DOM size (too many HTML elements)
  - Unoptimized inline SVGs or Base64-encoded images
  - Large HTML tables or repeated markup

---

## Large Images

- **What**: Image files that are excessively large (uncompressed, wrong format, oversized dimensions).
- **Why it matters**: Images are typically the largest page resources. Large images directly impact page load time and Largest Contentful Paint (LCP).
- **Severity**: **Medium** for images >500KB. **High** for images >1MB.
- **How to identify**: Export `Images:All`. Sort by "Size" column. Look for images >200KB.
- **Fix**:
  - Convert to modern formats (WebP, AVIF)
  - Compress images (lossy or lossless)
  - Resize to actual display dimensions (don't serve a 4000px image in a 800px container)
  - Implement responsive images with `srcset`
  - Lazy-load below-the-fold images

---

## Redirect Latency

- **What**: Multiple redirects in sequence adding cumulative delay before the final page loads.
- **Why it matters**: Each redirect adds a full round-trip to the server. A chain of 3 redirects can add 1-3 seconds of latency.
- **Severity**: **Medium** for 2-hop chains. **High** for 3+ hop chains.
- **How to identify**: Export `All Redirect Chains`. Look at total chain length and cumulative response time.
- **Fix**: Point redirects directly to the final destination URL. See also: `crawlability.md` > Redirect Chains.
```

- [ ] **Step 2: Commit**

```bash
git add skills/technical-seo/references/performance.md
git commit -m "feat: add performance reference — TTFB, page size, images, redirect latency"
```

---

### Task 8: Write structured-data.md

**Files:**
- Create: `skills/technical-seo/references/structured-data.md`

- [ ] **Step 1: Write structured-data.md**

```markdown
# Structured Data (Schema Markup)

Issues with JSON-LD/Microdata schema markup that affects rich results eligibility in search.

> **Data availability note:** Structured data analysis requires Screaming Frog's "Structured Data" extraction to be enabled in the crawl configuration. If this data is not available in the crawl export, note this gap and recommend enabling it for future crawls (Configuration > Spider > Extraction > Structured Data).

---

## Missing Schema on Key Page Types

- **What**: Pages that should have schema markup based on their content type, but don't.
- **Why it matters**: Schema markup enables rich results (stars, prices, FAQ dropdowns, breadcrumbs, etc.) which significantly increase CTR in search results.
- **Severity**: **Medium** — pages will still be indexed and ranked, but miss rich result opportunities.
- **How to identify**: Export `Structured Data:All`. Cross-reference with page URLs to find pages of specific types (products, articles, FAQs, recipes, events, local business) that lack schema.
- **Recommended schema by page type**:
  | Page Type | Recommended Schema |
  |-----------|-------------------|
  | Product pages | Product, Offer, AggregateRating |
  | Blog/articles | Article or BlogPosting |
  | FAQ pages | FAQPage |
  | How-to/tutorials | HowTo |
  | Events | Event |
  | Local business | LocalBusiness |
  | Recipes | Recipe |
  | Reviews | Review |
  | Breadcrumbs | BreadcrumbList |
  | Organization | Organization (homepage) |
- **Fix**: Add JSON-LD schema markup to pages. JSON-LD in a `<script type="application/ld+json">` block is the recommended format.

---

## Invalid Schema

- **What**: Schema markup that contains syntax errors, invalid types, or properties that don't exist in the schema.org vocabulary.
- **Why it matters**: Invalid schema is ignored by search engines. The site misses rich result eligibility despite having made the effort to add markup.
- **Severity**: **High** — effort invested with no return.
- **How to identify**: Export `Structured Data:All`. Look for validation errors. For detailed validation, recommend using Google's Rich Results Test on specific pages.
- **Fix**: Correct syntax errors, use valid schema.org types and properties. Test with Google's Rich Results Test before deploying.

---

## Incomplete Schema

- **What**: Schema markup that uses the correct type but is missing required or recommended properties.
- **Why it matters**: Missing required properties can disqualify the page from rich results. Missing recommended properties reduces the richness of the result.
- **Severity**: **Medium** for missing recommended properties. **High** for missing required properties.
- **Required properties by common type**:
  | Schema Type | Required Properties |
  |------------|-------------------|
  | Product | name, image, offers (with price, priceCurrency) |
  | Article | headline, image, author, datePublished |
  | FAQPage | mainEntity (with Question + acceptedAnswer) |
  | Event | name, startDate, location |
  | LocalBusiness | name, address |
  | BreadcrumbList | itemListElement (with position, name, item) |
- **Fix**: Add missing required properties. Then add recommended properties to maximize rich result display.

---

## Mismatched Schema

- **What**: Schema type doesn't match the page's actual content (e.g., Article schema on a product page, Product schema on a blog post).
- **Why it matters**: Mismatched schema sends wrong signals about page content, may trigger manual actions from Google, and won't produce relevant rich results.
- **Severity**: **High**
- **How to identify**: Export `Structured Data:All`. Review schema types against page URLs/content.
- **Fix**: Match the schema type to the page's actual content purpose. Remove mismatched schema and replace with the appropriate type.
```

- [ ] **Step 2: Commit**

```bash
git add skills/technical-seo/references/structured-data.md
git commit -m "feat: add structured-data reference — schema issues, required properties"
```

---

## Chunk 3: Examples and Finalization

### Task 9: Write ad-hoc-query.md example

**Files:**
- Create: `skills/technical-seo/examples/ad-hoc-query.md`

- [ ] **Step 1: Write ad-hoc-query.md**

```markdown
# Example: Ad-hoc Query

Demonstrates how the skill handles a targeted SEO question.

---

## Scenario: "Show me the 404 pages"

### Step 1: Preflight
Call `sf_check` — Screaming Frog v23.2, licensed, GUI closed. OK.

### Step 2: Identify Crawl
Call `list_crawls` — one crawl found: "example.com" (Database ID: 1, 15,234 pages, crawled 2026-03-10). Auto-selected.

### Step 3: Understand the Ask
Ad-hoc query — user wants to see 404 pages specifically.

### Step 4: Export
Consult `references/export-mappings.md`:
- "404s" → `export_tabs: ["Response Codes:Client Error (4xx)"]`

Call `export_crawl` with Database ID 1, export_tabs: ["Response Codes:Client Error (4xx)"].

### Step 5: Read and Analyze
Call `read_crawl_data` on the exported CSV.

Read `references/crawlability.md` for 404-specific guidance.

### Step 6: Present Findings

Example response:

---

**Found 67 pages returning 404 across example.com.**

The majority cluster under two URL patterns:

| Pattern | Count | Inlinks |
|---------|-------|---------|
| `/blog/2023/*` | 42 | 156 total inlinks |
| `/products/discontinued/*` | 18 | 23 total inlinks |
| Other | 7 | 3 total inlinks |

**Priority action — `/blog/2023/` URLs (Critical):**
These 42 URLs have 156 internal links pointing to them. That's significant link equity being wasted. Recommend:
1. Set up 301 redirects from each old blog URL to the most relevant current blog post.
2. If no relevant post exists, redirect to the parent `/blog/` category page.
3. After redirects are in place, update internal links where possible to point directly to the new URLs.

**Secondary — `/products/discontinued/` URLs (High):**
18 discontinued product pages with 23 inlinks. Recommend:
1. Redirect to the current version of the product, or the parent category if the product no longer exists.

**Low priority — remaining 7 URLs (Medium):**
Only 3 inlinks total. Fix when convenient, but these are not impacting SEO significantly.

Want me to export the full list of 404 URLs with their inlink counts?

---
```

- [ ] **Step 2: Commit**

```bash
git add skills/technical-seo/examples/ad-hoc-query.md
git commit -m "feat: add ad-hoc query example — 404 analysis walkthrough"
```

---

### Task 10: Write full-audit.md example

**Files:**
- Create: `skills/technical-seo/examples/full-audit.md`

- [ ] **Step 1: Write full-audit.md**

```markdown
# Example: Full Audit

Demonstrates how the skill handles a comprehensive crawl audit request.

---

## Scenario: "Audit this crawl"

### Step 1: Preflight
Call `sf_check` — Screaming Frog v23.2, licensed, GUI closed. OK.

### Step 2: Identify Crawl
Call `list_crawls` — two crawls found:
1. "example.com" (Database ID: 1, 15,234 pages, 2026-03-10)
2. "staging.example.com" (Database ID: 2, 847 pages, 2026-03-08)

Present options. User selects #1.

### Step 3: Understand the Ask
Full audit — comprehensive analysis across all SEO domains.

### Step 4: Export
Consult `references/export-mappings.md` → "Full Audit" row.

Call `export_crawl` with:
- export_tabs: ["Internal:All", "Response Codes:All", "Page Titles:All", "Meta Description:All", "H1:All", "Canonicals:All", "Directives:All", "Images:All"]
- bulk_export: ["All Inlinks", "All Redirect Chains"]
- save_report: ["Crawl Overview"]

### Step 5: Read and Analyze
Read each exported CSV via `read_crawl_data`. Read ALL reference files for comprehensive analysis.

Analyze each domain:
1. Crawlability — check status codes, redirects, orphans, depth, robots
2. On-page — check titles, metas, H1s, canonicals, hreflang
3. Link structure — check inlink distribution, broken links, depth
4. Performance — check response times, page sizes
5. Structured data — check schema if available

### Step 6: Present Report
Generate a structured report following `examples/report-template.md`.

Example output follows the template format with real findings organized by severity. See `examples/report-template.md` for the exact structure.

### Key Behaviors During Full Audit

- **Pagination**: For categories with >500 results, summarize with counts and patterns. Show up to 10 sample URLs per issue.
- **Cross-referencing**: Connect findings across domains (e.g., 404 pages WITH inlinks are more critical than 404 pages without).
- **Prioritization**: Weight issues by impact. A single noindexed high-traffic page is more critical than 50 pages with long titles.
- **Proactive findings**: If the data reveals issues the user didn't ask about, surface them. That's the point of a full audit.
- **Structured data gap**: If structured data export isn't available, note it in the report as a recommendation for future crawls.
```

- [ ] **Step 2: Commit**

```bash
git add skills/technical-seo/examples/full-audit.md
git commit -m "feat: add full-audit example — comprehensive audit walkthrough"
```

---

### Task 11: Write report-template.md

**Files:**
- Create: `skills/technical-seo/examples/report-template.md`

- [ ] **Step 1: Write report-template.md**

```markdown
# Audit Report Template

Use this template when generating a full technical SEO audit report.

---

## Template

```
# Technical SEO Audit — [domain] — [date]

## Summary

| Metric | Value |
|--------|-------|
| Pages crawled | [total] |
| Critical issues | [count] |
| High priority issues | [count] |
| Medium priority issues | [count] |
| Low priority issues | [count] |
| Top priority | [one-line summary of the single most impactful issue to fix first] |

---

## Critical Issues

### [Issue Name]
- **Category**: [Crawlability / On-Page / Link Structure / Performance / Structured Data]
- **Affected pages**: [count] ([percentage]% of crawled pages)
- **Impact**: [Why this matters — effect on rankings, crawl budget, indexation, or user experience]
- **Action**: [Specific steps to fix, in priority order]
- **Sample URLs**:
  | URL | Details |
  |-----|---------|
  | /example/path-1 | [relevant detail — e.g., "87 inlinks", "TTFB 4.2s"] |
  | /example/path-2 | [relevant detail] |
  | ... | [up to 10 samples] |

[Repeat for each critical issue]

---

## High Priority

### [Issue Name]
[Same structure as Critical]

---

## Medium Priority

### [Issue Name]
[Same structure as Critical]

---

## Low Priority

### [Issue Name]
[Same structure as Critical, but can be more concise — fewer samples]

---

## Next Steps

Prioritized action plan:

1. **[Action]** — [brief why, e.g., "recovers link equity from 156 internal links pointing to 404s"]
2. **[Action]** — [brief why]
3. **[Action]** — [brief why]
4. **[Action]** — [brief why]
5. **[Action]** — [brief why]
```

---

## Severity Classification Guide

| Severity | Criteria | Examples |
|----------|----------|---------|
| **Critical** | Directly prevents indexation or causes major crawl issues | Widespread 404s with inlinks, redirect loops, noindex on important pages, canonical to non-200 |
| **High** | Significantly impacts rankings or user experience | Missing titles on key pages, redirect chains 3+ hops, orphan pages, broken internal links |
| **Medium** | Affects SEO performance but not blocking | Duplicate meta descriptions, suboptimal canonicals, low inlink counts, multiple H1s |
| **Low** | Minor optimizations and best practices | Title length, missing OG tags, anchor text improvements, missing x-default hreflang |

## Report Guidelines

- Always include a Summary section with issue counts by severity.
- Group issues by severity, not by SEO domain. A critical link issue and a critical crawlability issue both go under "Critical Issues."
- Show affected page count AND percentage — percentages give context (2 pages out of 50 is different from 2 out of 50,000).
- Sample URLs: up to 10 per issue. If there are patterns (e.g., all under `/blog/`), mention the pattern.
- Next Steps: maximum 5-7 actions, ordered by impact. Be specific — not "fix redirects" but "update 42 redirect chains under /blog/ to point directly to final destinations."
```

- [ ] **Step 2: Commit**

```bash
git add skills/technical-seo/examples/report-template.md
git commit -m "feat: add audit report template with severity classification"
```

---

### Task 12: Install and register the plugin

**Files:**
- Modify: Claude Code settings to register the plugin

- [ ] **Step 1: Verify all files exist**

```bash
ls -R skills/technical-seo/
```

Expected output:
```
skills/technical-seo/:
SKILL.md    examples/   references/

skills/technical-seo/examples:
ad-hoc-query.md    full-audit.md    report-template.md

skills/technical-seo/references:
crawlability.md      export-mappings.md   link-structure.md
on-page.md           performance.md       structured-data.md
```

- [ ] **Step 2: Register the plugin in Claude Code**

Register the plugin using the Claude Code CLI:

```bash
claude plugin add /path/to/technical-seo-agent
```

Or manually add to `~/.claude/settings.json` under `enabledPlugins`:

```json
{
  "enabledPlugins": {
    "technical-seo@technical-seo": true
  }
}
```

- [ ] **Step 3: Verify skill is discoverable**

Start a new Claude Code session and check that `/technical-seo` appears as an available skill. Try invoking it with:

```
/technical-seo show me the 404 pages
```

If the Screaming Frog MCP is configured and a crawl exists, the skill should execute its workflow.

- [ ] **Step 4: Final commit**

```bash
git add -A
git commit -m "feat: complete technical-seo skill plugin — ready for testing"
```
