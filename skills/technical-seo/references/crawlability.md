# Crawlability Reference

Use this reference when analyzing crawl data related to HTTP status errors, redirects, orphan pages, crawl depth, robots directives, sitemap issues, or pagination. Each issue includes what it is, why it matters, severity classification, how to identify it in Screaming Frog crawl data, and the recommended fix.

---

## 1. HTTP Status Errors

### 1.1 404 Not Found

**What it is:** The server returns a 404 status code, indicating the requested URL does not exist. This includes both true 404s (page was never created or was deleted) and URLs that once existed but were removed without a redirect.

**Why it matters:** Every 404 is a dead end for both users and search engine crawlers. When a 404 page has inbound links (internal or external), the link equity flowing through those links is lost entirely. At scale, 404s waste crawl budget — Googlebot spends time requesting pages that return nothing useful, reducing the crawl capacity available for pages that matter. If a previously ranking page starts returning 404, its rankings are lost within days.

**Severity:**

| Condition | Severity | Rationale |
|-----------|----------|-----------|
| 404 page has 1+ internal or external inlinks | **Critical** | Link equity is being lost; users are hitting dead ends from your own navigation or external sites |
| 404 page appears in XML sitemap | **Critical** | You are explicitly telling search engines to crawl a page that does not exist |
| 404 page with zero inlinks, not in sitemap | **Medium** | No immediate ranking or UX damage, but contributes to crawl budget waste if Googlebot discovers it via other means |

**How to identify in crawl data:**

- **Export tab:** `Response Codes:Client Error (4xx)`
- **Key columns:** `Address`, `Status Code`, `Inlinks` (count), `Indexability`
- **Cross-reference:** Export `All Inlinks` and filter by destination URLs matching the 404 list to find which pages are linking to broken URLs. Check the XML sitemap export for 404 URLs that are erroneously included.

**What to look for:**
- Sort 404s by inlink count descending — the highest-inlink 404s are the most damaging
- Group by URL path pattern to find systemic issues (e.g., an entire `/products/` subdirectory returning 404 after a migration)
- Check if 404 URLs were previously indexed by looking for them in sitemap data or checking Google Search Console

**Recommended fix:**

1. **404s with inlinks:** Implement 301 redirects to the most relevant existing page. If no relevant page exists, redirect to the nearest parent category. Never redirect all 404s to the homepage — this creates soft 404 signals.
2. **404s in the sitemap:** Remove them from the XML sitemap immediately. Regenerate the sitemap to exclude non-200 URLs.
3. **404s with zero inlinks:** Low priority. Optionally return a proper 410 (Gone) status to tell search engines the removal is intentional and permanent, which accelerates de-indexation.
4. **Systemic 404s (pattern-based):** Investigate the root cause — was a section deleted, a URL structure changed, or a CMS misconfigured? Fix at the source, then implement redirect rules using regex patterns rather than individual redirect entries.

---

### 1.2 5xx Server Errors

**What it is:** The server returns a 500-range status code (500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, etc.), indicating the server failed to fulfill the request. These are server-side failures, not content issues.

**Why it matters:** 5xx errors are the most severe crawlability problem possible. The page cannot be rendered, indexed, or served to users. If Googlebot encounters persistent 5xx errors, it will reduce crawl rate for the entire domain, amplifying the damage beyond just the affected URLs. Pages returning 5xx will be dropped from the index if the errors persist beyond a few days.

**Severity:** **Critical** — always, regardless of inlink count or page importance.

**How to identify in crawl data:**

- **Export tab:** `Response Codes:Server Error (5xx)`
- **Key columns:** `Address`, `Status Code`, `Status`, `Inlinks`
- Look at the specific 5xx code for diagnosis:

| Status Code | Meaning | Common Cause |
|-------------|---------|--------------|
| 500 | Internal Server Error | Application crash, unhandled exception, broken server-side code |
| 502 | Bad Gateway | Upstream server (app server, API) is down or unresponsive |
| 503 | Service Unavailable | Server overloaded, maintenance mode, rate limiting |
| 504 | Gateway Timeout | Upstream server took too long to respond |

**What to look for:**
- Any 5xx errors at all — even one is worth investigating
- Pattern analysis: are 5xx errors concentrated on a specific URL pattern (e.g., `/api/`, a specific template, URLs with certain query parameters)?
- Timing: if the crawl was run during a deployment or maintenance window, 5xx errors may be transient. Recommend re-crawling to confirm.

**Recommended fix:**

