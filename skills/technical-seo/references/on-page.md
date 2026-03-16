# On-Page Reference

Use this reference when analyzing crawl data related to page titles, meta descriptions, headings, canonical tags, hreflang attributes, content quality, or Open Graph tags. Each issue includes what it is, why it matters, severity classification, how to identify it in Screaming Frog crawl data, and the recommended fix.

---

## 1. Page Titles

### 1.1 Missing Page Title

**What it is:** The page has no `<title>` element in the `<head>`, or the title tag is present but empty. Search engines have nothing to use as the primary headline in search results, so they will auto-generate one from heading text, anchor text of inbound links, or other on-page content.

**Why it matters:** The title tag is the single most important on-page ranking factor. It directly tells search engines what the page is about, and it is the clickable headline in search results that determines click-through rate. A missing title means:
- The page loses its strongest relevance signal for target keywords
- Google will auto-generate a title, which is frequently inaccurate, truncated, or pulls irrelevant text (brand name, breadcrumb, random heading)
- Click-through rate drops because auto-generated titles rarely match user intent as well as a crafted title would
- The page is competing at a fundamental disadvantage against every competitor that has a proper title

**Severity:** **Critical** — a missing title is one of the most basic and impactful SEO failures. Every indexable page must have a title.

**How to identify in crawl data:**

- **Export tab:** `Page Titles:Missing`
- **Key columns:** `Address`, `Title 1`, `Indexability`, `Status Code`
- If the `Page Titles:Missing` filter is unavailable, export `Page Titles:All` and filter for rows where `Title 1` is blank or empty

**What to look for:**
- Total count of pages with missing titles — even one is noteworthy on a well-maintained site
- Filter to only indexable pages (non-indexable pages with missing titles are low priority)
- Group by URL pattern or template type: if all pages in `/products/` are missing titles, the product page template has no title tag
- Check if the issue is a missing `<title>` element entirely or a present-but-empty `<title></title>` tag — both produce the same result but have different root causes (missing tag = template issue; empty tag = CMS dynamic insertion failing)

**Recommended fix:**

1. **Add a unique, keyword-targeted title tag** to every indexable page. Titles should be 30-60 characters, include the primary keyword near the front, and accurately describe the page content.
2. **Fix at the template level:** If an entire page type is missing titles, the template is the root cause. Ensure the template dynamically generates titles from content fields (product name, article title, category name).
3. **Check CMS configuration:** Some CMS platforms require an SEO plugin (e.g., Yoast, Rank Math, All in One SEO) to generate title tags. If no plugin is installed, titles may not render.
4. **Verify after deployment:** Re-crawl the affected URLs to confirm titles are rendering correctly. Check the rendered HTML (not just the source) in case JavaScript is responsible for injecting the title.

---

### 1.2 Duplicate Page Titles

**What it is:** Two or more indexable pages share the exact same `<title>` tag content. This includes both exact duplicates (identical character-for-character) and near-duplicates (same title with trivial variations like trailing whitespace).

**Why it matters:** Duplicate titles create two problems:
- **Keyword cannibalization:** When multiple pages have the same title, search engines cannot determine which page is the best result for the title's keywords. Instead of one strong page ranking, multiple pages compete against each other, often resulting in all of them ranking lower than a single consolidated page would.
- **Poor user experience in SERPs:** If two pages from the same site appear in search results with identical titles, users cannot distinguish between them. This reduces click-through rate and perceived site quality.

Google may also choose to rewrite one or both titles to differentiate them, which removes your control over how the pages are presented.

**Severity:** **High** — duplicate titles directly cause keyword cannibalization and signal poor content differentiation.

**How to identify in crawl data:**

- **Export tab:** `Page Titles:Duplicate`
- **Key columns:** `Address`, `Title 1`, `Title 1 Length`, `Indexability`
- The export groups pages by their shared title value, making it straightforward to see which pages share each title

**What to look for:**
- Sort by the number of pages sharing each title — titles shared by 10+ pages indicate a systemic template issue
- Common patterns for false positives:
  - Paginated pages often share titles (e.g., "Blog - Page 2", "Blog - Page 3" may all show as "Blog" if the page number is not appended). This is a real issue, not a false positive.
  - Faceted navigation pages may generate unique URLs with identical titles
- Filter to only indexable pages — duplicates among noindexed pages are not a ranking concern
- Check whether the duplicated pages have distinct content despite the shared title, or if they are true content duplicates (which is a larger problem)

**Recommended fix:**

1. **Write unique titles for each page.** Every indexable page should have a title that distinctly describes its content and targets its specific keywords.
2. **Template-level fix:** If the title is generated from a template, ensure the template pulls unique data for each page:
   - Product pages: `{Product Name} | {Brand}`
   - Category pages: `{Category Name} - {Site Name}`
   - Blog posts: `{Post Title} | {Blog Name}`
3. **For paginated pages:** Append the page number — `Blog Archive - Page 2 | Site Name`.
4. **For true content duplicates:** The title duplication is a symptom; the real problem is duplicate content. Consolidate the pages with 301 redirects or canonical tags, then the title duplication resolves itself.
5. **For faceted/filtered pages:** If filtered URLs generate duplicate titles, either:
   - Make the titles unique by including the filter values (e.g., "Running Shoes - Red - Size 10")
   - Canonicalize the filtered pages to the main category page
   - Noindex the filtered pages

---

### 1.3 Page Title Too Long (Over 60 Characters)

**What it is:** The page's `<title>` tag exceeds 60 characters. Google typically displays 50-60 characters of a title in desktop search results (approximately 600 pixels wide). Titles longer than this threshold are truncated with an ellipsis ("...").

**Why it matters:** A truncated title is not an SEO ranking penalty — Google still reads and uses the full title for ranking purposes. The impact is purely on click-through rate in search results:
- Users see an incomplete title, which may cut off the most important differentiating information
- Truncation can create awkward or misleading partial phrases
- The lost characters are often the brand name or secondary keywords at the end, reducing brand visibility

However, many legitimate titles naturally exceed 60 characters (long product names, location-specific pages, titles with necessary qualifiers), and forcing them under 60 characters can make them less descriptive.

**Severity:** **Low** — no ranking impact; purely a SERP presentation optimization. Only worth addressing in bulk if click-through rate is materially suffering.

**How to identify in crawl data:**

- **Export tab:** `Page Titles:Over 60 Characters`
- **Key columns:** `Address`, `Title 1`, `Title 1 Length`, `Title 1 Pixel Width`
- For a more accurate analysis, use the `Title 1 Pixel Width` column if available — pixel width (target: under 580px) is a better proxy for truncation than character count, because characters vary in width

