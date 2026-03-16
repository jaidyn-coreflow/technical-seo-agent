# Link Structure Reference

Use this reference when analyzing crawl data related to internal linking health, link equity distribution, broken internal links, anchor text optimization, or link depth. Each issue includes what it is, why it matters, severity classification, how to identify it in Screaming Frog crawl data, and the recommended fix.

---

## 1. Low Inlink Pages

**What it is:** A page that receives fewer than 3 internal links from other pages on the site. These pages are under-supported by the site's own link structure, meaning the site is not signaling their importance through internal navigation, contextual links, or related content sections. Low inlink count is relative to page importance -- a legal disclaimer page with 1 inlink is normal, but a core product category page with 1 inlink is a serious structural failure.

**Why it matters:** Internal links are the primary mechanism by which search engines evaluate page importance within a site. Every internal link is a vote of confidence: "this page matters enough for us to link to it." Pages with fewer than 3 internal links:
- **Receive minimal link equity:** PageRank flows through internal links. A page with 1-2 inlinks receives only a fraction of the equity available to well-linked pages, making it nearly impossible to compete for competitive keywords.
- **Are crawled less frequently:** Googlebot uses link count as a proxy for page importance when deciding how often to re-crawl. Low inlink pages may be crawled weekly or monthly instead of daily.
- **Signal low importance to ranking algorithms:** Google's internal documentation (leaked in 2024) confirmed that internal link count is a direct factor in how the system evaluates page importance within a site.

**Severity:**

| Condition | Severity | Rationale |
|-----------|----------|-----------|
| Important page (product, category, key landing page) with < 3 inlinks | **High** | The site's own structure is undermining this page's ability to rank |
| Content page (blog post, article) with < 3 inlinks | **Medium** | Expected for older or less relevant content, but still represents an optimization opportunity |
| Utility page (privacy policy, terms of service, contact) with < 3 inlinks | **Low** | These pages do not need ranking support; 1-2 inlinks from footer/navigation is sufficient |
| Key page with 0 inlinks | **Critical** | This is an orphan page -- see the Crawlability reference for orphan page handling |

**How to identify in crawl data:**

- **Export tab:** `Internal:All`
- **Key columns:** `Address`, `Inlinks` (count), `Unique Inlinks`, `Indexability`, `Crawl Depth`
- **Bulk export:** `All Inlinks` -- provides the full inlink detail, including which pages link to each URL and the anchor text used
- Filter for pages where `Unique Inlinks < 3` and `Indexability = Indexable` and `Status Code = 200`

**What to look for:**
- Sort indexable pages by inlink count ascending -- the pages with the fewest inlinks are the most under-supported
- Cross-reference with page type: are product pages, category pages, or high-value content pages in the low-inlink group? These are the highest priority fixes.
- Compare inlink counts across page types: if category pages average 15 inlinks but a specific category page has 2, that page is structurally neglected
- Check crawl depth: pages with low inlinks AND high crawl depth (4+) have compounding discoverability problems
- Look for patterns: are all pages in a specific section under-linked? This suggests a navigation or template gap rather than individual page oversight

**Recommended fix:**

1. **Add contextual links from related content.** The highest-value internal links are in-body links from topically related pages. If a product page has few inlinks, link to it from relevant blog posts, buying guides, comparison pages, and related product pages.
2. **Add "related content" or "you may also like" sections** to templates. These dynamically generated link modules can add 3-8 internal links to every page based on category, tags, or content similarity.
3. **Create hub pages or pillar pages** that link to all content within a topic cluster. A well-structured hub page can add an inlink to dozens of under-linked pages at once.
4. **Review navigation and category structures.** If important pages are not reachable through navigation menus, sidebar links, or category listings, the information architecture needs restructuring.
5. **Add breadcrumb navigation** with structured data. Breadcrumbs create a reliable internal link path from every page back through the site hierarchy, adding at least 2-3 inlinks per page.
6. **Prioritize by business value.** Fix inlink deficits on pages that target the most valuable keywords or drive the most revenue first. Not every page needs the same level of internal link support.

---

## 2. Link Equity Distribution