1. **Escalate immediately** to the engineering/infrastructure team with the full list of affected URLs and the specific status codes.
2. **Group by pattern** to help engineering diagnose: "All URLs matching `/products/*` return 502" is more actionable than a flat URL list.
3. **If 503 with Retry-After header:** The server is intentionally rate-limiting. This may indicate the crawl rate was too aggressive, or the server is in maintenance mode. Check server configuration.
4. **After fix:** Re-crawl the affected URLs to verify they now return 200. Check Google Search Console for any indexing drops that occurred during the outage period.

---

### 1.3 Soft 404s

**What it is:** A page returns a 200 OK status code but displays error-page content ("page not found", "this product is no longer available", empty page, near-empty template). The server tells crawlers the page is fine, but the content says otherwise. Google explicitly identifies and flags soft 404s in Search Console.

**Why it matters:** Soft 404s are worse than real 404s in one specific way: they get indexed. A true 404 is immediately recognized as a dead page and excluded from the index. A soft 404 enters the index as a thin, low-quality page, which can dilute overall site quality signals. At scale, hundreds of indexed soft 404s send a site-wide quality signal to Google that the site has substantial low-value content.

**Severity:** **High** — because the page is being indexed when it should not be, and the 200 status prevents search engines from correctly identifying it as an error.

**How to identify in crawl data:**

- Soft 404s are not directly labeled in Screaming Frog's standard exports. Identify them through these proxy signals:
  - **Export tab:** `Internal:All` — look for pages with very low word count (< 50 words) combined with 200 status
  - **Export tab:** `Page Titles:All` — look for pages with titles containing "not found", "error", "page not found", "404", "no longer available"
  - **Google Search Console:** The Coverage report explicitly flags soft 404s — cross-reference with crawl data
- **Key columns to examine:** `Address`, `Status Code`, `Word Count`, `Title 1`, `H1-1`

**What to look for:**
- Pages returning 200 with word count under 50 — especially if they follow a URL pattern that suggests product/content pages
- Pages with titles or H1s matching error-page patterns
- Large groups of pages with identical low word counts (suggests a template rendering with no content)

**Recommended fix:**

1. **Return a proper 404 or 410 status code** for pages that are genuinely gone. This is the most important fix — it immediately tells search engines to stop indexing these pages.
2. **If the content was removed but the URL should persist** (e.g., out-of-stock product), either:
   - Keep the page live with useful content (related products, back-in-stock notification, category links)
   - 301 redirect to the most relevant alternative
3. **If caused by a CMS/template issue** (empty template rendering on database miss), fix the application logic to return 404 when no content record exists for a URL.
4. **Add a meta noindex tag** as a stopgap if the status code cannot be changed quickly — but fixing the status code is the correct long-term solution.

---

## 2. Redirects

### 2.1 Redirect Chains (3+ Hops)

**What it is:** A redirect chain occurs when URL A redirects to URL B, which redirects to URL C, and so on. A chain of 3 or more hops is problematic. Chains of 2 hops (A -> B -> C) are acceptable but not ideal.

**Why it matters:** Each hop in a redirect chain adds latency (typically 50-200ms per hop). Google has stated it will follow up to 10 redirects but may lose patience with long chains and simply stop following. More importantly, link equity degrades with each hop — while Google has said a single 301 passes full equity, the practical reality at 3+ hops is measurable ranking loss. Redirect chains also slow down crawling, meaning Googlebot spends its crawl budget on redirect hops instead of indexing content.

**Severity:**

| Chain Length | Severity | Rationale |
|-------------|----------|-----------|
| 3 hops | **High** | Noticeable equity loss, added latency, should be collapsed |
| 4-5 hops | **High** | Significant equity loss, material latency impact |
| 6+ hops | **Critical** | Risk of crawler abandonment, severe equity loss |
| Chain ending in 4xx/5xx | **Critical** | The final destination is broken — all linking pages effectively point to a dead end |

**How to identify in crawl data:**

- **Bulk export:** `All Redirect Chains`
- **Export tab:** `Response Codes:Redirection (3xx)`
- **Key columns:** `Address`, `Redirect URL`, `Redirect Type`, `Number of Redirects` (hop count)
- The bulk export shows full chain paths: source -> hop 1 -> hop 2 -> ... -> final destination

**What to look for:**
- Sort by hop count descending to find the worst chains first
- Check the final destination status — if a chain ends in a 404 or 5xx, it is **Critical** regardless of hop count
- Group chains by pattern: migrations often produce chains where an old URL structure redirects to a transitional structure, which redirects to the current structure
- Identify chains that cross domains (e.g., old-domain.com -> new-domain.com/old-path -> new-domain.com/new-path)

**Recommended fix:**