**What to look for:**
- Focus on pages where the title is significantly over 60 characters (70+) — titles at 61-65 characters are often fine
- Check what is being truncated: if the cut-off text is just the site name appended at the end, the impact is minimal. If the primary keyword or unique descriptor is being cut off, the impact is higher.
- Identify templates that consistently produce long titles — e.g., a pattern like "{Product Name} - {Category} - {Subcategory} | {Brand Name}" that always exceeds 60 characters

**Recommended fix:**

1. **Prioritize by impact:** Only address titles where the truncation cuts off meaningful information. A title of 63 characters that truncates the site name is not worth changing.
2. **Front-load keywords:** Put the most important keywords and differentiators at the beginning of the title so they are always visible, regardless of truncation.
3. **Shorten or remove the site/brand name suffix** if it pushes titles over 60 characters. The brand name can be removed from titles on well-established domains without meaningful ranking loss.
4. **For template-generated titles:** Adjust the template formula to produce shorter output — e.g., remove the subcategory, abbreviate common terms, or truncate long product names at a natural word boundary.
5. **Do not sacrifice descriptiveness for brevity.** A 65-character title that accurately describes the page is better than a 55-character title that is vague. Google uses the full title for ranking regardless of display truncation.

---

### 1.4 Page Title Too Short (Under 30 Characters)

**What it is:** The page's `<title>` tag is shorter than 30 characters. While there is no minimum title length enforced by search engines, very short titles typically indicate the title is not sufficiently descriptive.

**Why it matters:** A short title is often a symptom of one of these problems:
- The title is generic ("Home", "Products", "Blog") and does not include target keywords
- The title was auto-generated from a field that is naturally short (e.g., a product SKU or category code)
- The template only uses a single data field instead of constructing a descriptive title

Short titles miss the opportunity to include secondary keywords, qualifiers, and descriptive language that improve both rankings and click-through rate. A title like "Running Shoes" is technically valid but loses to "Best Running Shoes for Marathon Training 2026 | BrandName" in both relevance signaling and SERP click appeal.

**Severity:** **Low** — short titles are a missed optimization opportunity, not a technical failure.

**How to identify in crawl data:**

- **Export tab:** `Page Titles:All` — filter for `Title 1 Length < 30`
- **Key columns:** `Address`, `Title 1`, `Title 1 Length`
- There is no dedicated Screaming Frog filter for short titles; you must filter the full titles export manually

**What to look for:**
- Generic/boilerplate titles: "Home", "About", "Contact", "Products" — these miss keyword opportunities
- Single-word or two-word titles that describe a section rather than a page's specific content
- Titles that are just the brand name with no page-specific descriptor
- Aggregate by template type: if all blog post titles are under 30 characters, the title template may be pulling from the wrong field

**Recommended fix:**

1. **Expand titles to include primary and secondary keywords** along with descriptive modifiers. A good title formula is: `{Primary Keyword} - {Secondary Keyword/Qualifier} | {Brand Name}`.
2. **For homepage titles:** Replace generic "Home" with a keyword-targeted title that describes the business and primary offering — e.g., "Custom Running Shoes for Endurance Athletes | BrandName".
3. **For category pages:** Include the category name, a descriptor, and the site name — e.g., "Women's Running Shoes - Lightweight & Breathable | BrandName".
4. **Review template logic:** If the CMS is generating short titles, adjust the template to concatenate multiple content fields.
5. **Prioritize pages with search traffic potential.** Utility pages like "Contact Us" or "Privacy Policy" do not need keyword-optimized titles.

---

### 1.5 Page Title Same as H1

**What it is:** The page's `<title>` tag content is identical (or nearly identical) to the page's `<h1>` heading.

**Why it matters:** This is commonly flagged by SEO tools but is rarely a meaningful problem. In many cases, the title and H1 _should_ be similar because they both describe the same page. The concern is a missed optimization opportunity:
- The title tag is optimized for search engine results (concise, keyword-rich, includes brand)
- The H1 is optimized for on-page user experience (can be longer, more descriptive, more engaging)
- If they are identical, you are using the same text for two different purposes instead of tailoring each to its context

However, Google has stated that having the same title and H1 is perfectly fine and is the norm for most pages on the web. There is no penalty or ranking disadvantage.

**Severity:** **Low** — this is an optimization opportunity, not a problem. Many well-ranking pages have identical title and H1 tags.

**How to identify in crawl data:**

- **Export tab:** `Page Titles:All` cross-referenced with `H1:All`
- **Key columns:** `Address`, `Title 1`, `H1-1`
- Compare the `Title 1` and `H1-1` columns for each row — look for exact matches
- Screaming Frog may also flag this under `Page Titles:Same as H1` if that filter is available in the version being used

**What to look for:**
- This is only worth investigating on high-value, high-traffic pages where marginal CTR improvements matter
- Do not flag this as an issue for low-traffic utility pages, thin pages, or pages without significant search potential
- Check whether the combined title+H1 strategy is missing keyword variations — e.g., the title could target "running shoes" while the H1 targets "best shoes for running"

**Recommended fix:**

1. **Differentiate title and H1 where it adds value:**
   - **Title:** Optimize for search results — concise, primary keyword near the front, include brand name, stay under 60 characters
   - **H1:** Optimize for on-page engagement — can be more conversational, include long-tail variations, does not need the brand name
2. **Example:**
   - Title: `Best Running Shoes for Marathon Training | BrandName`
   - H1: `The Best Running Shoes to Get You Through Marathon Training in 2026`
3. **Do not force changes.** If the title and H1 are naturally the same and the page is ranking well, changing them purely to make them different is not recommended. Only differentiate when it serves a keyword or UX purpose.
4. **Priority:** This should be near the bottom of any SEO task list. Address it only after all Critical, High, and Medium issues are resolved.

---

## 2. Meta Descriptions

### 2.1 Missing Meta Description

**What it is:** The page has no `<meta name="description" content="...">` tag in the `<head>`, or the tag is present but the content attribute is empty. Search engines have no author-provided description to use as the snippet in search results.

**Why it matters:** Meta descriptions are not a ranking factor — Google has confirmed this repeatedly. They do not directly influence where a page ranks. However, they significantly impact click-through rate:
- The meta description is the snippet text shown below the title in search results (when Google chooses to use it)
- A well-written description acts as ad copy, persuading users to click
- Without a meta description, Google auto-generates a snippet by extracting text from the page content. These auto-generated snippets are often a random paragraph that may not be compelling or relevant to the search query.

Google rewrites meta descriptions approximately 60-70% of the time to better match the search query, so even pages with meta descriptions do not always display them. This further reduces the severity of missing descriptions — but when Google _does_ use the provided description, having a good one matters.

