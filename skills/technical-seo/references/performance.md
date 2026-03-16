# Performance Reference

Use this reference when analyzing crawl data related to page response time (TTFB), page size, large images, or redirect latency. Each issue includes what it is, why it matters, severity classification, how to identify it in Screaming Frog crawl data, and the recommended fix.

**Important limitation:** Screaming Frog captures server-side performance metrics -- response time (TTFB) and page/resource size. It does not measure Core Web Vitals (Largest Contentful Paint, Cumulative Layout Shift, Interaction to Next Paint) because those require a real browser rendering in a user-like environment. For Core Web Vitals data, use Google Search Console (Core Web Vitals report) or PageSpeed Insights (per-URL analysis). The metrics in this reference are complementary: a page with a 3-second TTFB will inevitably fail LCP thresholds, so server-side metrics from crawl data are a valid starting point for diagnosing performance problems. But passing the thresholds here does not guarantee passing Core Web Vitals.

---

## 1. Response Time (TTFB)

**What it is:** Time to First Byte (TTFB) measures the elapsed time between the crawler sending an HTTP request and receiving the first byte of the response from the server. This includes DNS resolution, TCP connection, TLS handshake, and server processing time. In Screaming Frog, this is recorded in the `Response Time` column in milliseconds or seconds (depending on version and configuration).

**Why it matters:** TTFB is the foundation of every other speed metric. Nothing renders, paints, or becomes interactive until the server responds. A high TTFB adds a fixed delay to every downstream metric -- if TTFB is 2 seconds, LCP cannot possibly be under 2 seconds, and Google's "Good" LCP threshold is 2.5 seconds. This means a 2-second TTFB leaves only 500ms for the entire render pipeline, which is nearly impossible on complex pages.

Google has confirmed page speed as a ranking signal since 2018 (Speed Update for mobile) and reinforced it with the Page Experience update in 2021. While TTFB alone is not a direct ranking factor, it is the single largest controllable contributor to page speed. Slow TTFB also reduces crawl efficiency: Googlebot allocates a crawl time budget per domain, and slow-responding pages mean fewer pages crawled per session.

**Severity:**

| Condition | Severity | Rationale |
|-----------|----------|-----------|
| TTFB > 2 seconds | **High** | Almost certainly causes LCP failure; Googlebot crawl rate reduction likely; poor user experience with visible page load delay |
| TTFB 1-2 seconds | **Medium** | Leaves minimal headroom for client-side rendering; at risk of LCP failure depending on page complexity; noticeable delay for users on mobile |
| TTFB < 1 second | **Acceptable** | Within normal range; further optimization is beneficial but not urgent |

**How to identify in crawl data:**

- **Export tab:** `Internal:All`
- **Key columns:** `Address`, `Response Time`, `Status Code`, `Content Type`, `Size`
- Sort by `Response Time` descending to surface the slowest pages first
- Filter to `Content Type` = `text/html` to focus on pages (exclude images, CSS, JS which have their own performance considerations)

**What to look for:**
- Pages with response time exceeding 2 seconds -- these are the highest priority
- Patterns in slow pages: do all pages on a specific subdirectory, template type, or CMS section share high TTFB? This points to a backend performance issue with that template or data query rather than a server-wide problem
- Compare TTFB across page types: if category pages average 400ms but product pages average 1,800ms, the product page template likely has an expensive database query or external API call
- Check whether slow pages are also large pages -- a page with high TTFB and large size has compounding speed issues
- Look for pages with TTFB > 1 second that are high-priority for SEO (landing pages, category pages, top-traffic pages) -- these are the most impactful to fix

**Recommended fix:**

1. **Server-side caching.** Implement full-page caching (Varnish, Nginx FastCGI cache, CDN edge caching) for pages that do not require real-time personalization. A cached page should respond in under 200ms. This is the single highest-impact fix for TTFB.
2. **Database query optimization.** If specific page types are slow, profile the backend queries for that template. Common culprits: unindexed database columns, N+1 query patterns, full-table scans on large product/content tables, expensive JOINs that run on every page load.
3. **CDN implementation.** Serve pages from a CDN with edge caching enabled. This reduces TTFB by eliminating the geographic latency between the user (or crawler) and the origin server. Major CDNs (Cloudflare, Fastly, CloudFront) can cache HTML at the edge.
4. **Reduce server-side processing.** Eliminate synchronous external API calls during page rendering. If a page waits for a third-party API response before rendering, that API's latency is added directly to TTFB. Move external calls to asynchronous/client-side or cache the API responses.
5. **Upgrade hosting infrastructure.** If TTFB is consistently high across all page types and the above optimizations are already in place, the server hardware or hosting plan is likely undersized. Upgrade to faster CPU/RAM, or move from shared to dedicated/cloud hosting.