1. **Collapse the chain:** Update each redirect in the chain to point directly to the final destination. URL A should redirect to URL C (or wherever the chain ultimately resolves), not to URL B.
2. **Update internal links:** Any internal link pointing to a URL that redirects should be updated to point directly to the final destination URL. This eliminates the redirect entirely for internal traffic.
3. **For chains ending in 4xx/5xx:** Fix the final destination first, then collapse the chain.
4. **Prioritize by inlink count:** Chains with the most inbound links (internal + external) are losing the most equity and should be fixed first.
5. **Bulk fix via server config:** If chains follow a pattern (e.g., all from a specific migration), write a single rewrite rule that maps old URLs directly to current URLs, bypassing intermediate hops.

---

### 2.2 Redirect Loops

**What it is:** A redirect loop occurs when URL A redirects to URL B, and URL B redirects back to URL A (or through a series of hops that eventually cycle back). The page can never be reached — the browser or crawler bounces between URLs until it gives up.

**Why it matters:** A redirect loop is a complete failure. The page is 100% inaccessible to users and search engines. Browsers display "too many redirects" errors. Googlebot will stop following and the page will be de-indexed. If the looping URLs have inbound links, all that equity is permanently lost.

**Severity:** **Critical** — always. A redirect loop is functionally equivalent to the page not existing.

**How to identify in crawl data:**

- **Bulk export:** `All Redirect Chains` — look for chains where the final destination URL matches any earlier URL in the chain
- **Export tab:** `Response Codes:Redirection (3xx)` — look for URLs with redirect destinations that eventually point back to themselves
- Screaming Frog may flag these explicitly in the crawl overview or issues tab
- **Key signal:** A redirect chain where the "Number of Redirects" is unusually high (8-10) and the chain does not terminate in a 200

**What to look for:**
- Any chain where the final URL equals the first URL
- Chains that hit the maximum redirect count without resolving to a 200 page
- Patterns: loops often occur when HTTP redirects to HTTPS which redirects back to HTTP, or when www redirects to non-www which redirects back to www

**Recommended fix:**

1. **Identify the conflicting rules:** Loops are almost always caused by two redirect rules that contradict each other. Common causes:
   - HTTP->HTTPS redirect + a rule that redirects the HTTPS version back to HTTP
   - www->non-www redirect at the DNS/CDN level + non-www->www redirect at the server level
   - CMS-level redirect + .htaccess/nginx redirect targeting the same URL with different destinations
2. **Remove one of the conflicting rules.** Determine which redirect is correct and delete the other.
3. **Test the fix** by following the full redirect path manually (use `curl -IL <url>` to trace the chain).
4. **Re-crawl the affected URLs** to confirm the loop is resolved and the page now returns 200.

---

### 2.3 Temporary Redirects (302) Used as Permanent

**What it is:** A 302 (Found) or 307 (Temporary Redirect) is used for a redirect that is clearly permanent in intent — such as a domain migration, a permanent URL structure change, or a page that was permanently moved. The redirect works for users (they reach the right page), but the signal to search engines is wrong.

**Why it matters:** With a 302, search engines are told the redirect is temporary, so they may:
- Keep the old URL in the index instead of replacing it with the new one
- Not transfer full link equity to the destination — Google has said they generally do pass equity through 302s, but the behavior is less reliable and consistent than with 301s
- Continue crawling the old URL on every crawl cycle, wasting crawl budget
- Display the old URL in search results even though it redirects

The practical risk varies. Google has become better at inferring permanent intent from long-lived 302s, but relying on inference instead of explicit signals is an unnecessary gamble.

**Severity:** **Medium** — not immediately damaging, but creates ongoing inefficiency and uncertainty about indexation signals.

**How to identify in crawl data:**

- **Export tab:** `Response Codes:Redirection (3xx)`
- **Key columns:** `Address`, `Status Code`, `Redirect URL`, `Redirect Type`
- Filter for `Status Code = 302` or `Status Code = 307`
- Assess intent: is this redirect genuinely temporary (A/B test, seasonal page, maintenance), or is it permanent in practice?

**What to look for:**
- 302 redirects that have been in place for more than a few weeks — if a redirect has been active for months, it is de facto permanent
- 302 redirects used in domain migrations or HTTPS migrations — these should always be 301
- 302 redirects from old URL structures to new URL structures after a site redesign
- A large number of 302 redirects (suggests the server or CMS defaults to 302 instead of 301)

**Recommended fix:**

1. **Change the status code from 302 to 301** for any redirect that is permanent in intent. This is usually a one-line change in server config, .htaccess, nginx config, or CMS redirect settings.
2. **Keep 302 for genuinely temporary redirects:** seasonal pages, A/B tests, temporary maintenance pages, content that will return to the original URL.
3. **Check CMS defaults:** Some CMS platforms (notably some WordPress redirect plugins) default to 302. Change the default to 301.
4. **Bulk fix:** If the server/CMS defaults are causing widespread 302s, a single configuration change can fix all of them at once.