**Severity:** **Medium** — no ranking impact, but a meaningful click-through rate optimization missed across potentially every search query the page appears for.

**How to identify in crawl data:**

- **Export tab:** `Meta Description:Missing`
- **Key columns:** `Address`, `Meta Description 1`, `Indexability`, `Status Code`
- Filter to indexable pages only — noindexed pages without descriptions are not a concern

**What to look for:**
- Total count and percentage of indexable pages missing meta descriptions
- Group by template type: if all product pages are missing descriptions, the template does not include the description tag
- Prioritize pages with existing search traffic (from Google Search Console data if available) — those are the pages where CTR improvements from descriptions will have measurable impact
- Check whether the CMS or SEO plugin is configured to auto-generate descriptions from page content (some do this, some do not)

**Recommended fix:**

1. **Write unique, compelling meta descriptions** for high-traffic and high-priority pages. Descriptions should be 120-160 characters, include the target keyword (Google bolds keyword matches in snippets), and end with a clear value proposition or call to action.
2. **Template-level auto-generation:** For sites with thousands of pages, configure the CMS or SEO plugin to auto-generate descriptions from content fields:
   - Product pages: Pull from the product short description
   - Blog posts: Pull from the first paragraph or excerpt
   - Category pages: Use a template like "Browse our selection of {Category Name}. {Count} products with free shipping and returns."
3. **Prioritize by traffic:** Hand-write descriptions for the top 50-100 pages by organic traffic. Auto-generate for the rest.
4. **Do not duplicate descriptions across pages.** A shared boilerplate description is nearly as unhelpful as no description. See section 2.2.

---

### 2.2 Duplicate Meta Descriptions

**What it is:** Two or more indexable pages share the exact same meta description content.

**Why it matters:** Like duplicate titles, duplicate meta descriptions prevent users from distinguishing between pages in search results. However, because meta descriptions are not a ranking factor, the impact is limited to SERP click-through rate:
- When two pages from the same site appear in results with the same description, users see no reason to prefer one over the other
- Google is more likely to ignore (rewrite) a meta description if it detects the same description is used across many pages, because a shared description is obviously not specific to each page
- Widespread duplication signals that descriptions are boilerplate rather than crafted — which makes Google less likely to use any of them

**Severity:** **Medium** — no ranking impact, but undermines the purpose of having descriptions at all. At scale (hundreds or thousands of duplicates), it signals low-quality templating.

**How to identify in crawl data:**

- **Export tab:** `Meta Description:Duplicate`
- **Key columns:** `Address`, `Meta Description 1`, `Meta Description 1 Length`, `Indexability`
- The export groups pages sharing the same description value

**What to look for:**
- Descriptions shared by 10+ pages almost always indicate a template or CMS issue (a hardcoded default description, or a description field pulling from a site-wide setting instead of page-level content)
- Common false positive: paginated pages may legitimately share a base description. This is still worth fixing but is lower priority.
- Check whether duplicates are among pages that also have duplicate titles — if both are duplicated, these may be full content duplicates (a bigger problem)

**Recommended fix:**

1. **Write unique descriptions for high-value pages** — hand-crafted descriptions for the most important pages, auto-generated for the rest.
2. **Fix template-level duplication:** If the CMS is inserting a site-wide default description on every page, update the template to pull page-specific content. The description tag should reference a page-level field, not a global setting.
3. **Remove boilerplate descriptions entirely.** It is better to have _no_ meta description (letting Google auto-generate from content) than to have the same description on every page. A shared description provides no value and may cause Google to distrust all descriptions on the site.
4. **For large sites:** Implement auto-generation rules per template type (see 2.1 recommended fix). Even formulaic descriptions like "{Product Name} - {Category}. Shop now with free shipping." are better than site-wide boilerplate.

---

### 2.3 Meta Description Too Long (Over 160 Characters)

**What it is:** The page's meta description exceeds 160 characters. Google typically displays 150-160 characters of the description in desktop search results and somewhat less on mobile. Descriptions longer than this are truncated.

**Why it matters:** Like long titles, long descriptions are not a ranking issue. Google reads the full description regardless of display length. The concern is purely presentational:
- Truncated descriptions end with "..." which can cut off important information
- The truncated text may stop mid-sentence, creating an awkward or incomplete snippet
- On mobile, the truncation point is even shorter (~120 characters), so descriptions over 160 characters may lose significant content on mobile SERPs

However, Google rewrites descriptions for the majority of search queries anyway, so the provided description's exact length is only relevant when Google chooses to use it.

**Severity:** **Low** — no ranking impact; minor SERP presentation issue that only applies when Google uses the provided description.

**How to identify in crawl data:**

- **Export tab:** `Meta Description:All` — filter for `Meta Description 1 Length > 160`
- **Key columns:** `Address`, `Meta Description 1`, `Meta Description 1 Length`
- There may be a dedicated filter like `Meta Description:Over 160 Characters` depending on the Screaming Frog version

**What to look for:**
- Descriptions significantly over 160 characters (200+) — these are almost certainly being truncated in every SERP appearance
- Check what is being truncated: if the first 155 characters form a complete, compelling snippet, the extra length is harmless
- Template-generated descriptions that concatenate multiple fields and consistently produce long output

**Recommended fix:**

1. **Trim descriptions to 150-155 characters** to provide a buffer against truncation on both desktop and mobile.
2. **Front-load the most important information** so that even if truncated, the visible text is compelling and complete.
3. **Write to natural sentence boundaries.** Ensure the description reads well at the truncation point — end a sentence or clause before 155 characters, even if there is room for more.
4. **Priority:** This is a low-impact optimization. Address it when doing a meta description pass for other reasons (missing or duplicate descriptions), not as a standalone project.

---

### 2.4 Meta Description Too Short (Under 70 Characters)

**What it is:** The page's meta description is shorter than 70 characters. While there is no minimum length requirement, very short descriptions underutilize the available SERP real estate.

**Why it matters:** The meta description snippet is an opportunity to persuade users to click. A short description:
- Wastes valuable SERP space that could be used to communicate value, differentiate from competitors, or include additional keywords
- May look sparse or incomplete compared to competitor listings with full-length descriptions
- Often indicates the description is generic or auto-generated from a short field (e.g., a product tagline or category name alone)

Google is also more likely to replace a very short description with auto-generated content from the page, since the provided description may not give users enough information.

**Severity:** **Low** — missed optimization opportunity; no ranking impact.

**How to identify in crawl data:**

- **Export tab:** `Meta Description:All` — filter for `Meta Description 1 Length < 70`
- **Key columns:** `Address`, `Meta Description 1`, `Meta Description 1 Length`