**What it is:** Link equity distribution measures whether the site's internal linking structure concentrates link authority on the most important pages. A well-optimized site funnels the majority of internal link equity toward pages that target the most competitive and commercially valuable keywords. Poor distribution occurs when link equity is spread evenly across all pages (including low-value utility pages), or when high-priority pages receive fewer inlinks than low-priority ones.

**Why it matters:** Every site has a finite amount of internal link equity, originating primarily from the homepage (which receives the most external backlinks) and flowing outward through internal links. How that equity is distributed determines which pages have the ranking power to compete:
- **Concentrated equity on key pages** means those pages rank higher for competitive terms, driving more organic traffic and revenue
- **Diluted equity across all pages equally** means no single page has enough authority to compete for high-value keywords
- **Inverted distribution** (utility pages getting more links than money pages) is the most damaging pattern: the site is actively directing its authority away from the pages that matter most

This is not a binary pass/fail issue -- it is an optimization spectrum. Even well-structured sites can improve equity distribution.

**Severity:** **Medium** -- poor equity distribution is a significant optimization opportunity, but it does not cause indexation failures or crawl errors. It affects ranking potential rather than basic visibility.

**How to identify in crawl data:**

- **Export tab:** `Internal:All`
- **Key columns:** `Address`, `Unique Inlinks`, `Inlinks`, `Crawl Depth`, `Indexability`
- **Bulk export:** `All Inlinks` -- needed for detailed analysis of which pages link where
- Calculate the inlink distribution: what percentage of total internal links go to the top 10% of pages? What percentage go to pages that are not indexable or not strategically important?

**What to look for:**
- **Rank pages by inlink count** and compare against business priority. The pages with the most inlinks should be the pages you most want to rank. If your homepage has 500 inlinks but your top product category has 8, equity is not flowing where it matters.
- **Check utility page inlink counts.** Pages like privacy policy, terms of service, cookie policy, and contact pages appear in the site-wide footer, giving them hundreds or thousands of inlinks. This is normal and unavoidable, but if these utility pages have significantly more inlinks than your key landing pages, it highlights the distribution problem.
- **Identify "link sinks":** pages that receive many inlinks but do not pass equity forward (e.g., pages with noindex, pages with few outlinks). These absorb equity without contributing to the site's ranking goals.
- **Compare inlink counts within a page type.** If your e-commerce site has 50 category pages and most have 20-30 inlinks, but your highest-revenue category has only 5, that is a specific gap to fix.

**Recommended fix:**

1. **Audit and prioritize your page hierarchy.** Create a tiered list: Tier 1 pages (highest commercial value, most competitive keywords), Tier 2 pages (supporting content, secondary keywords), Tier 3 pages (utility, legal, low-priority). Internal linking effort should follow this tiering.
2. **Add prominent internal links to Tier 1 pages** from high-authority pages. Link from the homepage, top-level category pages, and high-traffic blog posts to your most important pages. These links carry the most equity.
3. **Implement topic clusters** with hub-and-spoke linking. Each hub page (Tier 1) links to all spokes (Tier 2 content), and each spoke links back to the hub. This concentrates equity on the hub while supporting the spokes.
4. **Use contextual links over navigational links.** A link within paragraph text on a topically relevant page passes more relevance signal than a link in a sidebar or footer. Prioritize in-content linking for Tier 1 pages.
5. **Reduce unnecessary outlinks from high-authority pages.** If your homepage links to 200 pages in the footer, each link gets 1/200th of the homepage's equity. Reducing footer links to only essential pages concentrates more equity on each remaining link.
6. **Review and optimize navigation menus.** Ensure that primary navigation links point to Tier 1 pages. Demote lower-priority pages to sub-navigation or footer links.

---

## 3. Broken Internal Links

**What it is:** An internal link on the site points to a URL that returns a 4xx (typically 404 Not Found) or 5xx (server error) status code. The link exists in the HTML of a live page, but the destination is broken. This includes links in navigation, body content, footers, sidebars, and any other on-page element.

**Why it matters:** Broken internal links are damaging on multiple fronts:
- **Wasted link equity:** Every internal link passes PageRank to its destination. When that destination is a 404, the equity is lost entirely -- it does not bounce back to the linking page or get redistributed.
- **Bad user experience:** Users who click a broken link hit a dead end. On e-commerce sites, a broken link to a product page means a lost sale. On content sites, it erodes trust and increases bounce rates.
- **Wasted crawl budget:** Googlebot follows every internal link it encounters. Each broken link wastes a crawl request on a dead page instead of a live one.
- **Quality signal:** A site with many broken internal links signals poor maintenance to search engines. While there is no confirmed "broken link penalty," site-wide quality signals do affect crawl priority and ranking.