---

## 3. Orphan Pages

**What it is:** An orphan page is a URL that exists on the site (returns 200, may even be indexed) but has zero internal inlinks. No other page on the site links to it. The only way search engines can discover it is through the XML sitemap, external links, or previously cached URLs.

**Why it matters:** Internal links are the primary mechanism by which search engines discover and evaluate pages. A page with zero internal links is:
- **Poorly discoverable:** Googlebot may never find it if it is not in the sitemap, or may crawl it infrequently
- **Perceived as low-value:** Google uses internal link count as a signal of page importance. Zero inlinks = the site itself does not consider this page important enough to link to
- **Missing link equity:** Internal links distribute PageRank throughout the site. Orphan pages receive none.

**Severity:**

| Condition | Severity | Rationale |
|-----------|----------|-----------|
| Orphan page is a key content/product/category page | **High** | Important page is essentially hidden from search engines |
| Orphan page is in the XML sitemap but has no inlinks | **High** | You are asking search engines to index a page your own site does not link to — contradictory signals |
| Orphan page is a low-value or deprecated page | **Low** | May not need internal links; consider whether it should exist at all |

**How to identify in crawl data:**

- **Bulk export:** `Orphan Pages`
- **Key columns:** `Address`, `Status Code`, `Indexability`, `Word Count`, `Title 1`
- Cross-reference with XML sitemap: are orphan pages included in the sitemap?
- Check `Indexability` — if an orphan page is indexable, it can still appear in search results despite having no internal support

**What to look for:**
- Total count of orphan pages as a percentage of total crawled pages — a high percentage (>10%) suggests structural navigation issues
- Group by URL pattern: are all orphan pages from a specific section (e.g., old blog posts, archived products)?
- Check word count and content quality: orphan pages with substantial content (500+ words) are higher priority than thin pages
- Verify indexability: orphan pages that are indexable and have content are the most important to fix

**Recommended fix:**

1. **Add internal links** from relevant pages. The best approach is contextual links from topically related content, followed by inclusion in category/hub pages, followed by footer/sidebar navigation.
2. **If the page should not exist:** Either 301 redirect it to a relevant page, or return a 404/410 and remove it from the sitemap.
3. **Audit site architecture:** A high number of orphan pages often indicates a structural problem — pages created by the CMS that are not integrated into the navigation, blog posts not linked from category pages, product pages not linked from collection pages.
4. **Ensure the XML sitemap includes orphan pages** as a stopgap while internal links are being added. The sitemap alone is not a substitute for internal links, but it ensures discovery.
5. **Prioritize by value:** Fix orphan pages that have the most content, the most external backlinks, or the highest potential search traffic first.

---

## 4. Crawl Depth

**What it is:** Crawl depth is the number of clicks required to reach a page from the site's homepage (or other seed URL). A page at depth 1 is directly linked from the homepage. A page at depth 3 requires three clicks through internal links to reach. Screaming Frog measures this as the shortest path from the crawl start URL.

**Why it matters:** Crawl depth directly affects how frequently and reliably search engines crawl a page. Pages deeper in the site structure are:
- **Crawled less frequently:** Googlebot allocates its crawl budget based on perceived importance, and deep pages are perceived as less important
- **Indexed more slowly:** New deep pages may take weeks or months to be discovered and indexed
- **Given less PageRank:** Each layer of depth dilutes the internal link equity flowing from the homepage

Google's John Mueller has confirmed that crawl depth (clicks from homepage) matters more than URL path depth (number of slashes in the URL).

**Severity:**

| Crawl Depth | Severity | Rationale |
|-------------|----------|-----------|
| 1-3 clicks | **None** | Normal, healthy site structure |
| 4-5 clicks | **Medium** | Pages may be crawled less frequently; acceptable for low-priority content but problematic for important pages |
| 6+ clicks | **High** | Pages are likely crawled infrequently and may struggle to rank; indicates a flat architecture is needed |
| 10+ clicks | **Critical** | Pages may not be crawled at all in a reasonable timeframe |

**How to identify in crawl data:**

- **Export tab:** `Internal:All`
- **Key columns:** `Address`, `Crawl Depth`, `Inlinks`, `Indexability`
- Generate a distribution: count pages at each depth level to see the overall site structure shape
- **Save report:** `Crawl Overview` — often includes crawl depth distribution statistics

**What to look for:**
- **Distribution shape:** A healthy site has most pages at depth 1-3, with a long tail dropping off sharply. A site with significant content at depth 5+ has architectural problems.
- **Important pages at high depth:** Filter for indexable pages at depth 4+ and check whether any are high-value pages (product pages, category pages, key content pages).
- **Correlation with inlinks:** Deep pages usually have few inlinks. If a page is deep AND has few inlinks, it has compounding discoverability problems.