**What to look for:**
- Descriptions that are just the site name, a single sentence fragment, or a repeated tagline
- Template-generated descriptions pulling from a short content field (product tagline, category code)
- Pages where the short description is contextually appropriate (e.g., a utility page like "Contact Us") — these do not need long descriptions

**Recommended fix:**

1. **Expand descriptions to 120-155 characters** to fully utilize the available SERP space.
2. **Include the primary keyword, a supporting detail, and a call to action** — e.g., "Shop lightweight running shoes built for marathon distance. Free shipping on all orders over $75. Browse 200+ styles."
3. **Fix template auto-generation** if it is pulling from a short field. Configure it to concatenate multiple fields or add template text around the dynamic content.
4. **Priority:** Address alongside missing and duplicate descriptions as part of a meta description audit, not as an independent task.

---

## 3. Headings (H1)

### 3.1 Missing H1

**What it is:** The page does not contain an `<h1>` heading element. The H1 is the primary heading of the page and serves as the top-level content label for both users and search engines.

**Why it matters:** The H1 is a significant on-page relevance signal — not as strong as the title tag, but more important than all other heading levels (H2-H6). A missing H1:
- Removes a key relevance signal that reinforces the title tag's keywords
- Makes the page's content hierarchy unclear to search engines (what is this page about?)
- Hurts accessibility — screen readers use heading structure to navigate content, and the H1 is the expected starting point
- May indicate a broader template issue where semantic HTML is not being used (divs styled to look like headings instead of actual heading elements)

**Severity:** **High** — the H1 is a core on-page element, and its absence signals poor content structure that affects both rankings and accessibility.

**How to identify in crawl data:**

- **Export tab:** `H1:Missing`
- **Key columns:** `Address`, `H1-1`, `Indexability`, `Status Code`, `Title 1`
- Filter to indexable, 200-status pages — H1 issues on redirected, erroring, or noindexed pages are not a priority

**What to look for:**
- Total count and percentage of indexable pages missing H1s
- Group by URL pattern or template: if all pages of a specific type are missing H1s, the template needs to be updated
- Check if the page has visible heading text that is styled to look like a heading but is not in an `<h1>` tag (e.g., a `<div class="page-title">` or a `<span>` with large font styling). This is a common implementation error.
- Check if the page uses `<h2>` as its top-level heading with no `<h1>` — this indicates the heading hierarchy starts at the wrong level

**Recommended fix:**

1. **Add an H1 tag to every indexable page.** The H1 should describe the page's primary topic and include the primary target keyword (naturally, not forced).
2. **Fix at the template level:** Identify which page template is missing the H1 and add it. The H1 should be the first heading in the main content area, typically above the primary content body.
3. **Convert styled text to semantic headings:** If the page has text that visually looks like a heading but uses `<div>`, `<span>`, or `<p>` with CSS styling, change it to a proper `<h1>` element.
4. **Ensure the H1 is in the HTML source,** not injected via JavaScript — search engines process rendered DOM, but relying on JS for such a fundamental element introduces unnecessary risk.
5. **Do not use the logo or site name as the H1** (a common pattern on homepages). The H1 should describe the page's content, not the site's brand. Use a separate, content-descriptive H1 for the homepage.

---

### 3.2 Multiple H1s

**What it is:** The page contains more than one `<h1>` heading element. HTML5 technically allows multiple H1s within distinct sectioning elements (`<article>`, `<section>`), but from an SEO perspective, the best practice is a single H1 per page.

**Why it matters:** Multiple H1s dilute the relevance signal. When a page has one H1, search engines treat it as the definitive topic declaration. With multiple H1s, the signal is split across multiple headings, and search engines must decide which one represents the page's primary topic. Practically:
- The ranking impact is generally small — Google's John Mueller has said multiple H1s are "not a problem" and Google can handle them. However, "can handle" is not the same as "optimal."
- It often indicates a structural issue: a CMS or theme adding an H1 for the site logo/name in the header template AND an H1 for the page content, resulting in every page having two H1s
- It can confuse accessibility tools that expect a single H1 as the page's primary landmark

**Severity:** **Medium** — not a ranking emergency, but a structural issue worth fixing, especially if it is template-wide.

**How to identify in crawl data:**

- **Export tab:** `H1:Multiple`
- **Key columns:** `Address`, `H1-1`, `H1-2`, `Indexability`
- The `H1-2` column (and `H1-3` if present) shows the additional H1 values

**What to look for:**
- Check the `H1-1` and `H1-2` values: is one of them the site name/logo text? This is the most common cause — the header template wraps the logo in an `<h1>` tag.
- Count how many pages have multiple H1s — if it is site-wide, it is almost certainly a template/theme issue
- Check for pages with 3+ H1s — this is more likely to cause confusion than 2 H1s
- Review the content: are the multiple H1s used intentionally for distinct content sections (e.g., a comparison page with two products), or accidentally from poor template structure?

**Recommended fix:**

1. **Reduce to a single H1 per page.** The H1 should be the page's primary content heading.
2. **If the site logo/name is wrapped in an H1:** Change it to a `<div>`, `<span>`, or `<p>`. The logo is site-wide chrome, not page-specific content. If it needs to be a heading for accessibility, use a visually hidden H1 for the page content and make the logo a non-heading element.
3. **Demote secondary H1s to H2:** Any heading that was an H1 but is not the primary page heading should become an H2 (or lower, depending on the content hierarchy).
4. **Fix at the theme/template level:** This is almost always a one-time template change that fixes the issue site-wide.
5. **In HTML5 sectioning contexts:** If the page intentionally uses `<article>` or `<section>` elements with their own H1s, this is valid HTML5 but still suboptimal for SEO. Consider demoting sectioned H1s to H2 for cleaner signaling.

---

### 3.3 Duplicate H1s Across Pages

**What it is:** Two or more pages share the exact same H1 heading text. This is distinct from multiple H1s on a single page (3.2) — this is the same H1 text used on different pages.

**Why it matters:** Duplicate H1s suggest content overlap or poor differentiation between pages:
- If two pages have the same H1, they may be targeting the same topic, which leads to keyword cannibalization
- A shared H1 is often a symptom of template-generated headings that do not pull unique data (e.g., all product pages showing "Product Details" as the H1 instead of the actual product name)
- It reduces the reinforcement value of the H1 — each page should have a heading that distinctly aligns with its unique title and content

**Severity:** **Medium** — indicates content differentiation issues, but the ranking impact of H1 duplication alone is smaller than title duplication.

**How to identify in crawl data:**

- **Export tab:** `H1:Duplicate`
- **Key columns:** `Address`, `H1-1`, `Title 1`, `Indexability`
- The export groups pages sharing the same H1 text

