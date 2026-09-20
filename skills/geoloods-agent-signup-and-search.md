---
generated: '2026-09-20'
method: generated
name: Sign up as an agent and search for a place
description: Mint an unauthenticated gl_agent_* trial key with POST /v2/agent/signup, then resolve a named place to coordinates with GET /v2/search.
api: openapi/geoloods-openapi.json
operations: [agentSignup, searchPlaces]
source: >-
  Grounded in openapi/geoloods-openapi.json (OpenAPI 3.0.3, captured 2026-09-20 from
  https://geoloods.io/openapi.json) and https://geoloods.io/llms-full.txt. Every operationId verified
  verbatim in the spec. Auth per authentication/geoloods-authentication.yml, errors per
  errors/geoloods-problem-types.yml, limits per rate-limits/geoloods-rate-limits.yml.
---

# Sign up as an agent and search for a place

Geoloods is agent-native: an agent can obtain a working key with no browser, email, or human account, then immediately search the gazetteer.

## Auth
- Base URL: `https://api.geoloods.io/v2/`.
- Signup is unauthenticated. Every other endpoint needs the key in the `X-API-Key` header (or `api_key` query param). See `authentication/geoloods-authentication.yml`.

## Steps

1. **Mint a trial key** — `agentSignup` (`POST /v2/agent/signup`). Body is optional; `{}` is valid, or send `{"agent":"your-agent","purpose":"short intent"}` (agent max 64 chars, purpose max 200). A `201` returns `api_key` (prefix `gl_agent_`), `quota` (1000), `remaining`, `expires_at`, and `ttl_seconds` (43200 = 12h).
2. **Search a named place** — `searchPlaces` (`GET /v2/search?query=Oslo`). Required param is `query` (the live param is `query`, NOT `q`). Optional `lang` (e.g. `en`, `nl`). Send the key as `X-API-Key`. Returns a bare JSON array of Location objects (Geonameid, Name, Latitude, Longitude, CountryCode, CountryName, Timezone, ...).

```
curl -sS -X POST https://api.geoloods.io/v2/agent/signup \
  -H 'Content-Type: application/json' -d '{}'

curl -sS 'https://api.geoloods.io/v2/search?query=Oslo' \
  -H 'X-API-Key: gl_agent_...'
```

## Rules an agent must follow

- **One active agent key per client IP.** If you already hold an active trial key, a second `POST /v2/agent/signup` returns `429` with `retry_after_seconds` and the existing key's `expires_at` — the secret is NOT re-revealed. Cache the key from step 1; do not re-mint on every run. See `rate-limits/geoloods-rate-limits.yml`.
- **The trial is not free forever.** After `ttl_seconds` (12h) the key is rejected; create a human plan at https://geoloods.io/auth (Free 1,000/mo, Pro, Scale).
- **Named places only.** `query` resolves cities, towns, villages, parks, landmarks — not streets, addresses, or café/hotel names. Venue-like queries may return nothing or an unrelated place; for "what's around" use `searchNearby`.
- **No idempotency key exists**, but signup is IP-scoped so retries do not mint duplicate keys (`conventions/geoloods-conventions.yml`). Read endpoints are GETs and safe to retry.
- **Errors are a flat `{"error": string}`**, not RFC 9457. A `401` means a missing/invalid key. See `errors/geoloods-problem-types.yml`.
- **Rate-limit headers** `X-RateLimit-Limit` / `X-RateLimit-Remaining` / `X-RateLimit-Reset` (unix seconds) are on responses; back off when remaining hits 0.