**Recommended fix:**

1. **Flatten the site architecture** by adding links from higher-level pages (homepage, category pages, hub pages) to deep content. The goal is to get all important pages within 3 clicks of the homepage.
2. **Add breadcrumb navigation** with structured data — this creates a clear internal link path from every page back to the homepage and through the hierarchy.
3. **Implement hub/pillar pages** that link to all related content within a topic cluster, reducing depth for the entire cluster.
4. **Add "related content" or "popular pages" modules** on templates to create cross-links between deep pages.
5. **Review pagination:** Deeply paginated archives (page 50+) push content to extreme depths. Consider "load more" patterns, filtering, or linking to specific page ranges.
6. **Do not solve with sitemap alone:** The XML sitemap helps with discovery but does not substitute for internal link structure. Google uses link structure, not sitemap presence, to evaluate page importance.

---

## 5. Robots Directives

### 5.1 Blocked by robots.txt

**What it is:** The page is disallowed in the site's `robots.txt` file, preventing search engine crawlers from accessing it. Note: `robots.txt` prevents crawling, not indexing — if a blocked page has external links pointing to it, Google may still index the URL (with a "blocked by robots.txt" note in Search Console) but cannot see its content.

**Why it matters:** If a page that should be indexed is blocked by `robots.txt`, search engines cannot crawl its content, cannot evaluate its quality, and cannot rank it properly. The page may still appear in search results with no title or snippet (just the URL), which is worse than not appearing at all. Conversely, if a page is intentionally blocked (admin pages, staging, internal tools), this is correct behavior.

**Severity:**

| Condition | Severity | Rationale |
|-----------|----------|-----------|
| Indexable/important page blocked by robots.txt | **Critical** | Directly prevents the page from being crawled and ranked |
| Page blocked by robots.txt AND has inlinks | **Critical** | Link equity flows to the URL but search engines cannot access the content |
| Intentionally private/admin page blocked | **None** | Correct configuration — no action needed |
| Staging or test URLs blocked | **None** | Correct configuration |

**How to identify in crawl data:**

- **Export tab:** `Directives:All`
- **Key columns:** `Address`, `Indexability`, `Indexability Status`, `Robots.txt Status`
- Filter for `Robots.txt Status = Blocked` or `Indexability Status = Blocked by Robots.txt`
- Cross-reference with inlinks and sitemap data

**What to look for:**
- Pages blocked by robots.txt that are also in the XML sitemap — this is a direct contradiction (sitemap says "index this", robots.txt says "don't crawl this")
- Important page types (product pages, category pages, blog posts) that are blocked
- Overly broad `Disallow` rules that accidentally block public content (e.g., `Disallow: /search` blocking `/search-results/best-running-shoes/`)
- Check if the block is at the directory level vs. specific URL level

**Recommended fix:**

1. **Remove the robots.txt block** for pages that should be indexed. Update the `Disallow` rule to be more specific, or remove it entirely.
2. **If the page should not be indexed but also should not be blocked:** Remove the robots.txt block and use a `<meta name="robots" content="noindex">` tag instead. This is the correct way to prevent indexing while allowing crawling (which lets search engines see and follow the links on the page).
3. **Audit robots.txt rules:** Review the full robots.txt file for overly broad rules. A single `Disallow: /` typo can block the entire site.
4. **Fix sitemap contradictions:** If a URL is blocked by robots.txt, remove it from the XML sitemap. Never include blocked URLs in the sitemap.
5. **Use robots.txt for crawl budget management,** not for hiding content. If content should be hidden from search, use noindex. If content should not be crawled at all (e.g., infinite faceted navigation), robots.txt is appropriate.

---

### 5.2 Noindex/Nofollow Conflicts

**What it is:** A page has conflicting or contradictory robots directives. Common conflicts include:
- `noindex` in the meta robots tag but the page is in the XML sitemap
- `noindex` on a page that has significant internal links pointing to it
- `noindex` in the HTTP header AND the meta tag (not a conflict per se, but redundant — and can mask issues if one is later removed)
- `noindex, follow` combined with `nofollow` links TO the page (equity flows in via links but the page says "don't index me", and the page itself blocks outbound equity flow)
- Canonical tag pointing to a different URL combined with a `noindex` tag on the same page (conflicting signals — canonical says "this is the same as X", noindex says "don't index this")