---

## 2. Page Size

**What it is:** The total size of the HTML document returned by the server, measured in bytes. In Screaming Frog, this is the `Size` column for HTML pages. This measures the transfer size of the HTML document itself, not the total page weight including all sub-resources (images, CSS, JS, fonts). However, an oversized HTML document is a strong indicator of an overall page weight problem, and Screaming Frog also reports the size of individual resources when crawled.

**Why it matters:** Page size directly affects download time, especially for users on mobile or slow connections. A 5MB HTML document takes approximately 10 seconds to download on a 4Mbps 3G connection -- and rendering cannot begin until the HTML is fully downloaded and parsed. Large pages also:
- **Consume more crawl budget.** Googlebot has a bandwidth budget per domain. Oversized pages use more of that budget per page, meaning fewer total pages are crawled per session.
- **Strain mobile devices.** Parsing and rendering large HTML documents consumes CPU and memory on mobile devices, leading to longer Time to Interactive and poor Interaction to Next Paint.
- **Increase hosting costs.** Bandwidth is not free at scale. Sites serving millions of pages with bloated HTML pay significantly more for CDN and origin bandwidth.

**Severity:**

| Condition | Severity | Rationale |
|-----------|----------|-----------|
| HTML size > 5MB | **High** | Excessive by any standard; likely contains inline resources, duplicated content, or serialized data that does not belong in HTML |
| HTML size 3-5MB | **Medium** | Above recommended thresholds; will cause load time issues on slower connections and increase crawl budget consumption |
| HTML size < 3MB | **Low/Acceptable** | Within normal range for most page types; further optimization is beneficial for complex pages but not urgent |

**How to identify in crawl data:**

- **Export tab:** `Internal:All`
- **Key columns:** `Address`, `Size` (bytes), `Content Type`, `Word Count`, `Response Time`
- Sort by `Size` descending to surface the largest pages first
- Filter to `Content Type` = `text/html` to isolate HTML documents

**What to look for:**
- Pages exceeding 5MB -- investigate immediately; HTML documents this large almost always contain something that should not be there
- Compare page size against word count: a page with 500 words but a 4MB HTML document has an enormous amount of non-content markup, inline data, or embedded resources
- Pattern analysis: if all pages of a specific type (e.g., category pages with product listings) are oversized, the template is the problem, not individual content
- Check for outliers: if most pages are 200-400KB but a handful are 3MB+, those specific pages likely have an issue (e.g., an embedded data table, inline SVGs, serialized JSON-LD at excessive scale)
- Cross-reference with response time: large pages with fast TTFB are a bandwidth problem; large pages with slow TTFB have compounding server + download issues

**Recommended fix:**

1. **Remove inline resources from HTML.** Common bloat sources: inline base64-encoded images, inline CSS that should be in external stylesheets, inline JavaScript that should be in external files, large SVGs embedded directly in the HTML. Move these to external files served with proper caching headers.
2. **Reduce excessive DOM size.** Pages with deeply nested or duplicated HTML elements (thousands of `<div>` wrappers, massive product listing tables rendered server-side) should be refactored. Consider paginating long listings or implementing "load more" patterns to reduce initial page size.
3. **Audit JSON-LD structured data size.** Some CMS configurations emit enormous JSON-LD blocks (e.g., serializing an entire product catalog on a category page). Structured data should describe the current page's content only, not the entire site or section.
4. **Enable HTML compression.** Ensure Gzip or Brotli compression is enabled on the server. This typically reduces HTML transfer size by 60-80%. If Screaming Frog reports the uncompressed size, check server headers to confirm compression is active.
5. **Lazy-load below-the-fold content.** For pages with legitimately large content (long-form articles, extensive product listings), implement lazy loading or progressive rendering so the initial HTML payload is smaller.

---

## 3. Large Images

**What it is:** Images served on the site that have excessively large file sizes. This includes images linked from HTML pages as `<img>` elements, CSS background images, and Open Graph/social images. In Screaming Frog, the `Images:All` export lists every image discovered during the crawl with its file size.