**What to look for:**
- H1s shared by 10+ pages almost always indicate a hardcoded template heading (e.g., "Welcome", "Products", "Blog")
- Cross-reference with titles: if both the title and H1 are duplicated across the same set of pages, the content differentiation problem is more severe
- Filter to indexable pages only
- Common legitimate duplicates to deprioritize: paginated pages may share H1s ("Blog"), and this is expected (though adding "Page X" is better)

**Recommended fix:**

1. **Make every indexable page's H1 unique** and descriptive of that page's specific content.
2. **Fix template-generated H1s** to pull from page-specific content fields:
   - Product pages: `{Product Name}`
   - Category pages: `{Category Name}`
   - Blog posts: `{Post Title}`
3. **If pages have duplicate H1s AND duplicate content:** The H1 duplication is a symptom. Address the content duplication first via canonicalization, consolidation, or differentiation.
4. **For paginated sections:** Add context to the H1 — "Blog" becomes "Blog - Page 3" or "Blog Archive: Posts 21-30".

---

## 4. Canonical Tags

### 4.1 Missing Canonical Tag

**What it is:** The page does not include a `<link rel="canonical" href="...">` tag in the `<head>` and does not specify a canonical via the HTTP `Link` header. No canonical signal is provided to search engines.

**Why it matters:** The canonical tag tells search engines which URL is the "official" version of the content. Without it:
- Search engines must determine the canonical URL on their own using signals like internal links, sitemap presence, and redirect patterns. Their choice may not match your intent.
- Duplicate content issues are harder to resolve — if a page is accessible at multiple URLs (with/without trailing slash, with/without www, with query parameters), the canonical tag is the primary mechanism for consolidating them.
- Link equity may be split across multiple URL versions instead of consolidated to a single canonical.

That said, a missing canonical is not catastrophic. Google generally does a reasonable job of inferring canonicals, and most sites only run into problems when pages are accessible at many URL variants.

**Severity:** **Medium** — the impact depends on how many URL variants exist for each page. Sites with clean URL structures and no duplicate content may see minimal effect. Sites with query parameters, trailing slash variations, or multiple access paths will see greater impact.

**How to identify in crawl data:**

- **Export tab:** `Canonicals:All` — filter for rows where `Canonical Link Element 1` is blank/empty
- **Key columns:** `Address`, `Canonical Link Element 1`, `Indexability`, `Status Code`
- Also check the HTTP headers: some sites implement canonicals via the `Link` HTTP header instead of the HTML element. Screaming Frog captures this separately.

**What to look for:**
- Percentage of indexable pages missing canonicals — a high percentage suggests the site does not implement canonicals at all
- Check by template type: which page templates include canonical tags and which do not?
- Cross-reference with URL variants: are there pages accessible at multiple URLs (e.g., `/page` and `/page/` and `/page?ref=xyz`) that lack canonical tags? These are the highest-priority fixes.
- Check if the site uses a CMS or SEO plugin that should be adding canonicals automatically — the plugin may be misconfigured

**Recommended fix:**

1. **Add self-referencing canonical tags to every indexable page.** A self-referencing canonical (`<link rel="canonical" href="https://www.example.com/current-page/">`) is the baseline best practice.
2. **Implement at the template level** so every page template includes a canonical tag pointing to the page's own URL.
3. **Use absolute URLs** in canonical tags, not relative URLs. Include the protocol (`https://`) and the preferred domain (`www` or non-`www`).
4. **Ensure consistency:** The canonical URL should match the URL in the sitemap, the internal links, and the preferred URL format (trailing slash or no trailing slash — pick one and be consistent).
5. **For CMS sites:** Install or configure an SEO plugin that auto-generates self-referencing canonicals. Most major CMS platforms have this capability built in or available via plugin.

---

### 4.2 Canonical Pointing to Non-200 Page

**What it is:** The page's canonical tag points to a URL that does not return a 200 status code. The canonical target may be a 301/302 redirect, a 404 error, a 5xx error, or any other non-200 status.

**Why it matters:** The canonical tag says "the official version of this content lives at [URL]." If that URL is broken:
- **Canonical to 404:** Search engines are told the canonical is a page that does not exist. This creates a paradox — the current page may be deindexed because the canonical says "index the other URL instead", but the other URL cannot be indexed because it is a 404. The result is often that neither URL is indexed.
- **Canonical to 301/302:** Search engines must follow the redirect to find the actual page, adding an unnecessary extra step. The redirect target becomes the effective canonical, but this indirect chain is fragile and may not be followed correctly in all cases.
- **Canonical to 5xx:** The canonical target is temporarily or permanently broken, preventing search engines from confirming the canonical relationship.

This is one of the most damaging canonical misconfigurations because it can directly cause pages to be dropped from the index.

**Severity:** **Critical** — canonicals pointing to non-200 URLs can cause indexation loss for the current page.

**How to identify in crawl data:**

- **Export tab:** `Canonicals:All`
- **Key columns:** `Address`, `Canonical Link Element 1`, `Status Code` (of the canonical target), `Indexability`
- Cross-reference the canonical target URLs against the main crawl data to find their status codes
- Look for canonical targets that were not crawled (they may be external URLs or URLs excluded from the crawl scope)

**What to look for:**
- Any page where the canonical target returns a non-200 status code — every instance needs investigation
- Group by canonical target: if many pages canonical to the same broken URL, fixing that one URL resolves all of them
- Check if the canonical targets were recently changed (migration, URL restructure) — this is the most common cause
- Canonical targets that are redirects: the redirect destination should be the canonical target instead

**Recommended fix:**

1. **Canonical to 404/410:** Either:
   - Update the canonical tag to point to the current page (self-referencing canonical) if the current page is the correct version
   - Update the canonical tag to point to a different, live (200) page if a valid canonical target exists
   - If the page itself should not exist, redirect it or return a proper 404 instead of canonicalizing to a broken URL
2. **Canonical to redirect:** Update the canonical tag to point directly to the redirect's final destination URL (the 200-status URL at the end of the redirect chain). Never canonical to a URL that redirects.
3. **Canonical to 5xx:** This is likely a temporary server issue. Verify whether the target URL is permanently broken or intermittently failing. If permanent, update the canonical. If temporary, fix the server issue.
4. **Post-migration audit:** After any URL migration, audit all canonical tags site-wide. Bulk find-and-replace old URLs with new URLs in canonical tags.
5. **Validate:** After fixing, re-crawl to confirm all canonical targets return 200.

---

### 4.3 Conflicting Canonical and Noindex