**Why it matters:** Conflicting directives confuse search engines. When signals contradict each other, Google must choose which to honor, and its choice may not match your intent. Common consequences:
- A page you want indexed is not indexed because of a stray `noindex` tag
- A page you want de-indexed remains indexed because the canonical tag overrides the `noindex` in Google's evaluation
- Link equity is trapped: `nofollow` on internal links to a page means equity is not passed, even if the destination page is valuable

**Severity:** **High** — conflicting signals create unpredictable behavior, and the impact depends on which signal search engines choose to honor.

**How to identify in crawl data:**

- **Export tab:** `Directives:All`
- **Key columns:** `Address`, `Meta Robots 1`, `X-Robots-Tag 1`, `Canonical Link Element 1`, `Indexability`, `Indexability Status`, `Inlinks`
- **Cross-reference with:** XML sitemap export (are noindexed pages in the sitemap?) and `Canonicals:All`

**Specific conflict patterns to look for:**

| Conflict | How to Find | Impact |
|----------|-------------|--------|
| Noindex + in sitemap | Filter `Directives:All` for `noindex`, then cross-reference with sitemap URLs | Contradictory: sitemap says index, tag says don't |
| Noindex + canonical to self | Filter for pages with `noindex` AND self-referencing canonical | Minor conflict: canonical is redundant on a noindex page, but not harmful |
| Noindex + canonical to different URL | Filter for pages with `noindex` AND canonical pointing elsewhere | Ambiguous: Google may follow canonical instead of noindex |
| Noindex on page with high inlinks | Filter for `noindex` pages, sort by inlink count descending | Link equity is flowing to a page that will not be indexed |
| Meta robots noindex + X-Robots-Tag index (or vice versa) | Compare `Meta Robots 1` and `X-Robots-Tag 1` columns for contradictions | Google takes the most restrictive directive — noindex wins |

**Recommended fix:**

1. **Resolve each conflict to a single, clear intent.** Decide: should this page be indexed or not? Then make ALL signals consistent with that decision.
2. **If the page should be indexed:**
   - Remove the `noindex` tag (both meta and HTTP header)
   - Ensure it is in the XML sitemap
   - Ensure internal links to the page do not use `rel="nofollow"`
   - Set a self-referencing canonical tag
3. **If the page should NOT be indexed:**
   - Add `noindex` via meta tag or X-Robots-Tag
   - Remove the page from the XML sitemap
   - Remove or update the canonical tag (a canonical tag on a noindex page sends mixed signals)
   - Consider whether internal links to this page should remain (they can — `noindex` does not block crawling, and links on the noindex page can still pass equity)
4. **Audit at the template level:** Conflicting directives usually come from different systems (CMS adds canonical, plugin adds noindex, server config adds X-Robots-Tag). Check each template type for consistency.

---

## 6. Sitemap Issues

### 6.1 Non-200 URLs in Sitemap

**What it is:** The XML sitemap contains URLs that do not return a 200 status code. These may be 301 redirects, 302 redirects, 404 errors, 410 errors, or 5xx errors. The sitemap is supposed to contain only canonical, indexable, 200-status URLs.

**Why it matters:** The XML sitemap is a direct signal to search engines: "these are the pages I want you to crawl and index." Including non-200 URLs in the sitemap:
- **Wastes crawl budget:** Search engines will request these URLs, encounter errors or redirects, and have spent crawl capacity on nothing useful
- **Erodes trust in the sitemap:** If a significant percentage of sitemap URLs are invalid, search engines may reduce the weight they give to the sitemap as a crawling signal
- **Delays indexation of new content:** Crawl budget spent on invalid sitemap URLs is budget not spent on new or updated pages

**Severity:** **High** — the sitemap is one of the primary mechanisms for guiding crawl behavior, and polluting it with bad URLs undermines its purpose.

**How to identify in crawl data:**

- Cross-reference the XML sitemap URLs with the main crawl data. Filter for sitemap URLs where `Status Code != 200`.
- **Export tab:** `Response Codes:All` — check the `In Sitemap` column (if available) or cross-reference URL lists
- **Key columns:** `Address`, `Status Code`, `In Sitemap`
- Also check: are redirected URLs in the sitemap? The sitemap should contain the final destination URL, not the redirect source.

**What to look for:**
- Count of non-200 sitemap URLs as a percentage of total sitemap URLs — greater than 5% is a significant problem
- Breakdown by status code: 301s in the sitemap are common after migrations; 404s suggest stale sitemap generation; 5xx suggests server issues
- Check sitemap generation: is the sitemap auto-generated by the CMS, manually maintained, or generated by a plugin? The generation method determines the fix.

**Recommended fix:**