**Why it matters:** Images are typically the largest resources on a web page, often accounting for 50-80% of total page weight. A single unoptimized image can add seconds to page load time and is frequently the Largest Contentful Paint (LCP) element. Google's page speed ranking signal penalizes slow pages, and oversized images are the most common cause of slow pages. Beyond rankings:
- **LCP impact:** If the LCP element is an image (hero images, product images, featured images), its file size directly determines LCP timing. A 2MB hero image on a 4Mbps connection takes 4 seconds to download alone, failing Google's 2.5-second LCP threshold.
- **Bandwidth cost to users:** Mobile users on metered connections are downloading megabytes of image data they did not ask for. This directly impacts user experience and bounce rates.
- **Crawl efficiency:** Googlebot downloads images to process them for Google Images and to understand page content. Oversized images slow this process.

**Severity:**

| Condition | Severity | Rationale |
|-----------|----------|-----------|
| Image > 1MB | **High** | Far exceeds what any web image should be; likely uncompressed, wrong format, or serving print-resolution dimensions for a web viewport |
| Image 500KB-1MB | **Medium** | Above recommended web thresholds; will measurably slow page load, especially on mobile; may cause LCP failure if the image is the LCP element |
| Image < 500KB | **Low/Acceptable** | Acceptable for most use cases; hero images and full-width banners in this range are normal, but further compression is often possible |

**How to identify in crawl data:**

- **Export tab:** `Images:All`
- **Key columns:** `Address` (image URL), `Size` (bytes), `Status Code`, `Alt Text`
- Sort by `Size` descending to find the largest images
- Cross-reference with `Internal:All` to identify which pages serve the largest images -- the `Images:All` export shows the image URLs, while `All Inlinks` filtered by image type shows which pages reference them

**What to look for:**
- Any image over 1MB -- these should be investigated and optimized without exception
- Image format: are large images being served as PNG or BMP when they could be JPEG, WebP, or AVIF? PNGs are appropriate for graphics with transparency or sharp edges (logos, icons) but wrong for photographs and complex imagery
- Quantity of images per page: a page with 50 images at 300KB each has a 15MB image payload even though no single image is "large." Look at aggregate image weight per page, not just individual file sizes
- Pattern analysis: if all product images are 800KB+, the image pipeline (upload process, CMS resizing, CDN transformation) is not compressing or resizing correctly. This is a systemic fix, not a per-image fix.
- Check for images served at dimensions far larger than their display size -- a 4000x3000 pixel image displayed at 400x300 pixels is downloading 100x the necessary pixel data

**Recommended fix:**

1. **Convert to modern formats.** Serve images in WebP (supported by all modern browsers) or AVIF (better compression, growing support) with JPEG/PNG fallbacks. WebP typically achieves 25-35% smaller file sizes than equivalent-quality JPEG. Implement this via CDN image transformation (Cloudflare Polish, Imgix, Cloudinary) or build-time conversion.
2. **Compress aggressively.** For JPEG/WebP, quality settings of 75-85 are visually indistinguishable from 100 in most web contexts but can reduce file size by 40-60%. Use tools like ImageOptim, Sharp, or Squoosh for manual optimization; configure the CDN or CMS to apply compression automatically on upload.
3. **Serve responsive images.** Use `<img srcset>` and `<picture>` elements to serve appropriately sized images for each viewport. A mobile user on a 375px-wide screen should not download a 2000px-wide image. Implement width-based srcset at minimum: typically 400w, 800w, 1200w, and 1600w variants.
4. **Implement lazy loading.** Add `loading="lazy"` to images below the fold. This prevents the browser from downloading images that are not visible on initial page load, reducing initial page weight. Do not lazy-load the LCP image (hero image, first visible image) -- this should load eagerly.
5. **Set explicit width and height attributes.** While this does not reduce file size, it prevents Cumulative Layout Shift (CLS) caused by images loading and pushing content around. Include `width` and `height` attributes on all `<img>` elements.
6. **Fix the image pipeline.** If large images are a systemic issue across the site, the fix is not to optimize individual images but to fix the upload/processing pipeline. Configure the CMS or asset management system to automatically resize, compress, and convert images on upload. Set maximum dimension and file size limits.

---

## 4. Redirect Latency

**What it is:** The cumulative time added to page load by redirect hops between the initial URL request and the final destination. Each redirect (301, 302, 307, 308) requires a complete HTTP round trip: the browser requests URL A, receives a redirect response pointing to URL B, then must initiate a new request to URL B. If URL B also redirects, the cycle repeats. Each hop typically adds 50-300ms depending on server response time and geographic distance between client and server.