**Severity:** **High** -- broken internal links combine equity loss, UX damage, and crawl waste. They should be fixed promptly.

| Condition | Severity | Rationale |
|-----------|----------|-----------|
| Broken link from navigation/header/footer (site-wide template) | **Critical** | Every page on the site has a broken link, multiplying the equity loss and UX damage across the entire site |
| Broken link in body content of high-traffic page | **High** | Directly impacts user experience on an important page and wastes equity |
| Broken link in body content of low-traffic page | **Medium** | Lower immediate impact but still wastes equity and should be fixed |
| Broken link pointing to 5xx (server error) | **Critical** | May indicate a broader server issue; see Crawlability reference for 5xx handling |

**How to identify in crawl data:**

- **Export tab:** `Response Codes:Client Error (4xx)` -- shows all destination URLs returning 4xx
- **Bulk export:** `All Inlinks` -- filter by destination URLs that match the 4xx list to find which pages contain the broken links
- **Key columns:** `Address` (the broken destination), `Status Code`, `Inlinks` (count of pages linking to it), then in the inlinks export: `Source` (the page containing the broken link), `Destination`, `Anchor Text`

**What to look for:**
- Sort broken destinations by inlink count descending -- a broken URL linked from 50 pages (likely a template/navigation link) is far more damaging than one linked from 1 page
- Group broken links by source page: if a single page has 10+ broken outgoing links, the page content is severely outdated or was migrated incorrectly
- Check for patterns in broken destinations: if all broken URLs share a path prefix (e.g., `/old-products/`), this suggests a section was removed or restructured without updating internal links
- Distinguish between broken links to internal pages vs. broken links to internal resources (CSS, JS, images) -- both matter, but broken page links have SEO impact while broken resource links affect rendering

**Recommended fix:**

1. **Update the link to point to the correct live URL.** If the destination page was moved, update the link to the new URL. This is always the best fix.
2. **If the destination was permanently removed:** Either update the link to point to the most relevant alternative page, or remove the link entirely if no suitable alternative exists.
3. **Implement 301 redirects** from broken destination URLs to relevant live pages. This is a faster fix than updating every source page, especially when the broken URL has many inlinks. However, updating the source links is still preferred as a second step to eliminate redirect hops.
4. **Fix template-level broken links first.** A single broken link in a site-wide header or footer template affects every page. Fixing one template link can resolve hundreds or thousands of broken link instances.
5. **Set up ongoing monitoring.** Broken links accumulate over time as content is removed, products are discontinued, or URLs change. Schedule regular crawls (monthly for large sites, quarterly for small sites) to catch new broken links before they accumulate.
6. **Check CMS link management.** Some CMS platforms track internal links and can flag or automatically update links when a page's URL changes. Enable this feature if available.

---

## 4. Excessive Outlinks

**What it is:** A page contains more than 100 outgoing internal links (or a combination of internal and external links exceeding 100). This includes all links in the page's HTML: navigation, footer, sidebar, body content, related content modules, tag clouds, and any other linked elements. The 100-link threshold is a practical guideline, not a hard rule -- Google has stated there is no strict limit, but historically recommended keeping pages under 100 links.

**Why it matters:** Every outgoing link on a page dilutes the link equity passed to each destination. If a page has 10 outgoing links, each destination receives roughly 1/10th of the page's equity. If that same page has 200 outgoing links, each destination receives roughly 1/200th. For important pages (especially the homepage and top-level category pages), excessive outlinks mean the equity passed to each destination is negligible:
- **Equity dilution:** High-priority destination pages receive less equity per link when surrounded by hundreds of other links
- **Crawl prioritization:** When Googlebot encounters a page with 200+ links, it must decide which to follow. Priority links may not be crawled on every visit.
- **User experience:** Pages with excessive links (especially large tag clouds, mega-menus with hundreds of items, or long lists of links) can overwhelm users and reduce engagement with the most important CTAs

**Severity:**