**What it is:** The page has both a `<meta name="robots" content="noindex">` directive and a canonical tag pointing to a different URL. These two signals send contradictory messages:
- The `noindex` tag says "do not index this page"
- The canonical tag pointing to another URL says "the canonical version of this content lives at [other URL] — index that one instead"

While both signals individually suggest "do not index the current URL", the combination is ambiguous because the canonical implies the content should exist elsewhere, while `noindex` says the content should not be in the index at all.

**Why it matters:** Google has stated that when `noindex` and canonical conflict, it generally honors the `noindex` (the more restrictive directive), but behavior is not guaranteed to be consistent. Specific risks:
- Google may follow the canonical and ignore the `noindex`, indexing the canonical target but also keeping the current page accessible in some contexts
- Google may honor the `noindex` and also ignore the canonical, meaning the intended canonical consolidation does not happen
- The conflicting signals waste crawl attention — Google must process both signals and decide which to follow
- It often indicates a configuration error where two different systems (CMS adds canonical, plugin adds noindex) are acting independently without coordination

**Severity:** **High** — the conflicting signals create unpredictable indexation behavior and indicate a configuration problem that should be resolved.

**How to identify in crawl data:**

- **Export tab:** `Directives:All` cross-referenced with `Canonicals:All`
- **Key columns:** `Address`, `Meta Robots 1`, `Canonical Link Element 1`, `Indexability`, `Indexability Status`
- Filter for pages where `Meta Robots 1` contains `noindex` AND `Canonical Link Element 1` is not empty AND the canonical points to a different URL (not self-referencing)
- A self-referencing canonical + noindex is less problematic (the canonical is simply redundant on a noindexed page)

**What to look for:**
- Pages where noindex is intentional but the canonical is leftover from a previous configuration
- Pages where the canonical is intentional but a plugin/setting added noindex unintentionally
- Template-level conflicts: does the page template always add a canonical, even on pages that are noindexed?
- Check the canonical target: is it indexable and returning 200? If so, the intent may be "don't index this page, but consolidate signals to the canonical target" — which should be handled with a 301 redirect instead

**Recommended fix:**

1. **Determine the intent for each page:** Should this page be indexed (remove noindex) or not (remove/update canonical)?
2. **If the page should NOT be indexed and content exists at the canonical URL:**
   - Remove the canonical tag (it is unnecessary on a noindex page and creates confusion)
   - Keep the `noindex` tag
   - Alternatively, 301 redirect the noindexed page to the canonical target — this is cleaner than noindex + canonical and achieves the same result
3. **If the page should NOT be indexed and the canonical was added in error:**
   - Remove the canonical tag
   - Keep the `noindex` tag
4. **If the page SHOULD be indexed:**
   - Remove the `noindex` tag
   - Ensure the canonical tag is correct (self-referencing or pointing to the preferred URL)
5. **Audit template-level configurations** to ensure canonical tags and robots directives are managed by the same system or are at least aware of each other.

---

## 5. Hreflang

### 5.1 Missing Return Tags

**What it is:** Hreflang annotations require bidirectional confirmation. If page A declares page B as its French (`fr`) equivalent with `<link rel="alternate" hreflang="fr" href="B">`, then page B must also declare page A as its corresponding language variant. When page B does not include a return hreflang tag pointing back to page A, the annotation is incomplete and may be ignored.

**Why it matters:** Google treats hreflang as a set of mutual agreements between pages. If the return tag is missing, Google may:
- Ignore the hreflang annotation entirely for that language pair — the intended language targeting will not work
- Show the wrong language version in search results for users in specific regions
- Cause the correct language version to compete against the wrong version for local queries (e.g., the English page ranking in France instead of the French page)

Missing return tags are the most common hreflang implementation error and one of the most impactful, because they silently invalidate what appears to be a correct implementation from one side.

**Severity:** **High** — missing return tags can completely negate hreflang implementation for affected language pairs, causing incorrect language versions to rank in target markets.

**How to identify in crawl data:**

- **Export tab:** `Hreflang:All`
- **Key columns:** `Address`, hreflang attribute columns (varies by Screaming Frog version — typically one column per declared hreflang value)
- Screaming Frog specifically flags "Missing Reciprocal Hreflang" or "Non-Reciprocal Hreflang" in its hreflang audit
- **Cross-reference:** For each page's hreflang targets, verify the target page's hreflang annotations include a return link

**What to look for:**
- Pages declaring hreflang targets that do not link back — these are the broken relationships
- One-directional patterns: often one language version is correctly configured (e.g., the English site) but the other versions (e.g., the French, German, Spanish sites) were not updated
- New language versions added to one site but not reciprocated on other language versions — common after launching a new locale
- Verify that both the source and target pages are accessible (200 status) and indexable — a return tag on a 404 or noindexed page does not count

**Recommended fix:**

1. **Add return hreflang tags** to every page that is referenced by another page's hreflang annotation. Every hreflang relationship must be bidirectional.
2. **Implement hreflang via a centralized system** to ensure consistency:
   - XML sitemap hreflang (recommended for large sites) — all language mappings are defined in one place, reducing the chance of missing return tags
   - HTTP headers (for non-HTML resources like PDFs)
   - HTML `<link>` tags (most common but hardest to keep synchronized across multiple sites/CMSs)
3. **Include self-referencing hreflang:** Every page should include an hreflang tag pointing to itself. This is required and confirms the page's own language/region identity.
4. **Audit after any new language launch** — when adding a new locale, every existing locale must be updated to include hreflang entries for the new one, and the new locale must reference all existing ones.
5. **Automate where possible.** Manual hreflang management across multiple sites is error-prone. Use a CMS plugin, a translation management system, or sitemap-based hreflang to reduce human error.

---

### 5.2 Missing x-default

**What it is:** The hreflang annotation set does not include an `x-default` value. The `x-default` hreflang tells search engines which page to show when no other hreflang value matches the user's language or region — it is the fallback.

**Why it matters:** Without `x-default`:
- Search engines have no explicit fallback for users whose language/region does not match any declared hreflang value
- Google must infer which page to show for unmatched regions, which may not be the page you prefer
- Users in unlisted regions may see a random language version or the strongest-ranking version, which may not be appropriate

The impact varies by site. Sites targeting a small number of well-defined markets (e.g., English US + Spanish Mexico) will see less impact from missing `x-default` than sites targeting many regions with gaps (e.g., targeting EN-US, EN-GB, FR-FR but not EN-AU or FR-CA).

**Severity:** **Medium** — missing `x-default` creates ambiguity for non-targeted regions but does not break hreflang for targeted regions.

**How to identify in crawl data:**

