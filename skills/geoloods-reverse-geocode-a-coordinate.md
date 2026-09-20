---
generated: '2026-09-20'
method: generated
name: Reverse geocode a coordinate
description: Turn a lat/lng into a named place with GET /v2/geocode, choosing settlement-preferring (smart) or nearest-feature (simple) behavior, and fall back to /v2/nearby for venues.
api: openapi/geoloods-openapi.json
operations: [reverseGeocode, searchNearby]
source: >-
  Grounded in openapi/geoloods-openapi.json (OpenAPI 3.0.3, captured 2026-09-20 from
  https://geoloods.io/openapi.json) and https://geoloods.io/llms-full.txt. Every operationId verified
  verbatim in the spec. Auth per authentication/geoloods-authentication.yml, errors per
  errors/geoloods-problem-types.yml.
---

# Reverse geocode a coordinate

Resolve a latitude/longitude to the named gazetteer place that describes it.

## Auth
- Base URL: `https://api.geoloods.io/v2/`. Send `X-API-Key` (or `api_key` query). See `authentication/geoloods-authentication.yml`.

## Steps

1. **Reverse geocode** — `reverseGeocode` (`GET /v2/geocode?lat=52.370216&lng=4.895168`). Required: `lat`, `lng`. Optional `filter`, `results`, `lang`.
   - `filter=smart` (default) — prefers settlement and admin place labels; HTL/RSRT/MALL excluded. Use this when you want "which town is this".
   - `filter=simple` — opt-in nearest gazetteer feature (may be a hotel). Use this when you want the literal closest feature.
2. **Want venues / "what's around" instead?** — `searchNearby` (`GET /v2/nearby?lat=59.9139&lng=10.7522&radius_km=50`). Required `lat`, `lng`; optional `radius_km` (default 10, range 0.1-100), `limit`, `lang`. Returns named places within the radius sorted by distance, including hotels/venues when you want them.

```
curl -sS 'https://api.geoloods.io/v2/geocode?lat=52.370216&lng=4.895168' \
  -H 'X-API-Key: YOUR_API_KEY'

curl -sS 'https://api.geoloods.io/v2/geocode?lat=52.370216&lng=4.895168&filter=simple' \
  -H 'X-API-Key: YOUR_API_KEY'
```

## Rules an agent must follow

- **Pick the filter deliberately.** `smart` and `simple` answer different questions. Defaulting to `smart` avoids surprise hotel results; switch to `simple` only when the nearest raw feature is what you need.
- **Hotels/venues belong on `/nearby`, not `/geocode`.** `geocode` with `smart` deliberately excludes HTL/RSRT/MALL.
- **Responses are bare arrays** of Location objects; an empty array is a valid "nothing near here" answer, not an error.
- **Errors are a flat `{"error": string}`** (not RFC 9457); `401` = missing/invalid key. See `errors/geoloods-problem-types.yml`.
- **Read-only and safe to retry.** These are GETs; there is no idempotency key because none is needed (`conventions/geoloods-conventions.yml`).
- **Watch `X-RateLimit-Remaining`** and back off before it reaches 0; the window resets at the `X-RateLimit-Reset` unix timestamp. See `rate-limits/geoloods-rate-limits.yml`.