| Condition | Severity | Rationale |
|-----------|----------|-----------|
| Content page (blog post, product page) with > 100 outlinks | **Medium** | Equity dilution on pages where focused linking would be more effective |
| Homepage with > 150 outlinks | **Medium** | Homepage equity is the most valuable; excessive links dilute it across too many destinations |
| Navigation/hub page with > 100 outlinks | **Low** | Navigation pages and category hubs are expected to have many links; this is their purpose |
| Any page with > 250 outlinks | **Medium** | Diminishing returns on equity per link; review whether all links are necessary |
| Tag cloud or auto-generated link block creating 100+ links | **Medium** | Often low-value links that dilute equity without adding navigation value |

**How to identify in crawl data:**

- **Export tab:** `Internal:All`
- **Key columns:** `Address`, `Outlinks` (total count), `Unique Outlinks`, `External Outlinks`, `Indexability`
- Filter for pages where `Outlinks > 100` or `Unique Outlinks > 100`
- **Bulk export:** `All Outlinks` -- shows every outgoing link from each page, allowing you to see what is generating the high link count

**What to look for:**
- Sort pages by outlink count descending to find the worst offenders
- Examine the outlinks export for the top offenders: are the links from navigation (expected), body content (potentially fine), or auto-generated modules (often the problem)?
- Check whether the page type justifies the link count: a sitemap-style index page with 500 links is by design; a blog post with 150 links likely has a problem
- Identify link patterns: tag clouds, "related posts" modules showing 50+ posts, mega-menus with every subcategory, or footer link blocks listing every city/location page
- Compare homepage outlink count against the number of Tier 1 pages -- if the homepage links to 200 pages but only 20 are strategically important, the other 180 are diluting equity

**Recommended fix:**

1. **Audit and reduce navigation links.** If the main navigation menu lists every subcategory (producing 80+ nav links), restructure to show only top-level categories with expandable sub-menus that load on interaction rather than being present in the initial HTML.
2. **Limit "related content" modules.** Show 4-8 related items instead of 20-50. The goal is to link to the most relevant alternatives, not every possible alternative.
3. **Remove or reduce tag clouds.** Tag clouds with 50-100+ links are an outdated pattern that dilutes equity. Replace with a curated list of 10-15 top categories or topics.
4. **Use `nofollow` strategically on low-value links.** Links to login pages, "add to cart" actions, external partner sites, and other non-SEO-relevant destinations can use `rel="nofollow"` to prevent equity flow. Note: this does not "save" equity for other links (Google treats nofollow links as suggestions, and the equity may simply evaporate), but it clarifies link intent.
5. **Consolidate footer links.** If the footer contains links to every location page, policy page, and utility page, reduce to essential links only. Move comprehensive link lists to dedicated sitemap or directory pages.
6. **Paginate long link lists.** If a category page must link to 500 products, paginate with 20-50 per page rather than listing all on one page. This reduces per-page outlink count and creates a more focused equity flow.

---

## 5. Anchor Text Optimization

**What it is:** Anchor text is the visible, clickable text of a hyperlink. For internal links, anchor text provides a direct relevance signal to search engines about the destination page's topic. Generic anchor text ("click here", "read more", "learn more", "this page") wastes this signal by telling search engines nothing about what the destination page is about. Optimized anchor text uses descriptive, keyword-relevant phrases that align with the destination page's target keywords.

**Why it matters:** Internal link anchor text is one of the strongest on-page relevance signals available:
- **Search engines use anchor text to understand the destination page.** When 10 internal links point to a page with the anchor text "running shoes for women," Google understands that the destination page is about women's running shoes, reinforcing the page's topical relevance.
- **Generic anchors waste this signal.** When 10 internal links point to a page with the anchor text "click here," Google learns nothing about the destination from those links. The link equity still flows, but the relevance signal is lost.
- **Anchor text diversity matters.** If every internal link to a page uses the exact same anchor text, it looks unnatural. A healthy anchor text profile uses variations: "women's running shoes," "running shoes for women," "best women's runners," and natural phrases that include the core topic.

**Severity:** **Low** -- anchor text optimization is a refinement, not a foundational fix. It will not cause indexation failures or crawl issues, and its ranking impact is incremental rather than transformative. However, for highly competitive keywords, anchor text optimization can be the marginal improvement that pushes a page from position 5 to position 3.

