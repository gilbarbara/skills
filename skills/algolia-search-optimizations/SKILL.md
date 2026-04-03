---
name: algolia-search-optimizations
description: "Audit and optimize Algolia DocSearch setups: diagnose content indexing gaps, fix crawler selectors, add synonyms, create custom records for non-crawlable pages, and verify improvements via analytics. Use when the user mentions DocSearch not returning good results, search gaps, no-result queries, crawler config debugging, search relevance tuning, Algolia analytics, or wants to improve their documentation site's search quality. Also trigger when they mention search audit, missing search results, or want to check if their Algolia index is working correctly."
---

# Algolia Search Optimizations

Systematic workflow for auditing and improving an Algolia DocSearch setup. This covers the full cycle: diagnose gaps via analytics, validate index content, fix crawler config, add synonyms, create custom records, and verify improvements.

This skill is for sites already using DocSearch. For implementing Algolia search from scratch (InstantSearch, indexing API), use the `algolia-search` skill instead.

## Prerequisites

You'll need from the user:
- **App ID** — found in the DocSearch component or Algolia dashboard
- **Search API key** — the public key (read-only, safe to share)
- **Analytics/Admin API key** — required for analytics queries (ask the user, don't guess)
- **Index name** — usually visible in the DocSearch component config

The search API key is often visible in the site's source code. The analytics key must be provided by the user.

## Phase 1: Query Analytics for Gaps

Start here. The analytics API reveals what users search for and where the index fails them. See `references/analytics-api.md` for all endpoints and curl examples.

Run these queries in parallel:

1. **Top searches** (`clickAnalytics=true`) — what users search most, hit counts, CTR
2. **No-result searches** — queries returning 0 hits (highest priority gaps)
3. **No-click searches** — results exist but nobody clicks (relevance problem)
4. **Search volume** — daily search count to understand traffic patterns
5. **Click positions** — where users click (position 1 = ranking is good)
6. **Click-through rate** — overall CTR (requires `insights` enabled in DocSearch component)

Present findings as a table:

```
Query              Searches  Hits  Clicks  CTR     Assessment
disableBeacon      9         0     -       -       old prop name, not indexed
dark               4         0     -       -       No dark mode docs exist
callback           6         8     0       0%      Results exist but wrong page
```

Categorize gaps:
- **Missing content** — term exists in docs but not indexed (crawler issue)
- **Missing synonym** — user searches term A, docs use term B
- **Missing docs** — no content exists for this topic
- **Wrong result** — content indexed but irrelevant page ranks first

## Phase 2: Validate Index Content

Query the index directly to understand what's actually indexed.

### Record type distribution

```bash
curl -s "https://{APP_ID}-dsn.algolia.net/1/indexes/{INDEX}/query" \
  -H "X-Algolia-Application-Id: {APP_ID}" \
  -H "X-Algolia-API-Key: {SEARCH_KEY}" \
  -d '{"query":"","hitsPerPage":0,"facets":["type"]}'
```

DocSearch creates records of types: `lvl1`, `lvl2`, `lvl3` (headings) and `content` (body text). If there are zero `content` records, the crawler isn't extracting body text — this is the most common and impactful issue.

### Content quality check

Sample content records and verify they contain actual page text:

```bash
# Filter to content-type records only
-d '{"query":"","hitsPerPage":5,"facetFilters":["type:content"],"attributesToRetrieve":["content","hierarchy","url","type"]}'
```

Red flags:
- `content: null` — selectors don't match any elements
- Content is navbar/header text (e.g., "Documentation\r\nDemos\r\nSearch") — selectors are too broad
- Content is identical across all records — wrong element matched

### Garbled anchors

Scan for auto-generated IDs (from React Aria, Radix, Headless UI, etc.):

```bash
# Fetch all anchors and check for patterns like _R_, :r, data-
-d '{"query":"","hitsPerPage":200,"attributesToRetrieve":["anchor","url"]}'
```

Filter for anchors matching `^_R_`, `^:r`, or other framework-generated patterns. These create records that link to wrong positions on the page.

## Phase 3: Diagnose Crawler Issues

When content isn't indexed correctly, the problem is almost always a mismatch between CSS selectors in the crawler config and the actual DOM structure.

### Fetch what the crawler sees

```bash
curl -s "https://your-site.com/docs/some-page" | python3 -c "
import sys, re
html = sys.stdin.read()

# Check for semantic elements
for tag in ['article', 'main', 'section']:
    count = html.lower().count(f'<{tag}')
    print(f'<{tag}> count: {count}')

# Count content elements inside main
main = re.search(r'<main[^>]*>(.*?)</main>', html, re.DOTALL)
if main:
    content = main.group(1)
    for tag in ['p', 'li', 'td', 'h1', 'h2', 'h3']:
        count = len(re.findall(f'<{tag}[\\\\s>]', content))
        print(f'  <{tag}> inside main: {count}')
"
```

If `renderJavaScript: false` in the crawler config, this is exactly what the crawler gets. If the page relies on client-side rendering, the crawler may see an empty shell.

### Common selector issues

**No `<article>` element**: Selectors like `article p` match nothing. Use the actual content wrapper (e.g., `#docs-content p`, `main p`, `.prose p`).

**Nav/header contamination**: Broad selectors like `p, li` match navbar elements that appear before `<main>`. The DocSearch helper tries selectors in order and uses the first match — but `<li>` tags in the header may rank before content `<p>` tags in DOM order. Fix by scoping to the content container.

**Content in tables**: Many docs sites use tables for API reference. If `<td>` isn't in the content selector, prop names and descriptions won't be indexed.

**Auto-generated IDs**: Interactive components (accordions, tabs, playgrounds) generate IDs like `_R_4dlt8anpfl5ulb_`. Use Cheerio to remove these elements before extraction.

## Phase 4: Recommend Fixes

### Crawler config fixes

The `recordExtractor` receives a Cheerio instance (`$`) for DOM manipulation before extraction:

```js
recordExtractor: ({ $, helpers }) => {
  // Remove elements that generate garbled anchors
  $("#playground").remove();
  $("[id^='_R_']").remove();

  return helpers.docsearch({
    recordProps: {
      lvl1: ["#content-wrapper h1", "main h1", "head > title"],
      content: ["#content-wrapper p", "#content-wrapper li", "#content-wrapper td"],
      lvl0: { selectors: "", defaultValue: "Documentation" },
      lvl2: ["#content-wrapper h2"],
      lvl3: ["#content-wrapper h3"],
      // ...
    },
    aggregateContent: true,
    recordVersion: "v3",
  });
}
```

Replace `#content-wrapper` with the actual content container ID/class. Adding `td` to content selectors is important for API reference pages.

**Do not remove `<pre>` or `<code>` elements** — code blocks often contain prop names, API examples, and configuration patterns that users search for. Only remove elements that generate noise (garbled anchors, interactive widgets, navigation).

### Synonyms

Add in the Algolia dashboard under Relevance Optimizations > Synonyms. Each term is entered as a separate tag (type, press Enter, repeat).

**Bidirectional synonyms**: Both terms find each other's results.
- `i18n, internationalization, locale, translation, language`
- `a11y, accessibility`

**One-way synonyms**: Searching A also finds B, but not vice versa.
- `callback` -> `onEvent` (v2 to v3 migration terms)
- `disableBeacon` -> `skipBeacon`

### Custom records for non-crawlable pages

For interactive pages (demos, playgrounds) where the DOM isn't useful, add a second crawler action that returns records directly:

```js
{
  indexName: "Documentation",
  pathsToMatch: [
    "https://your-site.com/demos/overview",
    "https://your-site.com/demos/modal",
    // explicit URLs, no wildcards to avoid sub-routes
  ],
  recordExtractor: ({ url }) => {
    const descriptions = {
      '/demos/overview': 'Feature showcase with hooks, spotlight config, and skip behavior',
      '/demos/modal': 'Tours inside modals with portalElement and z-index handling',
    };

    const path = new URL(url).pathname.replace(/\/$/, '');
    const content = descriptions[path];
    if (!content) return [];

    const title = path.split('/').pop()
      .replace(/-/g, ' ')
      .replace(/\b\w/g, c => c.toUpperCase());

    return [{
      type: 'content',
      lang: 'en',
      content,
      hierarchy: {
        lvl0: 'Examples',
        lvl1: `${title} Demo`,
        lvl2: null, lvl3: null, lvl4: null, lvl5: null, lvl6: null,
      },
      url: url.href,
      url_without_anchor: url.href,
      anchor: null,
      weight: { pageRank: 0, level: 100, position: 0 },
      recordVersion: 'v3',
    }];
  }
}
```

Use curated keyword-rich descriptions. The `lvl0` value controls grouping in search results — "Documentation" and "Examples" sort alphabetically, so "Examples" appears after "Documentation".

### DocSearch component

Enable click tracking if not already done:

```tsx
<DocSearch
  appId="..."
  apiKey="..."
  indexName="..."
  insights={true}
/>
```

This enables Algolia Insights, which populates CTR and click position data in analytics. Without it, those fields are always `null`.

## Phase 5: Verify After Re-crawl

After the user applies changes and triggers a re-crawl, run verification:

1. **Record count and types** — confirm content records increased
2. **Content quality** — sample content records, verify real text (not nav garbage)
3. **Garbled anchors** — scan for auto-generated IDs, should be 0
4. **Previously failing queries** — test every query that had 0 hits before
5. **Result quality** — for key queries, check the top 3 results make sense
6. **Click tracking** — if `insights` was just enabled, confirm `trackedSearchCount > 0` after a day or two

Run all test queries in a batch:

```bash
for q in "term1" "term2" "term3"; do
  count=$(curl -s "https://{APP_ID}-dsn.algolia.net/1/indexes/{INDEX}/query" \
    -H "X-Algolia-Application-Id: {APP_ID}" \
    -H "X-Algolia-API-Key: {SEARCH_KEY}" \
    -d "{\"query\":\"$q\",\"hitsPerPage\":0}" | python3 -c "import sys,json; print(json.load(sys.stdin)['nbHits'])")
  echo "$q -> $count hits"
done
```
