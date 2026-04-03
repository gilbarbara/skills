# Algolia Analytics API Reference

All endpoints require these headers:
```
X-Algolia-Application-Id: {APP_ID}
X-Algolia-API-Key: {API_KEY}
```

The analytics API key is different from the search API key. It must have analytics permissions (admin key or a key with `analytics` ACL).

## Base URLs

- **Analytics**: `https://analytics.algolia.com`
- **Search/Index**: `https://{APP_ID}-dsn.algolia.net`

## Search Analytics

### Top searches

```bash
curl -s "https://analytics.algolia.com/2/searches?index={INDEX}&limit=50&clickAnalytics=true" \
  -H "X-Algolia-Application-Id: {APP_ID}" \
  -H "X-Algolia-API-Key: {ANALYTICS_KEY}"
```

Response fields per search: `search`, `count`, `nbHits`, `clickThroughRate`, `averageClickPosition`, `clickCount`, `trackedSearchCount`.

`clickThroughRate` and `averageClickPosition` are `null` unless `insights` is enabled in the DocSearch component and `trackedSearchCount > 0`.

### No-result searches

```bash
curl -s "https://analytics.algolia.com/2/searches/noResults?index={INDEX}&limit=50" \
  -H "X-Algolia-Application-Id: {APP_ID}" \
  -H "X-Algolia-API-Key: {ANALYTICS_KEY}"
```

These are the highest priority gaps. Each entry has `search` (query text) and `count` (how often it was searched).

### No-click searches

```bash
curl -s "https://analytics.algolia.com/2/searches/noClicks?index={INDEX}&limit=50" \
  -H "X-Algolia-Application-Id: {APP_ID}" \
  -H "X-Algolia-API-Key: {ANALYTICS_KEY}"
```

Results existed but nobody clicked. May indicate wrong results ranking first.

### Search count (daily)

```bash
curl -s "https://analytics.algolia.com/2/searches/count?index={INDEX}" \
  -H "X-Algolia-Application-Id: {APP_ID}" \
  -H "X-Algolia-API-Key: {ANALYTICS_KEY}"
```

Returns `count` (total) and `dates` array with per-day breakdown.

### User count (daily)

```bash
curl -s "https://analytics.algolia.com/2/users/count?index={INDEX}" \
  -H "X-Algolia-Application-Id: {APP_ID}" \
  -H "X-Algolia-API-Key: {ANALYTICS_KEY}"
```

## Click & Conversion Analytics

These require `insights={true}` in the DocSearch component. Data starts flowing after the flag is enabled — there's no retroactive data.

### Click-through rate

```bash
curl -s "https://analytics.algolia.com/2/clicks/clickThroughRate?index={INDEX}" \
  -H "X-Algolia-Application-Id: {APP_ID}" \
  -H "X-Algolia-API-Key: {ANALYTICS_KEY}"
```

Returns `rate` (overall), `trackedSearchCount`, `clickCount`, and daily breakdown.

Healthy CTR for docs search: 20-40%.

### Click positions

```bash
curl -s "https://analytics.algolia.com/2/clicks/positions?index={INDEX}" \
  -H "X-Algolia-Application-Id: {APP_ID}" \
  -H "X-Algolia-API-Key: {ANALYTICS_KEY}"
```

Returns position buckets with click counts. If 70%+ of clicks are at position 1, ranking is working well.

### Conversion rate

```bash
curl -s "https://analytics.algolia.com/2/conversions/conversionRate?index={INDEX}" \
  -H "X-Algolia-Application-Id: {APP_ID}" \
  -H "X-Algolia-API-Key: {ANALYTICS_KEY}"
```

Usually 0% for DocSearch since conversion events aren't configured by default.

## Index Queries

These use the **search API key** (public, read-only). No analytics key needed.

### Test a query

```bash
curl -s "https://{APP_ID}-dsn.algolia.net/1/indexes/{INDEX}/query" \
  -H "X-Algolia-Application-Id: {APP_ID}" \
  -H "X-Algolia-API-Key: {SEARCH_KEY}" \
  -d '{"query":"search term","hitsPerPage":3,"attributesToRetrieve":["hierarchy","content","url","type"]}'
```

### Record type distribution

```bash
-d '{"query":"","hitsPerPage":0,"facets":["type"]}'
```

Expected types: `content` (body text), `lvl1`-`lvl6` (headings). If the content count is 0, the body text isn't being indexed.

### Filter by type

```bash
-d '{"query":"","hitsPerPage":5,"facetFilters":["type:content"],"attributesToRetrieve":["content","hierarchy","url","type"]}'
```

### Scan for garbled anchors

```bash
-d '{"query":"","hitsPerPage":200,"attributesToRetrieve":["anchor","url"]}'
```

Then filter results for anchors matching auto-generated patterns: `^_R_`, `^:r`, random hashes.

### Batch test queries

```bash
for q in "term1" "term2" "term3"; do
  count=$(curl -s "https://{APP_ID}-dsn.algolia.net/1/indexes/{INDEX}/query" \
    -H "X-Algolia-Application-Id: {APP_ID}" \
    -H "X-Algolia-API-Key: {SEARCH_KEY}" \
    -d "{\"query\":\"$q\",\"hitsPerPage\":0}" \
    | python3 -c "import sys,json; print(json.load(sys.stdin)['nbHits'])")
  echo "$q -> $count hits"
done
```

## Date Filtering

Most analytics endpoints accept `startDate` and `endDate` query params (format: `YYYY-MM-DD`):

```bash
curl -s "https://analytics.algolia.com/2/searches?index={INDEX}&startDate=2026-03-01&endDate=2026-03-31" \
  -H "X-Algolia-Application-Id: {APP_ID}" \
  -H "X-Algolia-API-Key: {ANALYTICS_KEY}"
```

The default range is the last 7 days.