**How to identify in crawl data:**

- **Bulk export:** `All Inlinks` -- this is the primary data source, containing every internal link with its anchor text
- **Key columns (in inlinks export):** `Source`, `Destination`, `Anchor Text`, `Type` (text link vs. image link), `Follow`
- Filter for text links (`Type = Hyperlink`) and group by destination URL, then examine the anchor text values for each destination

**What to look for:**
- **High-value pages with predominantly generic anchors.** Filter inlinks to your Tier 1 pages and check anchor text. If the majority of links use "click here," "read more," "learn more," or other non-descriptive text, there is an optimization opportunity.
- **Image links without alt text.** Internal links using images pass equity but have no anchor text unless the image has an `alt` attribute. A linked image with no alt text is equivalent to an anchor text of "" (empty) -- the link passes equity but no relevance signal.
- **Over-optimized anchor text.** If every single internal link to a page uses the exact same keyword phrase, this can appear manipulative. Look for diversity: a mix of exact-match keywords, partial-match phrases, branded terms, and natural language.
- **Anchor text mismatches.** Links where the anchor text does not match the destination page's topic (e.g., anchor text says "blue widgets" but the destination is a page about red widgets) send confusing relevance signals.

**Recommended fix:**

1. **Update generic anchors on high-value links.** Start with links on high-authority pages (homepage, top category pages, high-traffic blog posts) that point to Tier 1 pages. Change "click here" or "learn more" to descriptive phrases that include the destination page's target keywords.
2. **Use natural, varied anchor text.** For a page targeting "best wireless headphones," use a mix: "wireless headphones," "the best wireless headphones," "top-rated wireless headphones," "these Bluetooth headphones," and natural contextual phrases. Avoid using the exact same anchor text on every link.
3. **Add alt text to linked images.** Every image that serves as a link should have descriptive alt text that functions as anchor text. Use alt text that describes the destination content, not just the image.
4. **Fix anchor text mismatches.** If a link's anchor text does not match the destination's topic, either update the anchor text to be relevant or change the link to point to the correct destination.
5. **Do not over-optimize.** Anchor text should read naturally within the surrounding content. Forcing exact-match keyword anchors into every link creates a poor reading experience and can trigger over-optimization filters. When in doubt, write the anchor text for the user, not for search engines.
6. **Prioritize contextual body links.** Anchor text in body content carries more relevance weight than anchor text in navigation, footers, or sidebars. Focus optimization efforts on in-content links first.

---

## 6. Link Depth (Click Depth)

**What it is:** Link depth (also called click depth) is the minimum number of clicks required to reach a page from the homepage through internal links. A page at depth 1 is linked directly from the homepage. A page at depth 4 requires navigating through 4 intermediate pages to reach it. Link depth is distinct from URL path depth (the number of directory levels in the URL): a page at `example.com/a/b/c/d/page` has a URL path depth of 5, but if the homepage links directly to it, its click depth is 1.

**Why it matters:** Click depth is one of the strongest structural signals of page importance:
- **Crawl frequency decreases with depth.** Googlebot allocates crawl budget based on perceived importance, and pages deeper in the structure are crawled less often. Pages at depth 6+ may only be crawled every few weeks, meaning content updates take much longer to be reflected in search results.
- **Link equity diminishes with each level.** PageRank flows from the homepage outward, losing strength at each hop. A page at depth 2 receives exponentially more homepage equity than a page at depth 5.
- **Indexation speed correlates with depth.** New pages at depth 1-2 are typically indexed within days. New pages at depth 5+ may take weeks or months.
- **Google's John Mueller has confirmed** that click depth (clicks from homepage) matters more than URL path depth for how Google evaluates page importance.

Key pages -- product pages, category pages, high-value content -- should be reachable within 3 clicks from the homepage. Anything beyond that indicates a structural problem that limits ranking potential.

**Severity:**

| Click Depth | Severity | Rationale |
|-------------|----------|-----------|
| 1-3 clicks | **None** | Healthy structure; pages are well-supported and easily discoverable |
| 4-5 clicks | **Medium** | Pages may be crawled less frequently; acceptable for low-priority content but problematic for pages targeting competitive keywords |
| 6+ clicks | **High** | Pages are likely crawled infrequently, receive minimal homepage equity, and may struggle to rank for any meaningful keywords |
| 8+ clicks | **Critical** | Pages may not be crawled within a reasonable timeframe; effectively invisible to search engines without sitemap support |

