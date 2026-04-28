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
- **Search API key** — public, read-only. Powers Phase 0 (settings/synonyms/rules), Phase 2 (record inspection), Phase 4 snapshot loop.
- **Analytics API key** — admin or analytics ACL. Powers Phase 1 (analytics endpoints).
- **Index name** — usually visible in the DocSearch component config

The search API key is often visible in the site's source code. The analytics key must be provided by the user. If a Phase 0 or Phase 2 curl returns `403 "Method not allowed with this API key"`, the user gave you the analytics key by mistake — ask for the search key.

## Phase 0: Inventory Deployed Configuration

Before analyzing gaps or proposing changes, fetch what is already live in Algolia. Skipping this leads to redundant proposals and fixes that conflict with existing rules. Everything in this phase is read-only and uses the keys from Prerequisites.

The deployed Algolia state is the source of truth — a repo may also have committed crawler-config or notes files (`crawler*.json`, `algolia.md`, etc.). Treat those as a secondary cross-check, not the primary source.

### Deployed synonyms

```bash
curl -s "https://{APP_ID}-dsn.algolia.net/1/indexes/{INDEX}/synonyms/search" \
  -H "X-Algolia-Application-Id: {APP_ID}" \
  -H "X-Algolia-API-Key: {SEARCH_KEY}" \
  -d '{"query":"","hitsPerPage":1000}'
```

Returns one-way and bidirectional synonyms. Skip proposing any synonym already present.

### Index settings

```bash
curl -s "https://{APP_ID}-dsn.algolia.net/1/indexes/{INDEX}/settings" \
  -H "X-Algolia-Application-Id: {APP_ID}" \
  -H "X-Algolia-API-Key: {SEARCH_KEY}"
```

Pay attention to: `customRanking`, `ranking`, `searchableAttributes`, `attributeForDistinct`, `distinct`, `removeWordsIfNoResults`, `minWordSizefor1Typo`, `minWordSizefor2Typos`, `ignorePlurals`. These together determine why any given record ranks where it does — required reading before proposing any ranking, preview, or relevance fix.

### Query rules

```bash
curl -s "https://{APP_ID}-dsn.algolia.net/1/indexes/{INDEX}/rules/search" \
  -H "X-Algolia-Application-Id: {APP_ID}" \
  -H "X-Algolia-API-Key: {SEARCH_KEY}" \
  -d '{"query":"","hitsPerPage":1000}'
```

Query rules can boost, filter, or redirect specific queries. Often deployed and otherwise invisible.

### Click-tracking check

CTR and click position fields in analytics depend on Algolia Insights. If those fields are `null` everywhere despite traffic, the DocSearch component on the site is missing `insights={true}`:

```tsx
<DocSearch appId="..." apiKey="..." indexName="..." insights={true} />
```

Flag this to the user as a prerequisite before any no-click analysis is meaningful.

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

### Action threshold

Don't propose changes for queries below the noise floor. Starting heuristics — **tune per index volume, these are not hard rules**:

- **No-result queries**: act only if ≥3 occurrences in the analytics window, or if the term is a known prop/API name and ≥2 occurrences.
- **No-click queries**: act only if ≥10 hits with 0 clicks, or ≥5 searches with 0 clicks.
- Single-occurrence misses, typos, and gibberish: log only, don't propose.

First, look at total search volume. Indexes under ~100 searches / 30 days need lower thresholds (e.g., halve the numbers) since absolute counts are noisier. High-volume indexes (10k+/month) can raise thresholds to filter louder background noise. State the threshold you used when reporting findings so the user can challenge it.

### Categorize gaps

Use this lens — it determines which phase has the fix:

- **0 hits returned** → either a content gap (term doesn't exist anywhere in the docs) or a synonym gap (docs use a different word). Verify by searching local source files for the term across any extension (`.md`, `.mdx`, `.rst`, `.html`, `.txt`, `.adoc`, etc.). If found, it's a synonym/indexing gap; if not, it's a content gap or a non-real term.
- **>0 hits, 0 clicks** → the term is indexed but the result that ranks first is the wrong page, or the preview snippet is empty/unhelpful. **Almost never a content gap.** Inspect the top hit's `type`, `content`, and ranking position. Cross-reference with the live index settings from Phase 0 (`customRanking`, `searchableAttributes`, `attributeForDistinct`, `distinct`) — these together determine why that record ranks first.

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

If a record's content is unexpectedly empty or stale, search local source files for the heading text or anchor across any extension the docs use (`.md`, `.mdx`, `.rst`, `.html`, `.txt`, `.adoc`, etc.). Confirm whether the source has the prose: if yes, the gap is in extraction (selectors, structure, JS rendering), not content; if no, it's a documentation gap and the source needs an addition before any indexing fix matters.

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

Phase 3 outputs map directly to Phase 4 fixes: selector mismatches → crawler `recordProps` scoping, ID noise → Cheerio `.remove()` calls, JS-only content → `renderJavaScript: true` or custom records.

## Phase 4: Recommend Fixes

Each recommendation in this phase is conditional on a specific diagnosis from Phases 0-3. Don't propose synonyms, custom records, or content edits as a checklist — propose them only when the evidence (deployed state from Phase 0, analytics from Phase 1, record inspection from Phase 2, crawler behavior from Phase 3) points to that fix.

### Verify named entities before proposing

Before proposing a synonym, migration entry, or content addition that names a specific prop, function, type, or API: grep the codebase to confirm the name exists (or existed). Don't fabricate legacy names from no-result analytics — single-occurrence misspellings of real props are common, and proposing migration entries for invented props erodes trust fast.

### Empty-preview heading-record pattern (diagnostic pointer)

**Pattern to watch for**: queries with multiple top hits showing the same hierarchy breadcrumb but no preview snippet (the record's `content` is `null`). DocSearch creates separate records for headings (`type: lvl2`/`lvl3`) and body text (`type: content`). When a heading record and a content record share an anchor, the live `customRanking` (notably `desc(weight.level)`) and `attributeForDistinct` (often `"url"`, anchor-included) decide which one survives the dedup pass.

Don't apply a fix blindly. Inspect the record set for the failing query, compare which `type` ranks first, then read the live `customRanking`, `searchableAttributes`, and `attributeForDistinct` values from Phase 0 and reason about how they interact. Possible fixes range from small (tiebreaker reorder) to invasive (search-attribute reorder, distinct-attribute change). Always validate with a snapshot diff (next subsection) before keeping any change.

### Before mutating live index settings

Settings changes apply at query time — no re-crawl needed, and revert is one save in the dashboard. But on a single live index, "test then keep or revert" is the only workflow. Capture a baseline first.

1. **BEFORE snapshot**: pick the top 25–30 highest-volume queries from Phase 1 analytics (plus any query you specifically intend to fix). Record the top 5 hits each — `type`, full `hierarchy`, `url`, `anchor`. Save to file.
2. **Apply the change** in the dashboard.
3. **AFTER snapshot**: rerun the same script.
4. **Diff**.

Acceptable changes: `type` swaps with same URL/anchor, ordering shifts within a page, content records replacing empty-preview heading records on the same anchor.

Unacceptable (revert immediately): URL or anchor changes on top hits, previously-top pages dropping out of top 5, page A's content now landing on page B's anchor.

Minimal snapshot loop:

```python
import json, urllib.request, sys
APP, KEY, INDEX = "...", "...", "..."
QUERIES = ["term1", "term2", "term3"]  # top-volume queries
label = sys.argv[1] if len(sys.argv) > 1 else "snapshot"
print(f"=== {label} ===")
for q in QUERIES:
    body = json.dumps({"query": q, "hitsPerPage": 5,
                       "attributesToRetrieve": ["type","hierarchy","url","anchor"]}).encode()
    req = urllib.request.Request(
        f"https://{APP}-dsn.algolia.net/1/indexes/{INDEX}/query",
        data=body,
        headers={"X-Algolia-Application-Id": APP, "X-Algolia-API-Key": KEY,
                 "Content-Type": "application/json"})
    d = json.loads(urllib.request.urlopen(req, timeout=15).read().decode("utf-8","replace"), strict=False)
    print(f"--- {q!r} (nbHits={d['nbHits']}) ---")
    for i, h in enumerate(d["hits"], 1):
        hier = h.get("hierarchy") or {}
        h2 = hier.get("lvl2") or "-"
        h3 = hier.get("lvl3") or "-"
        print(f"  {i}. {h['type']:<7} {h2:<30} {h3:<35} {h['url']}")
```

Run with `python3 snapshot.py BEFORE > before.txt`, apply the change, then `python3 snapshot.py AFTER > after.txt`, then `diff -u before.txt after.txt`.

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
