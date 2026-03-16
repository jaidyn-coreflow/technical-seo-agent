# Example: Ad-hoc Query — "Show me the 404 pages"

This walkthrough shows how the technical-seo skill handles a targeted, single-issue query from start to finish. The user asks a direct question and gets a direct, data-backed answer with prioritized actions.

---

## User prompt

> Show me the 404 pages

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

Screaming Frog v23.2 is installed, the license is active, and the GUI is closed (so the database is not locked). Proceed to Step 2.

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
    }
  ]
}
```

One crawl found. Auto-selected per SKILL.md rules — no need to prompt the user.

---

## Step 3: Understand the Ask

The user said "Show me the 404 pages." This is an **ad-hoc query**, not a full audit. The scope is narrow: HTTP 404 client errors. The skill will export only the data needed for this question, analyze it, and respond conversationally.

Per SKILL.md, the skill should also cross-reference inlinks to assess impact, and proactively flag related issues if they are clearly relevant.

---

## Step 4: Export the Right Data

The skill reads `references/export-mappings.md` to determine the correct export parameters.

**Mapping found:**

| Query Pattern | `export_tabs` |
|---|---|
| 404s, broken pages, client errors | `Response Codes:Client Error (4xx)` |

Since the crawlability reference says to cross-reference inlinks for 404 pages with inbound links, the skill also requests the `All Inlinks` bulk export.

**Tool call:**

```
export_crawl(
  crawl_id: 1,
  export_tabs: ["Response Codes:Client Error (4xx)"],
  bulk_export: ["All Inlinks"]
)
```

**Result:**

```json
{
  "exports": [
    {
      "tab": "Response Codes:Client Error (4xx)",
      "file": "/tmp/sf-exports/client-error-4xx.csv",
      "rows": 67
    },
    {
      "bulk": "All Inlinks",
      "file": "/tmp/sf-exports/all-inlinks.csv",
      "rows": 48210
    }
  ]
}
```

---

## Step 5: Read and Analyze

The skill reads `references/crawlability.md` for 404-specific guidance, then calls `read_crawl_data` to load the exported CSVs.

**Tool calls:**

```
read_crawl_data(file: "/tmp/sf-exports/client-error-4xx.csv")
read_crawl_data(file: "/tmp/sf-exports/all-inlinks.csv", filter_column: "Destination", filter_values: [<404 URLs>])
```

The skill reads the 67 4xx URLs, then filters the inlinks export to find which pages link to those 404 URLs. It groups by URL pattern, counts inlinks per group, and applies severity using the crawlability reference.

**Analysis summary (internal):**

- 67 total pages returning 404
- 42 URLs match `/blog/2023/*` — these have 156 total inlinks across the group
- 18 URLs match `/products/discontinued/*` — 23 total inlinks
- 7 remaining URLs scattered across various paths — 3 total inlinks
- No 404 URLs found in the XML sitemap

---

## Step 6: Present Findings

The skill responds conversationally per SKILL.md's ad-hoc response format — lead with the answer, group by pattern, provide specific actions.

---

### Skill response to the user

Found **67 pages returning 404** in the example.com crawl (March 10).

Three distinct patterns:

| URL Pattern | Count | Inlinks | Severity |
|---|---|---|---|
| `/blog/2023/*` | 42 | 156 | Critical |
| `/products/discontinued/*` | 18 | 23 | High |
| Other (misc paths) | 7 | 3 | Medium |

#### `/blog/2023/*` — Critical

42 deleted blog posts from 2023 are still receiving 156 internal links. This is significant link equity going to dead ends. The inlinks are coming primarily from sidebar "Related Posts" widgets and in-content cross-links from other blog posts.

**Action:** Implement 301 redirects for each of these 42 URLs to the most topically relevant current blog post. For posts with no clear match, redirect to the parent category page (e.g., `/blog/2023/seo-tips-for-ecommerce` to `/blog/category/ecommerce`). Then audit the "Related Posts" widget logic — it should exclude URLs that no longer resolve to 200.

Top 5 by inlink count:

| URL | Status | Inlinks |
|---|---|---|
| `/blog/2023/complete-guide-to-core-web-vitals` | 404 | 31 |
| `/blog/2023/ecommerce-seo-checklist` | 404 | 24 |
| `/blog/2023/internal-linking-strategy` | 404 | 19 |
| `/blog/2023/technical-seo-audit-template` | 404 | 17 |
| `/blog/2023/schema-markup-for-products` | 404 | 14 |

#### `/products/discontinued/*` — High

18 discontinued product pages with 23 inlinks. These are likely from category pages or cross-sell modules that still reference removed products.

**Action:** 301 redirect each discontinued product to its replacement product. Where no direct replacement exists, redirect to the parent category page (e.g., `/products/discontinued/widget-pro-v2` to `/products/widgets`). Update category page templates and cross-sell logic to filter out discontinued SKUs.

#### Other — Medium

7 scattered 404s across various paths with only 3 total inlinks. Low impact individually.

| URL | Status | Inlinks |
|---|---|---|
| `/about/team/jane-doe` | 404 | 2 |
| `/resources/pdf/2022-annual-report` | 404 | 1 |
| `/events/webinar-march-2024` | 404 | 0 |
| `/case-studies/acme-corp` | 404 | 0 |
| `/partners/old-partner-page` | 404 | 0 |
| `/careers/senior-seo-analyst` | 404 | 0 |
| `/news/product-launch-2023` | 404 | 0 |

**Action:** Redirect `/about/team/jane-doe` to `/about/team` (it has 2 inlinks). The 5 pages with zero inlinks are low priority — optionally return 410 (Gone) to signal intentional removal, or redirect where a relevant destination exists. No urgency.

---

**Summary:** The `/blog/2023/*` group is the priority. 156 inlinks pointing at dead pages means measurable link equity loss. Fix those redirects first, then update the widget that keeps linking to deleted posts so this does not recur.

Want me to show the full list of all 67 URLs, or dig into the inlink sources to see exactly which pages are linking to the broken URLs?

---

## Why this response works

1. **Leads with the answer.** The user asked to see 404 pages. The first line gives the count.
2. **Groups by pattern.** Instead of dumping 67 rows, the skill identifies three URL patterns and presents each with its own count, inlink total, and severity.
3. **Prioritizes with severity.** Critical/High/Medium assignments tell the user where to focus. The rationale is clear — link equity loss drives the severity, not just the page count.
4. **Gives specific actions.** Not "add redirects" but "redirect `/blog/2023/seo-tips-for-ecommerce` to `/blog/category/ecommerce`" and "audit the Related Posts widget logic."
5. **Shows sample URLs.** The top-5 table for the critical group is sorted by inlinks descending — the most damaging 404s surface first.
6. **Flags the root cause.** The sidebar widget generating links to deleted posts is the systemic issue. Fixing individual redirects without fixing the widget means new 404s will keep appearing.
7. **Offers to go deeper.** The closing question gives the user a clear next step without forcing more data on them.