- **Export tab:** `Hreflang:All`
- Look for pages that have hreflang annotations for specific languages but no `x-default` entry
- Screaming Frog may flag "Missing x-default Hreflang" or similar in its hreflang report
- Check systematically: if ANY page on the site has hreflang tags but no `x-default`, it should be flagged

**What to look for:**
- Is `x-default` missing site-wide (not implemented at all) or only on specific pages (implementation inconsistency)?
- Which page _should_ be the `x-default`? Typically this is:
  - The English-language version (most common)
  - A language/region selector page
  - The most internationally relevant version of the content
- Check whether pages that should be the `x-default` are indexable and return 200

**Recommended fix:**

1. **Add `x-default` hreflang to every page that has hreflang annotations.** The `x-default` URL should be the version you want users in non-targeted regions to see.
2. **Common `x-default` strategies:**
   - **International English version:** `<link rel="alternate" hreflang="x-default" href="https://www.example.com/page/">` — the English site serves as the default
   - **Language selector page:** `<link rel="alternate" hreflang="x-default" href="https://www.example.com/choose-country/">` — redirects users to choose their region
   - **Most globally relevant version:** Whatever version is most useful for users without a specific language/region match
3. **Ensure the `x-default` page is included in all return tags** — like any other hreflang entry, it must be reciprocal.
4. **Implement alongside all other hreflang tags** — `x-default` is not optional in any well-formed hreflang implementation.

---

### 5.3 Invalid Language Codes

**What it is:** The hreflang annotation uses language or region codes that do not conform to the ISO 639-1 (language) and ISO 3166-1 alpha-2 (region) standards. Examples of invalid codes: `hreflang="en-uk"` (should be `en-gb`), `hreflang="esp"` (should be `es`), `hreflang="english"` (should be `en`).

**Why it matters:** Search engines only recognize ISO-standard language and region codes. Any hreflang tag with an invalid code is completely ignored — as if it did not exist. This means:
- The language targeting for that invalid entry does not function at all
- Users in the intended region will not be directed to the correct page
- The site owner may believe hreflang is working (the tags are present in the HTML) when it is actually doing nothing for those entries

Invalid codes are a silent failure — there is no browser error, no visible symptom. Only crawl data analysis or manual inspection reveals the problem.

**Severity:** **High** — invalid codes cause complete failure of hreflang for the affected language/region pairs, with no visible indication that the implementation is broken.

**How to identify in crawl data:**

- **Export tab:** `Hreflang:All`
- Screaming Frog flags "Incorrect Hreflang Language & Region Codes" or similar in its hreflang analysis
- **Key columns:** Review the hreflang attribute values for non-standard codes
- Common invalid patterns:
  - Three-letter language codes (`eng`, `fra`, `deu`) instead of two-letter (`en`, `fr`, `de`)
  - `en-uk` instead of `en-gb` (UK is not a valid ISO 3166-1 code; GB is)
  - Full language names (`english`, `french`) instead of codes
  - Region without language (`us`, `gb`) instead of language-region (`en-us`, `en-gb`)
  - Non-existent region codes or deprecated codes

**What to look for:**
- List all unique hreflang values across the site and validate each against ISO 639-1 (language) and ISO 3166-1 alpha-2 (region)
- The most common error is `en-uk` — it appears correct intuitively but is invalid (the correct code for the United Kingdom is `gb`)
- Check for mixed formats: some hreflang tags using underscores (`en_US`) instead of hyphens (`en-us`) — hreflang requires hyphens
- Verify that language-only codes (e.g., `en`, `fr`) are used for language targeting without region specificity, and language-region codes (e.g., `en-us`, `fr-ca`) are used for region-specific targeting

**Recommended fix:**

1. **Replace all invalid codes with correct ISO standard codes.** Reference:
   - Languages: [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) — always two letters, lowercase
   - Regions: [ISO 3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) — always two letters, uppercase (but case does not matter in hreflang)
2. **Common corrections:**

   | Invalid | Correct | Reason |
   |---------|---------|--------|
   | `en-uk` | `en-gb` | United Kingdom = GB in ISO 3166-1 |
   | `eng` | `en` | ISO 639-1 uses 2-letter codes |
   | `esp` | `es` | Spanish = `es` in ISO 639-1 |
   | `en_us` | `en-us` | Hreflang uses hyphens, not underscores |
   | `english` | `en` | Full names are not valid |
   | `jp` | `ja` | Japanese language code is `ja`, not `jp` (JP is the country code) |

3. **Validate programmatically:** Before deploying hreflang, validate all codes against the ISO standards. Build validation into the CMS or deployment pipeline.
4. **After fixing:** Verify that return tags on all target pages also use the corrected codes.

---

## 6. Content

### 6.1 Thin Content / Low Word Count

**What it is:** The page has a very low word count, indicating it contains little substantive text content. There is no universal threshold — what counts as "thin" depends entirely on the page's purpose. A product page with 100 words and strong images may be adequate; a guide or article page with 100 words is almost certainly thin.

**Why it matters:** Google's quality guidelines emphasize that pages should provide sufficient value to justify their existence in the index. Thin content pages:
- Offer little relevance signal for search engines to match against queries — there simply are not enough words to establish topical depth
- Risk being classified as "thin content" in Google's quality evaluations, which can affect site-wide quality perception (particularly after Helpful Content updates)
- Often provide a poor user experience — users who land on a thin page from search typically bounce quickly
- Accumulation of many thin content pages dilutes overall site quality and can trigger algorithmic quality penalties at scale

**Severity:**

| Page Type | Severity | Rationale |
|-----------|----------|-----------|
| Content pages (articles, guides, blog posts) with < 300 words | **Medium** | Content pages are expected to provide substantive information; low word count indicates the content is insufficient |
| Product pages with < 50 words and no supplementary media | **Medium** | Product pages need at minimum a description, specs, and supporting content |
| Product pages with < 50 words but rich media (images, video, specs tables) | **Low** | Word count is not the only quality signal; rich media content compensates |
| Utility pages (contact, login, cart, thank-you) | **Low** | These pages are not meant to rank and naturally have low word counts |
| Category/listing pages with < 50 words (excluding product listings) | **Low** | Category pages derive value from their linked items more than body text, though introductory text improves relevance |

**How to identify in crawl data:**

- **Export tab:** `Internal:All`
- **Key columns:** `Address`, `Word Count`, `Indexability`, `Status Code`, `Title 1`, `Content Type`
- Filter for indexable, 200-status pages and sort by `Word Count` ascending
- Apply different thresholds based on page type (URL pattern analysis):
  - `/blog/`, `/articles/`, `/guides/`: flag under 300 words
  - `/products/`, `/items/`: flag under 50 words
  - `/category/`, `/collections/`: informational — check for descriptive text presence