**How to identify in crawl data:**

- **Export tab:** `Internal:All`
- **Key columns:** `Address`, `Crawl Depth`, `Unique Inlinks`, `Inlinks`, `Indexability`
- **Save report:** `Crawl Overview` -- typically includes a crawl depth distribution showing page counts at each depth level
- Generate a distribution: count indexable pages at each depth level to see the site structure shape

**What to look for:**
- **Distribution shape.** A healthy site has the majority of indexable pages at depth 1-3, with a rapid drop-off at depth 4+. If the distribution is flat or increases at higher depths, the site architecture is too deep.
- **Important pages at depth 4+.** Filter for indexable pages at depth 4+ and cross-reference with page type. Product pages, category pages, and key content pages at this depth are the highest-priority fixes.
- **Depth + low inlinks compound.** A page at depth 5 with 1 inlink has severe discoverability problems. Filter for pages at depth 4+ with fewer than 3 unique inlinks to find the most structurally disadvantaged pages.
- **Pagination-driven depth.** Deeply paginated archives (page 20, page 50, page 100) push linked content to extreme depths. If a product is only reachable through page 47 of a category listing, its effective depth might be 50+ clicks from the homepage.
- **Compare depth across page types.** If blog posts average depth 3 but product pages average depth 5, the product section's architecture needs restructuring.

**Recommended fix:**

1. **Flatten site architecture by adding links from high-level pages.** Link directly from the homepage and top-level category pages to important deep content. A single link from the homepage reduces a page's depth to 1, regardless of where it previously sat in the hierarchy.
2. **Implement breadcrumb navigation** with structured data (BreadcrumbList schema). Breadcrumbs create a consistent internal link path from every page back through the hierarchy to the homepage, ensuring every page has a clear, short path to the top of the site.
3. **Create hub pages and topic clusters.** A hub page at depth 1-2 that links to all related content within a topic brings every spoke page to depth 2-3. This is the most scalable way to reduce depth for large content sets.
4. **Add cross-linking modules to templates.** "Related products," "popular in this category," "recently published," and "editors' picks" modules on page templates create lateral links that reduce depth by providing alternative paths to content.
5. **Restructure navigation menus.** If important page types are buried behind 4+ levels of navigation hierarchy, flatten the menu structure. Use mega-menus or category landing pages to make key sections accessible within 1-2 clicks from the homepage.
6. **Fix pagination depth.** For deeply paginated sections:
   - Implement "jump to page" links that allow crawlers to reach page 50 in 2 clicks instead of 50
   - Add sub-category or filter landing pages that break large lists into smaller, shallower sets
   - Link to specific pagination ranges from the first page (e.g., "Pages: 1, 10, 20, 30, 40, 50") to create shortcut paths
7. **Do not rely on the sitemap as a substitute.** The XML sitemap helps with discovery, but Google uses internal link structure, not sitemap presence, to evaluate page importance. A deep page in the sitemap is still perceived as less important than a shallow page with strong internal link support.

---

## Quick Reference: Severity Summary

| Issue | Default Severity | Escalates To | Condition for Escalation |
|-------|-----------------|-------------|-------------------------|
| Low Inlink Pages (< 3 inlinks) | Medium | **High** | Page is a key product, category, or landing page |
| Low Inlink Pages (0 inlinks) | **High** | **Critical** | Page is an orphan (see Crawlability reference) |
| Link Equity Distribution | Medium | -- | Optimization opportunity, not a pass/fail issue |
| Broken Internal Links | **High** | **Critical** | Broken link is in a site-wide template (nav/footer) or destination returns 5xx |
| Excessive Outlinks (> 100) | Medium | -- | Expected on navigation and hub pages (Low); problematic on content pages |
| Anchor Text (generic anchors) | Low | -- | Refinement opportunity; does not cause crawl or indexation issues |
| Link Depth 4-5 clicks | Medium | -- | Acceptable for low-priority content |
| Link Depth 6+ clicks | **High** | **Critical** | 8+ clicks from homepage; pages may not be crawled |