1. **Remove all non-200 URLs from the sitemap.** This is non-negotiable.
2. **For redirected URLs:** Replace the old URL with the redirect destination URL in the sitemap.
3. **For 404/410 URLs:** Remove them from the sitemap entirely.
4. **Fix the sitemap generation process:**
   - If auto-generated: configure the generator to exclude non-200 URLs (most CMS sitemap plugins have this option)
   - If manually maintained: establish a process to update the sitemap when URLs are removed or redirected
   - If generated by a build process: add a validation step that checks each URL's status code before including it
5. **Set up monitoring:** Check the sitemap monthly (or after every migration/restructure) to catch non-200 URLs before they accumulate.

---

### 6.2 Indexable URLs Missing from Sitemap

**What it is:** Pages that are indexable (200 status, no noindex, self-canonicalizing) are not included in the XML sitemap. These pages can still be discovered and indexed via internal links, but they lack the explicit sitemap signal.

**Why it matters:** The XML sitemap serves as a discovery hint and a priority signal. Pages missing from the sitemap:
- May be discovered later (or not at all) if they have few internal links
- Miss out on `<lastmod>` signals that tell search engines when to re-crawl
- Are not explicitly flagged as pages the site owner considers important

That said, the sitemap is not required for indexation. Pages with strong internal links will be crawled and indexed regardless. The impact of being missing from the sitemap scales inversely with the page's internal link support.

**Severity:**

| Condition | Severity | Rationale |
|-----------|----------|-----------|
| Important pages with few inlinks missing from sitemap | **Medium** | These pages need the sitemap for discovery |
| Well-linked pages missing from sitemap | **Low** | Search engines already discover them via links; sitemap inclusion is a minor optimization |
| Very large sites (100k+ pages) with significant gaps | **Medium** | At scale, sitemap coverage matters more for ensuring complete crawling |

**How to identify in crawl data:**

- Compare the set of indexable internal URLs from the crawl against the sitemap URL list.
- **Export tab:** `Internal:All` — filter for `Indexability = Indexable` and check `In Sitemap` column
- The difference = indexable URLs missing from the sitemap

**What to look for:**
- Percentage of indexable pages missing from sitemap — greater than 20% suggests the sitemap generation is not comprehensive
- Group missing pages by URL pattern or page type — are all blog posts missing? All product pages? This indicates a sitemap configuration gap.
- Cross-reference with crawl depth: pages at depth 4+ that are also missing from the sitemap have compounding discoverability issues

**Recommended fix:**

1. **Add missing indexable URLs to the sitemap.** The simplest approach is to configure the sitemap generator to automatically include all indexable, 200-status, self-canonicalizing pages.
2. **Segment the sitemap** by content type if the site is large (e.g., `sitemap-products.xml`, `sitemap-blog.xml`, `sitemap-categories.xml`). This makes it easier to verify completeness per section.
3. **Ensure dynamic/JS-rendered pages are included:** If the CMS or build system does not know about client-side-rendered pages, they may be missing from auto-generated sitemaps. Add them manually or via a separate generation process.
4. **Do not over-optimize:** Including every indexable URL in the sitemap is ideal, but the impact of missing URLs is low if those pages have strong internal links. Prioritize pages with few inlinks or high depth.

---

## 7. Pagination

### 7.1 Missing rel=prev/next

**What it is:** Paginated sequences (e.g., `/blog/page/1`, `/blog/page/2`, `/blog/page/3`) do not include `<link rel="prev">` and `<link rel="next">` tags in the `<head>` to indicate the pagination relationship.

**Why it matters:** Google announced in 2019 that it no longer uses `rel=prev/next` as an indexing signal. However:
- **Bing and other search engines** still use `rel=prev/next` for understanding pagination
- **It clarifies content relationships** and can help search engines consolidate ranking signals across a paginated series
- **It is a best-practice signal** that the site is well-structured, even if the direct ranking impact on Google is minimal

**Severity:** **Low** — Google has deprecated this signal, but it remains useful for other search engines and is still considered a best practice by most SEO tooling.

**How to identify in crawl data:**

- **Export tab:** `Directives:All` or `Internal:All`
- **Key columns:** `Address`, `Rel Prev`, `Rel Next`
- Filter for URLs matching pagination patterns (e.g., URLs containing `/page/`, `?page=`, `?p=`, `/p/`) where `Rel Prev` and `Rel Next` are both empty
- Check the first page: it should have `rel="next"` but no `rel="prev"`. The last page should have `rel="prev"` but no `rel="next"`.

**What to look for:**
- Are ANY paginated URLs using rel=prev/next, or is it absent site-wide? Site-wide absence suggests the template does not implement it.
- Incorrect implementations: rel=prev/next pointing to non-existent pages, pointing to the wrong page in the sequence, or both prev and next pointing to the same URL.

**Recommended fix:**