**What to look for:**
- Pages with zero or near-zero word count (under 10 words) that return 200 and are indexable — these may be soft 404s (see crawlability reference section 1.3)
- Large groups of pages with identical very low word counts — this indicates a template rendering with no dynamic content (empty template shells)
- Content pages that were started but never completed (stub articles, placeholder posts)
- Compare against competitors: if competitor pages for the same topic have 1,500 words and yours has 200, the content depth gap is a ranking factor

**Recommended fix:**

1. **For content pages (articles, guides):** Expand the content to adequately cover the topic. This does not mean artificially inflating word count — it means ensuring the page provides comprehensive, useful information that satisfies search intent. Aim for the depth required by the topic, not a word count target.
2. **For product pages:** Add descriptive content: product descriptions, specifications, use cases, FAQs, customer reviews, comparison information. Even 150-250 words of unique product description significantly improves relevance.
3. **For category pages:** Add introductory/descriptive text above or below the product listings — 100-200 words describing the category, what products it contains, and buying guidance.
4. **For true stub/placeholder pages:** Either complete the content or remove the page (404/410) and remove it from the sitemap. Do not leave empty pages indexed.
5. **Do not inflate content artificially.** Adding filler text to hit a word count target is counterproductive. Google's algorithms are increasingly sophisticated at detecting low-quality padding. Quality and relevance matter more than word count.
6. **Consolidate thin pages:** If there are many thin pages covering similar sub-topics, consider merging them into a single comprehensive page. Five thin pages of 100 words each are less valuable than one well-structured page of 800 words.

---

## 7. Open Graph Tags

### 7.1 Missing Open Graph Tags

**What it is:** The page does not include Open Graph (`og:`) meta tags. Open Graph tags control how the page appears when shared on social media platforms (Facebook, LinkedIn, Twitter/X, Slack, Discord, and others). The key tags are:
- `og:title` — the title displayed in the social card
- `og:description` — the description displayed in the social card
- `og:image` — the image displayed in the social card
- `og:url` — the canonical URL for the social share
- `og:type` — the content type (article, website, product, etc.)

**Why it matters:** Open Graph tags have **no impact on search engine rankings**. Google does not use OG tags as ranking signals, and their absence will not affect organic search performance. The impact is entirely on social media presentation:
- Without OG tags, social platforms auto-generate preview cards by scraping the page, often producing poor results: wrong image (or no image), truncated or irrelevant description, ugly formatting
- Pages shared without a compelling preview card receive dramatically fewer clicks — social CTR can drop by 50% or more compared to a well-formatted card with an eye-catching image
- Brand perception suffers when shared links look broken or unpolished
- For content-marketing-driven sites where social sharing is a significant traffic source, missing OG tags directly impact referral traffic volume

**Severity:** **Low** — no search ranking impact. The relevance of this issue depends entirely on whether social media is a meaningful traffic channel for the site.

**How to identify in crawl data:**

- Screaming Frog does not have a dedicated Open Graph export tab by default. OG tags can be extracted via custom extraction or by checking the rendered HTML.
- **Custom extraction approach:** Configure Screaming Frog to extract `og:title`, `og:description`, `og:image` via CSS selectors or XPath before crawling. The extracted values will appear in the `Custom Extraction` tab.
- **Alternative:** Spot-check pages manually using browser developer tools, Facebook's Sharing Debugger (https://developers.facebook.com/tools/debug/), or Twitter's Card Validator.
- **Key indicators without custom extraction:** If the site's shared links on social media consistently show poor previews (no image, wrong title), OG tags are likely missing.

**What to look for:**
- Is the issue site-wide (no OG tags on any page) or template-specific (blog posts have OG tags but product pages do not)?
- Are `og:image` tags present? The image is the most impactful OG element — a share without an image gets dramatically less engagement than one with a compelling image.
- Check that `og:image` URLs are absolute and point to images that meet minimum dimensions (1200x630px for Facebook, 1200x675px for Twitter/X).
- Verify OG tags render in the HTML — if they are injected via JavaScript, some social platform crawlers may not execute JS and will miss them.

**Recommended fix:**

1. **Add OG tags to every publicly shareable page.** At minimum, include `og:title`, `og:description`, `og:image`, and `og:url`.
2. **Implement at the template level:**
   - `og:title`: Pull from the page title or a dedicated social title field
   - `og:description`: Pull from the meta description or a dedicated social description field
   - `og:image`: Pull from the page's featured image, hero image, or product image. Provide a site-wide default fallback image for pages without a specific image.
   - `og:url`: Use the canonical URL
3. **Image requirements:**
   - Minimum 1200x630px for optimal display across platforms
   - Use a 1.91:1 aspect ratio
   - File size under 8MB
   - Format: JPG or PNG (avoid SVG — most social platforms do not support it)
4. **Add Twitter/X-specific tags** if Twitter is a significant channel:
   - `<meta name="twitter:card" content="summary_large_image">`
   - `<meta name="twitter:site" content="@YourHandle">`
5. **Validate after implementation** using Facebook's Sharing Debugger and Twitter's Card Validator. These tools show exactly what the social card will look like and flag any errors.
6. **Priority:** Implement OG tags during template development or as part of a general site improvement pass. Do not prioritize this over any Critical, High, or Medium SEO issues.

---

## Quick Reference: Severity Summary

| Issue | Default Severity | Escalates To | Condition for Escalation |
|-------|-----------------|-------------|-------------------------|
| Missing Page Title | **Critical** | — | Always critical for indexable pages |
| Duplicate Page Titles | **High** | — | — |
| Title Too Long (>60 chars) | Low | — | — |
| Title Too Short (<30 chars) | Low | — | — |
| Title Same as H1 | Low | — | — |
| Missing Meta Description | **Medium** | — | — |
| Duplicate Meta Descriptions | **Medium** | — | — |
| Meta Description Too Long (>160 chars) | Low | — | — |
| Meta Description Too Short (<70 chars) | Low | — | — |
| Missing H1 | **High** | — | Always high for indexable pages |
| Multiple H1s | **Medium** | — | — |
| Duplicate H1s Across Pages | **Medium** | — | — |
| Missing Canonical | **Medium** | — | — |
| Canonical to Non-200 Page | **Critical** | — | Always critical |
| Conflicting Canonical + Noindex | **High** | — | — |
| Hreflang Missing Return Tags | **High** | — | — |
| Hreflang Missing x-default | **Medium** | — | — |
| Hreflang Invalid Language Codes | **High** | — | — |
| Thin Content (content pages) | **Medium** | — | — |
| Thin Content (utility pages) | Low | — | — |
| Missing Open Graph Tags | Low | — | No ranking impact; social presentation only |
