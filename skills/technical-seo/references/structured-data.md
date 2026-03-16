# Structured Data Reference

Use this reference when analyzing crawl data related to schema markup (structured data) — missing schema, invalid markup, incomplete properties, or mismatched schema types. Each issue includes what it is, why it matters, severity classification, how to identify it in Screaming Frog crawl data, and the recommended fix.

**Important prerequisite:** Structured data analysis requires Screaming Frog's "Structured Data" extraction to be enabled in the crawl configuration. This is found under **Configuration > Spider > Extraction > Structured Data**. If structured data columns are missing or empty in the crawl export, this extraction was not enabled during the crawl. Note this gap in your analysis and recommend the client enable structured data extraction for future crawls. Without this data, structured data issues cannot be assessed from the crawl export alone — the client would need to use Google's Rich Results Test or Schema Markup Validator on individual URLs instead.

**Format recommendation:** JSON-LD is the recommended format for implementing structured data. Google explicitly recommends JSON-LD over Microdata and RDFa. JSON-LD is easier to implement, easier to maintain (it lives in a `<script>` block separate from the HTML markup), and easier to debug. All recommendations in this reference assume JSON-LD implementation.

**Validation tool:** For validating structured data on individual pages, use [Google's Rich Results Test](https://search.google.com/test/rich-results). It shows exactly which rich result types are detected, which required and recommended properties are present or missing, and whether the markup has syntax errors. For broader schema validation beyond Google's supported types, use the [Schema Markup Validator](https://validator.schema.org/).

---

## 1. Missing Schema on Key Page Types

**What it is:** Pages that should have structured data markup based on their content type but have no schema implemented at all. For example, a product page with no `Product` schema, or a blog post with no `Article` schema. The page content clearly matches a schema.org type, but no structured data is present in the page source.

**Why it matters:** Structured data is the mechanism by which search engines understand page content in a machine-readable format. Without it:
- The page is **ineligible for rich results** (rich snippets, carousels, knowledge panels, FAQ dropdowns, recipe cards, event listings, etc.) — these enhanced search features drive significantly higher click-through rates than standard blue links
- Search engines must **infer** content type and properties from unstructured HTML, which is less reliable and less precise than explicit schema markup
- Competitors with structured data gain a **visual advantage** in search results — their listings take up more space, display more information, and attract more clicks
- Pages miss out on **entity understanding** — schema helps search engines connect the page to knowledge graph entities, improving relevance for related queries

The absence of schema is not a ranking penalty, but it is a missed opportunity that becomes a competitive disadvantage when competitors implement it.

**Severity:** **Medium** — missing schema does not cause ranking damage, but it forfeits rich result eligibility and reduces search result visibility compared to competitors. Prioritize schema implementation on page types where rich results are most impactful (products, FAQs, articles, events).

**Recommended schema by page type:**

| Page Type | Recommended Schema Type(s) | Rich Result Eligibility |
|-----------|---------------------------|------------------------|
| Product pages | `Product`, `Offer`, `AggregateRating` | Product rich results (price, availability, ratings in search results) |
| Blog posts / Articles | `Article` or `BlogPosting` | Article rich results (headline, thumbnail, date in Top Stories and Discover) |
| FAQ pages | `FAQPage` with `Question` + `Answer` | FAQ rich results (expandable Q&A directly in search results) |
| How-to / Tutorial pages | `HowTo` | How-to rich results (step-by-step instructions in search results) |
| Event pages | `Event` | Event rich results (date, location, ticket info in search results and Events experience) |
| Local business pages | `LocalBusiness` (or more specific subtype) | Local business knowledge panel (address, hours, phone, reviews) |
| Recipe pages | `Recipe` | Recipe rich results (image, rating, cook time, calories in search results) |
| Review pages | `Review` | Review snippet (star rating in search results) |
| All pages with breadcrumbs | `BreadcrumbList` | Breadcrumb trail displayed in search results instead of raw URL |
| Homepage | `Organization` (or `WebSite` with `SearchAction`) | Organization knowledge panel; sitelinks search box |

**How to identify in crawl data:**

- **Export tab:** `Internal:All` — if structured data extraction was enabled, look for columns related to structured data types (e.g., `Structured Data`, `Schema Type`, or similar depending on Screaming Frog version)
- **Key columns:** `Address`, `Content Type`, `Structured Data` (or related columns), `Indexability`
- Filter for indexable HTML pages where structured data columns are empty or blank
- **Cross-reference with URL patterns:** Identify page types by URL pattern (e.g., `/products/`, `/blog/`, `/events/`) and check whether those page groups have schema

**What to look for:**
- Group pages by URL pattern or template type and check schema coverage per group. A template with zero schema across hundreds of pages indicates a template-level implementation gap.
- Prioritize by page type impact: product pages and FAQ pages have the highest rich result impact; breadcrumbs have the broadest applicability
- Check competitor search results: if competitors are showing rich results for target queries and the site is not, missing schema is the likely cause

**Recommended fix:**

1. **Implement schema at the template level** — do not add schema to individual pages manually. Every CMS or site framework allows injecting JSON-LD into templates, ensuring every page of a given type automatically gets the correct schema.
2. **Start with the highest-impact types:**
   - `BreadcrumbList` — add to all pages; easiest to implement and universally applicable
   - `Product` + `Offer` — add to all product pages; directly drives purchase-intent rich results
   - `Article` / `BlogPosting` — add to all editorial content
   - `FAQPage` — add to any page with Q&A content
   - `Organization` — add to the homepage
3. **Use dynamic data** from the CMS to populate schema properties. Do not hardcode values — pull product prices, article dates, ratings, etc. directly from the page's data source to ensure accuracy.
4. **Validate after implementation** using Google's Rich Results Test on representative pages from each template type. Then monitor the Google Search Console Enhancements reports for schema errors and rich result impressions.

---

## 2. Invalid Schema

**What it is:** The page contains structured data markup that has syntax errors, uses invalid schema types, or references properties that do not exist in the schema.org vocabulary. Common forms of invalid schema include:
- **JSON-LD syntax errors:** Missing commas, unclosed brackets, unquoted property names, trailing commas — any malformed JSON that prevents parsing
- **Invalid `@type` values:** Using a type that does not exist in schema.org (e.g., `@type: "ProductPage"` instead of `@type: "Product"`, or `@type: "BlogArticle"` instead of `@type: "BlogPosting"`)
- **Invalid property names:** Using properties that do not exist for the specified type (e.g., `"cost"` instead of `"price"` on a `Product`, or `"writer"` instead of `"author"` on an `Article`)
- **Invalid property values:** Using the wrong value type for a property (e.g., a string where an object is expected, or a number where a date string is expected)
- **Microdata/RDFa syntax errors:** Incorrect `itemscope`, `itemtype`, or `itemprop` attributes (less common with JSON-LD adoption)

**Why it matters:** Invalid schema is completely ignored by search engines. If the JSON-LD cannot be parsed, or if the types and properties are not recognized, the markup provides zero value — it is as if the schema does not exist at all. The page is ineligible for any rich results, and the development effort that went into implementing the schema is wasted. Worse, invalid schema can persist undetected for months because there is no visible impact on the rendered page — it fails silently.

**Severity:** **High** — the site has invested effort in implementing structured data, but that effort produces no benefit due to errors. This is a higher priority than missing schema because the intent is already there and the fix is typically correcting syntax or type names rather than building from scratch.

**How to identify in crawl data:**

- **Export tab:** If Screaming Frog's structured data extraction is enabled, look for a `Structured Data Validation` column or error indicators in the structured data output
- **Key columns:** `Address`, `Structured Data` (type/content), any validation error columns
- Screaming Frog's structured data extraction will flag some validation errors (invalid types, missing required properties) depending on the version and configuration
- **Cross-reference with:** Google Search Console > Enhancements reports — these show structured data errors detected during Google's crawl, grouped by schema type

**What to look for:**
- Pages where structured data is present but marked as having errors — these are the highest priority because the fix is typically small
- Pattern analysis: if all product pages have the same schema error, it is a single template fix. Count affected pages per error type to prioritize.
- Common error patterns:
  - A CMS plugin or theme generating schema with a known bug
  - Dynamic values (prices, dates, ratings) that sometimes render as `null`, empty strings, or in the wrong format
  - Schema that was valid when implemented but has become invalid due to schema.org vocabulary updates

**Recommended fix:**

1. **Run affected URLs through Google's Rich Results Test** to get specific, detailed error messages. The test shows exactly which line of the JSON-LD has an error and what is wrong.
2. **For JSON-LD syntax errors:** Fix the JSON itself. Common fixes:
   - Add missing commas between properties
   - Close unclosed brackets or braces
   - Remove trailing commas after the last property in an object
   - Ensure all property names are double-quoted (JSON requires double quotes, not single quotes)
   - Escape special characters in string values
3. **For invalid types:** Replace with the correct schema.org type name. Refer to [schema.org/docs/full.html](https://schema.org/docs/full.html) for the complete type hierarchy.
4. **For invalid properties:** Replace with the correct property name for the given type. Check the schema.org documentation for the specific type to see its expected properties.
5. **For invalid property values:** Correct the value format. Common fixes:
   - Dates must be in ISO 8601 format (`2024-01-15` or `2024-01-15T10:30:00+00:00`)
   - Prices must be numeric strings without currency symbols (`"29.99"`, not `"$29.99"`)
   - Properties expecting objects (like `author`) need a nested `@type` and required sub-properties, not just a plain string
6. **Fix at the template level** — once the error is identified, fix the template that generates the schema, not individual pages. One template fix corrects the error across all pages using that template.
7. **Set up ongoing validation:** Add structured data validation to the deployment process (use Google's Rich Results API or structured-data-testing-tool in CI/CD) to catch regressions before they go live.

---

## 3. Incomplete Schema

**What it is:** The page has valid structured data with a correct `@type` and parseable JSON-LD, but it is missing properties — either required properties that are mandatory for rich result eligibility, or recommended properties that enhance the richness and quality of the result. Schema.org and Google's documentation distinguish between:
- **Required properties:** Must be present for the schema to be eligible for rich results. Missing a required property means Google will not generate a rich result from the markup at all, even though it can parse it.
- **Recommended properties:** Not mandatory, but their presence improves the quality and completeness of the rich result. Missing recommended properties means the rich result may appear but will be less informative or visually less compelling than competitors.

**Why it matters:**
- Missing **required** properties makes the schema functionally useless for rich results — the markup is valid and parseable, but Google will not use it because it lacks essential information. This is similar to invalid schema in practical impact.
- Missing **recommended** properties reduces the quality and attractiveness of the rich result. For example, a `Product` schema without `aggregateRating` will not show star ratings in search results, while a competitor's product listing with ratings will appear more trustworthy and clickable.

**Severity:**

| Condition | Severity | Rationale |
|-----------|----------|-----------|
| Missing **required** properties | **High** | Schema is present but non-functional for rich results; effectively the same as having no schema |
| Missing **recommended** properties | **Medium** | Rich results will appear but may be less complete or competitive than they could be |

**Required properties by common schema type (per Google's documentation):**

| Schema Type | Required Properties |
|-------------|-------------------|
| `Product` | `name`; also requires either `review`, `aggregateRating`, or `offers` for rich result eligibility |
| `Offer` (nested in Product) | `price` (or `priceSpecification`), `priceCurrency`, `availability` |
| `Article` / `BlogPosting` | `headline`, `image`, `datePublished`, `author` (with `name`) |
| `FAQPage` | `mainEntity` (array of `Question` items, each with `name` and `acceptedAnswer` containing `text`) |
| `Event` | `name`, `startDate`, `location` (with `name` and `address`) |
| `LocalBusiness` | `name`, `address` (with `streetAddress`, `addressLocality`, `addressRegion`, `postalCode`), `@id` |
| `BreadcrumbList` | `itemListElement` (array of `ListItem` with `position`, `name`, and `item`) |
| `Recipe` | `name`, `image` |
| `HowTo` | `name`, `step` (array of `HowToStep` with `text`) |
| `Organization` | `name`, `url`, `logo` |

**How to identify in crawl data:**

- **Export tab:** If structured data extraction is enabled, Screaming Frog will surface structured data properties per URL. Look for schema entries that have the correct type but are missing key properties.
- **Key columns:** `Address`, `Structured Data` columns (type, properties present)
- **Primary method:** Google Search Console > Enhancements reports. Google groups structured data issues by type and clearly lists "missing field" warnings for each required and recommended property.
- **Per-URL validation:** Run representative URLs from each template through Google's Rich Results Test — it explicitly lists which required and recommended properties are present and which are missing.

**What to look for:**
- Group by schema type and check which properties are consistently missing across all pages of that type — these are template-level gaps that can be fixed with a single template change
- Distinguish between properties that are **missing because the data does not exist** (e.g., no reviews on a new product) versus **missing because the template does not include them** (e.g., the template never outputs `aggregateRating` even when ratings data exists in the CMS). The latter is the fixable issue.
- Check Google Search Console Enhancements for a count of affected pages per missing property — this helps prioritize which properties to add first

**Recommended fix:**

1. **Address missing required properties first** — these are blocking rich result eligibility entirely.
2. **Update the template** to include all required properties for the schema type. Pull values dynamically from the CMS data source:
   - `Product`: Ensure the template outputs `name`, `offers` (with `price`, `priceCurrency`, `availability`), and `image`. Add `aggregateRating` if ratings data is available.
   - `Article`: Ensure the template outputs `headline`, `image`, `datePublished`, and `author` (as an object with at least `name` and `@type: "Person"` or `"Organization"`).
   - `FAQPage`: Ensure each Q&A pair is output as a `Question` with `name` (the question text) and `acceptedAnswer` (with `@type: "Answer"` and `text` containing the answer).
   - `Event`: Ensure `name`, `startDate` (ISO 8601), and `location` (with full address breakdown) are all present.
   - `LocalBusiness`: Ensure `name`, full `address` breakdown, and a unique `@id` are present.
   - `BreadcrumbList`: Ensure each breadcrumb item is a `ListItem` with `position` (integer), `name`, and `item` (URL).
3. **Handle missing data gracefully:** If a required property's data is not available for a specific page (e.g., a product with no price yet), either omit the schema entirely for that page or output a valid fallback. Do not output empty strings or null values for required properties — this creates invalid schema.
4. **Then address recommended properties** to maximize rich result quality. Common high-value recommended properties:
   - `Product`: `image`, `description`, `brand`, `sku`, `aggregateRating`, `review`
   - `Article`: `dateModified`, `publisher` (with `name` and `logo`), `description`
   - `Event`: `endDate`, `description`, `image`, `offers` (ticket info), `performer`, `organizer`
   - `LocalBusiness`: `telephone`, `openingHoursSpecification`, `geo` (latitude/longitude), `image`, `priceRange`
5. **Validate after implementation** using Google's Rich Results Test on representative pages. Confirm that rich results appear in the test preview and no required property warnings remain.

---

## 4. Mismatched Schema

**What it is:** The structured data type on a page does not match the actual content of the page. The schema claims the page is about one thing, but the page content is about something else. Examples:
- `Article` or `BlogPosting` schema on a product page
- `Product` schema on an informational/about page
- `FAQPage` schema on a page that does not contain any Q&A content
- `LocalBusiness` schema on every page of the site instead of only the location/contact page
- `Recipe` schema on a page that is not a recipe
- `Event` schema on a page describing a past event that is no longer relevant
- `NewsArticle` schema on evergreen content that is not news
- Generic `WebPage` schema used everywhere instead of specific, appropriate types

**Why it matters:** Mismatched schema is a form of structured data abuse, even if unintentional. The consequences are serious:
- **Google may issue a manual action** (penalty) for structured data that does not match page content. The Structured Data guidelines explicitly state that schema must represent content "on the page where it appears."
- **Rich results will be revoked** — Google actively monitors for mismatches and will remove rich results for pages (or entire sites) where schema does not reflect reality
- **Loss of trust signals** — if Google detects a pattern of mismatched schema across a site, it may reduce trust in ALL structured data on the domain, affecting even correctly-marked pages
- **Misleading users** — if a search result shows FAQ rich results but the page has no FAQ content, users will bounce, increasing pogo-sticking signals

This is not just a missed opportunity like missing schema — mismatched schema actively harms the site's standing with search engines.

**Severity:** **High** — mismatched schema risks manual actions, rich result revocation, and domain-wide trust degradation. This should be fixed with higher priority than missing or incomplete schema.

**How to identify in crawl data:**

- **Export tab:** If structured data extraction is enabled, export structured data types per URL and cross-reference with the page's content type (determined by URL pattern, template type, or manual review)
- **Key columns:** `Address`, `Structured Data` (type), `Title 1`, `H1-1`, `Word Count`
- **Method:** Group pages by schema type and spot-check whether the schema type matches the page content:
  - All URLs with `Product` schema: do they look like product pages based on URL pattern and title?
  - All URLs with `Article` schema: are they editorial content or something else?
  - All URLs with `FAQPage` schema: do the pages actually contain Q&A content?

**What to look for:**
- **Schema type does not match URL pattern:** e.g., URLs like `/about-us/` or `/contact/` having `Product` schema
- **Same schema type on all pages:** If every page on the site has `Article` schema, but many pages are products, categories, or landing pages, the schema is applied incorrectly at the template level
- **Copy-paste schema:** A JSON-LD block that is identical across every page (same `name`, same `description`, same properties) — this usually means schema was hardcoded on one page and then applied site-wide via a global template without being made dynamic
- **Schema for content that is not visible on the page:** The schema describes FAQ questions, product details, or reviews that do not actually appear in the rendered page content. Google requires that schema reflect content visible to users.
- **Outdated schema:** `Event` schema for events that have already occurred (with `startDate` in the past), or `Product` schema with prices or availability that do not match the current page content

**Recommended fix:**

1. **Audit schema-to-content alignment for every template type.** For each page template (product, article, category, landing page, FAQ, etc.), verify that the schema type is correct for that template's content.
2. **Remove mismatched schema immediately.** Having no schema is better than having wrong schema. Remove the incorrect markup first, then implement the correct type.
3. **Fix at the template level:**
   - Ensure each template outputs schema only for the type that matches its content
   - Product templates output `Product` schema; article templates output `Article` schema; FAQ templates output `FAQPage` schema
   - Do not apply a single schema type globally across all templates
4. **Make schema dynamic, not hardcoded.** Every property value in the schema must be pulled from the page's actual data. The `name` in the schema must match the page's title/heading. The `price` must match the displayed price. The `description` must reflect the page content.
5. **Remove schema for content not visible on the page.** If the schema references FAQ questions that are not displayed, product reviews that are not shown, or event details that are not on the page, either add that content to the page or remove it from the schema.
6. **Handle edge cases:**
   - **Past events:** Either remove `Event` schema from past events, or update the `eventStatus` property to `EventCancelled` or `EventPostponed` as appropriate. Do not leave `Event` schema with a `startDate` in the past on a page that no longer serves event information.
   - **Out-of-stock products:** Keep `Product` schema but update `availability` to `OutOfStock`. Do not remove the schema entirely — the product page still exists and is still a product.
   - **Multi-type pages:** Some pages legitimately contain multiple content types (e.g., a product page with embedded reviews and FAQ). In these cases, multiple schema types are appropriate — use `Product` + `FAQPage` + `Review` — but each schema type must correspond to content that is actually present and visible on the page.
7. **Validate with Google's Rich Results Test** after fixing. Confirm that the detected schema type matches the page content and that no warnings about mismatched content appear.

---

## Quick Reference: Severity Summary

| Issue | Default Severity | Escalates To | Condition for Escalation |
|-------|-----------------|-------------|-------------------------|
| Missing Schema on Key Page Types | **Medium** | — | — |
| Invalid Schema (syntax/type errors) | **High** | — | Always high; effort was invested but produces zero value |
| Incomplete Schema — Missing Required Properties | **High** | — | Schema is non-functional for rich results |
| Incomplete Schema — Missing Recommended Properties | **Medium** | — | Reduces rich result quality but does not block eligibility |
| Mismatched Schema (type does not match content) | **High** | **Critical** | Pattern of mismatches across site risks manual action |
