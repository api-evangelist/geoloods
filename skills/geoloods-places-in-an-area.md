---
generated: '2026-09-20'
method: generated
name: List places in an area
description: Enumerate named places inside a bounding box with GET /v2/bbox, or within a radius with GET /v2/nearby, and enrich country context with GET /v2/countries.
api: openapi/geoloods-openapi.json
operations: [searchBbox, searchNearby, listCountries]
source: >-
  Grounded in openapi/geoloods-openapi.json (OpenAPI 3.0.3, captured 2026-09-20 from
  https://geoloods.io/openapi.json) and https://geoloods.io/llms-full.txt. Every operationId verified
  verbatim in the spec.
---

# List places in an area

Get every named gazetteer place in a region — as a rectangle (bbox) or a circle (nearby) — and resolve country metadata.

## Auth
- Base URL: `https://api.geoloods.io/v2/`. Send `X-API-Key` (or `api_key` query). See `authentication/geoloods-authentication.yml`.

## Steps

1. **Bounding box** — `searchBbox` (`GET /v2/bbox?min_lat=59.8&max_lat=60.0&min_lng=10.6&max_lng=10.9`). Required: `min_lat`, `max_lat`, `min_lng`, `max_lng`. Optional `limit`, `lang`. The box may be at most **10 degrees on a side**.
2. **Or radius** — `searchNearby` (`GET /v2/nearby?lat=59.9139&lng=10.7522&radius_km=50`). Required `lat`, `lng`; optional `radius_km` (default 10, range 0.1-100), `limit`, `lang`. Sorted by distance.
3. **Enrich country context** — `listCountries` (`GET /v2/countries`) returns country objects (`code` ISO 3166-1 alpha-2, `name`, `iso3`, `iso_numeric`, `capital`, `area`, `population`, `continent`, `tld`, `currency_code`, `currency_name`). Join on the `CountryCode` field each place already carries.

```
curl -sS 'https://api.geoloods.io/v2/bbox?min_lat=59.8&max_lat=60.0&min_lng=10.6&max_lng=10.9' \
  -H 'X-API-Key: YOUR_API_KEY'
```

## Rules an agent must follow

- **Respect the ceilings.** bbox rejects boxes larger than 10 degrees per side; nearby caps `radius_km` at 100. Split a larger area into tiles/rings rather than sending one oversized request.
- **Use `limit`** to bound result-set size; there is no cursor pagination, so a very dense area is capped by `limit`, not paged (`conventions/geoloods-conventions.yml`).
- **Places already carry country context** inline (CountryName, CountryFlag, CurrencyCode). Only call `listCountries` when you need the full country record.
- **Responses are bare arrays;** an empty array is a valid empty region.
- **Errors are flat `{"error": string}`** (not RFC 9457); `401` = missing/invalid key. See `errors/geoloods-problem-types.yml`.
- **Rate-limit headers** `X-RateLimit-*` govern the monthly plan quota; back off at `X-RateLimit-Remaining` 0. See `rate-limits/geoloods-rate-limits.yml`.