**Why it matters:** Redirect latency is additive and unavoidable -- the browser cannot skip redirect hops or predict the final destination (HSTS preload being one exception for HTTP-to-HTTPS redirects). A 3-hop redirect chain with 200ms per hop adds 600ms of pure redirect latency before the destination page even begins to load. This latency compounds on top of the destination page's own TTFB and rendering time.

From a crawl efficiency perspective, each redirect hop consumes a separate crawl request from Googlebot. A 3-hop redirect chain uses 3 crawl requests to reach one page. At scale, a site with thousands of internal links pointing to URLs that redirect through chains is burning crawl budget on redirect resolution instead of content discovery.

Google has stated it will follow up to 10 redirect hops but may stop sooner. More critically, link equity degrades through long redirect chains -- while a single 301 passes full equity, practical evidence shows measurable ranking loss at 3+ hops.

**Severity:**

| Condition | Severity | Rationale |
|-----------|----------|-----------|
| 3+ hop redirect chain | **High** | Significant latency addition (typically 300-900ms+); link equity degradation becomes measurable; crawl budget waste multiplied per hop; indicates unresolved redirect debt from migrations or URL changes |
| 2-hop redirect chain | **Medium** | Adds noticeable latency (typically 100-400ms); single intermediate redirect is common and tolerable (e.g., HTTP->HTTPS->canonical), but should still be consolidated where possible |
| 1-hop redirect (single redirect to final destination) | **Low/Acceptable** | Normal and expected for many legitimate scenarios (HTTP->HTTPS, www->non-www, old URL->new URL); no action needed unless the redirect itself is slow |

**How to identify in crawl data:**

- **Export tab:** `Response Codes:Redirection (3xx)` -- shows all URLs returning redirect status codes
- **Bulk export:** `All Redirect Chains` -- shows the full chain for each redirecting URL, including every hop and the final destination
- **Key columns in redirect chain export:** `Source URL`, `Number of Redirects`, `Redirect URL 1`, `Redirect URL 2`, etc., `Final Destination URL`, `Final Status Code`
- **Cross-reference with `Internal:All`:** Check the `Response Time` column for redirecting URLs to measure the per-hop latency

**What to look for:**
- Any chain with 3 or more hops -- these are the highest priority and should be shortened to a single redirect from source directly to final destination
- Chains where the final destination returns a non-200 status code (especially 404) -- the redirect chain leads to a dead end, combining redirect latency waste with a broken destination
- Redirect chains involving internal links: if the site's own pages link to URLs that trigger multi-hop redirects, the internal linking should be updated to point directly to the final destination URL
- Chains caused by protocol + domain + path changes stacking up: e.g., `http://www.example.com/old-page` -> `https://www.example.com/old-page` -> `https://example.com/old-page` -> `https://example.com/new-page` is 3 hops that could be 1
- Systemic patterns: if an entire URL pattern (e.g., all `/blog/2019/*` URLs) redirects through chains, a single server-side rule can fix hundreds of chains at once

**Recommended fix:**

1. **Consolidate chains to single hops.** Update redirect rules so that each source URL redirects directly to the final destination in one hop. If `A -> B -> C -> D`, replace with `A -> D`, `B -> D`, `C -> D`. This is the most impactful fix.
2. **Update internal links to point to final destination URLs.** Every internal link should point to the canonical, final URL -- not to a URL that redirects. Export `All Inlinks`, filter for destinations that appear in the redirect chain list, and update those links in the CMS, templates, or navigation to point to the final URL. This eliminates the redirect entirely for internal navigation.
3. **Implement redirect rules at the edge.** Configure redirects at the CDN or load balancer level rather than the application server. Edge redirects respond in 10-50ms vs. 100-300ms for application-level redirects, reducing per-hop latency even when chains cannot be fully eliminated.
4. **Audit after migrations.** Redirect chains most commonly accumulate after site migrations, domain changes, or URL restructuring. After any migration, export all redirect chains and verify that no new multi-hop chains were created. Build this check into the migration QA process.
5. **Use 301 (permanent) over 302 (temporary) for permanent moves.** Ensure permanent URL changes use 301 status codes. 302 redirects signal to search engines that the move is temporary, which can delay link equity transfer and cause the old URL to persist in the index longer than necessary.