1. **Add rel=prev/next tags** to paginated page templates. Implementation:
   - First page: `<link rel="next" href="/page/2/">`
   - Middle pages: `<link rel="prev" href="/page/1/">` and `<link rel="next" href="/page/3/">`
   - Last page: `<link rel="prev" href="/page/4/">`
2. **Use absolute URLs** in the href values, not relative URLs.
3. **Ensure consistency with canonicals:** Each paginated page should have a self-referencing canonical. Do NOT canonical all paginated pages to page 1 — this tells search engines that pages 2+ are duplicates, which hides their unique content.
4. **Priority:** This is a low-priority fix. Address it as part of template improvements, not as an urgent SEO task.

---

### 7.2 Noindexed Pagination Pages

**What it is:** Paginated pages (page 2, page 3, etc.) have a `<meta name="robots" content="noindex">` tag, preventing search engines from indexing them.

**Why it matters:** The impact depends entirely on whether the paginated pages contain unique content:

- **If paginated pages contain unique items** (e.g., an e-commerce category page where page 2 has products 21-40, and those products are ONLY accessible through this pagination): noindexing these pages means those products may never be discovered or indexed by search engines. The pagination pages serve as the only crawl path to that content.
- **If paginated pages are pure navigation** (e.g., a blog archive where each post also has its own URL accessible through other links): noindexing pagination is acceptable and can even be beneficial to reduce index bloat.

**Severity:**

| Condition | Severity | Rationale |
|-----------|----------|-----------|
| Paginated pages with unique content (products, listings) that have no other crawl path | **Medium** | Content on those pages may not be discovered or indexed |
| Paginated pages where all items are independently linked elsewhere | **Low** | Noindex is acceptable; items are discovered through other paths |
| Noindex + nofollow on pagination pages | **High** | Nofollow prevents crawlers from following links on the page, cutting off discovery of items listed on paginated pages entirely |

**How to identify in crawl data:**

- **Export tab:** `Directives:All`
- **Key columns:** `Address`, `Meta Robots 1`, `Indexability`, `Indexability Status`
- Filter for URLs matching pagination patterns where `Indexability Status = Noindex`
- **Critical check:** Look at the meta robots value — is it `noindex` or `noindex, nofollow`? The presence of `nofollow` dramatically increases severity because it blocks link discovery.

**What to look for:**
- Which paginated sections use noindex? Are all paginated templates noindexed, or only specific ones?
- Check whether items on paginated pages have alternative crawl paths (are products linked from other pages besides the category pagination?)
- Check if `nofollow` is combined with `noindex` — this is the high-severity scenario
- Look at crawl depth of items that are only reachable through pagination — if they are at depth 5+ AND the pagination pages are noindexed with nofollow, those items are effectively invisible

**Recommended fix:**

1. **If paginated pages have unique content with no other crawl path:**
   - Remove `noindex` from pagination pages, OR
   - Ensure all items on paginated pages are linked from other indexable pages (e.g., add a "view all" page, add items to secondary navigation, create filtered landing pages)
2. **If noindex + nofollow is used:** At minimum, change to `noindex, follow`. This prevents the pagination page itself from being indexed while still allowing search engines to discover and follow links to the items on the page.
3. **If paginated pages are pure navigation (all items linked elsewhere):** `noindex, follow` is the ideal configuration. It keeps the pagination page out of the index while preserving link discovery.
4. **Never use noindex + nofollow on pagination** unless you intentionally want to block discovery of all content linked from those pages.

---

## Quick Reference: Severity Summary

| Issue | Default Severity | Escalates To | Condition for Escalation |
|-------|-----------------|-------------|-------------------------|
| 404 Not Found | Medium | **Critical** | Has inlinks or is in sitemap |
| 5xx Server Error | **Critical** | — | Always critical |
| Soft 404 | **High** | — | Always high when detected |
| Redirect Chain (3+ hops) | **High** | **Critical** | Chain ends in 4xx/5xx or 6+ hops |
| Redirect Loop | **Critical** | — | Always critical |
| 302 Used as Permanent | Medium | — | — |
| Orphan Page | **High** | Low | Page is low-value/deprecated |
| Crawl Depth 4-5 | Medium | — | — |
| Crawl Depth 6+ | **High** | **Critical** | 10+ clicks from homepage |
| Blocked by robots.txt | **Critical** | None | Page should be indexed; None if intentional |
| Noindex/Nofollow Conflicts | **High** | — | Always high when directives conflict |
| Non-200 URLs in Sitemap | **High** | — | — |
| Indexable URLs Missing from Sitemap | Low-Medium | — | — |
| Missing rel=prev/next | Low | — | — |
| Noindexed Pagination | Low-Medium | **High** | noindex + nofollow blocks discovery |
