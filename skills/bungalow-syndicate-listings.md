---
name: Syndicate Bungalow rental listings
description: >-
  Pull Bungalow's active markets and every marketable property in a chosen market, branch correctly
  between co-living and whole-home pricing, and keep a downstream listings surface fresh without
  violating Bungalow's syndication rules.
api: openapi/bungalow-openapi-original.yml
base_url: https://fieldstone.bungalow.com/api/v1/
operations:
  - "/markets/"
  - "/listings/properties/"
  - "/listings/properties/{id_or_slug}"
generated: '2026-08-01'
method: generated
source: https://fieldstone.bungalow.com/api/v1/docs/
---

# Syndicate Bungalow rental listings

## Authentication

None. The public JSON API is anonymous — send no `Authorization` header and no key. If you need the
extra fields the public API omits, email `integrations@bungalow.com` for an authenticated Hotpads or
Facebook Catalog XML feed instead.

## Step 1 — resolve markets (mandatory first call)

    GET /markets/?limit=100

Operation: `/markets/`

You cannot skip this. `/listings/properties/` requires a `market__slug` query parameter and there is
no global property enumeration.

Read `results[].slug` (e.g. `atlanta`, `bay-area`, `seattle`). Note `results[].timezone` — you need
it to render availability later. There were 23 active markets as of 2026-08-01.

**Wire-type trap:** `id` is declared `type: integer` in the spec but comes back as a decimal string
(`"715194178910805703"`). Keep it as a string.

## Step 2 — list marketable properties

    GET /listings/properties/?market__slug=atlanta&limit=100&offset=0

Operation: `/listings/properties/`

Paginate with `limit` (default 20, **max 100**) and `offset`, following `next` until it is `null`.
Useful filters: `neighborhood__slug`, `marketing_type` (`co_living` | `group_living`), `min_price`,
`max_price`, `pet_friendly`, `private_bathroom`, `washer`, `dryer`, `backyard`, `pool`,
`is_featured`, `is_coming_soon`.

## Step 3 — branch on the product type

This is the single most important rule in the API:

- `property_marketing_type == "group_living"` → whole home. Price from **`full_property_price`**.
  `street_address` is the real full address.
- `property_marketing_type == "co_living"` → individual rooms. Price from the **`room_prices`**
  array. The address is **deliberately obfuscated** and latitude/longitude fidelity is reduced.
  Do not attempt to re-identify it, and do not publish it as a precise address.

## Step 4 — enrich the ones you will publish

    GET /listings/properties/{id_or_slug}

Operation: `/listings/properties/{id_or_slug}`

Accepts either the numeric id or the slug. Returns fields the list operation omits:
`description_html`, `matterport_url`, `walkthrough_video_url`, `rooms`, `roommates`,
`roommate_living_preferences`, `house_profile`, `calendly_url`, `showings_available`.

## Step 5 — refresh on Bungalow's cadence

Bungalow's own guidance:

> Results are updated approximately every 10 minutes so any polling done on these endpoints should
> aim to match that timeframe where possible. Since pricing can change dynamically we recommend not
> waiting longer than 24 hrs to pull new results.

Responses carry `Cache-Control: max-age=1800`. There is **no webhook and no event stream** for
listing changes — polling is the only change-detection mechanism (see
`asyncapi/bungalow-webhooks.yml`).

## Syndication rule you must honor

> Bungalow requests that third-party listings sites only post a listing if there is less than 30
> days until the earliest availability date.

Filter on `earliest_available_date` before publishing.

## Errors

Every error is `application/json` with the envelope `{"error": {"code", "type", "message", ...}}` —
**not** RFC 9457. Handle at minimum:

| Status | `type` | What to do |
|---|---|---|
| 400 | `ValidationError` | Read `field_errors` / `non_field_errors`, fix, resubmit |
| 404 | `NotFound` | Bad market id or property id/slug — drop it from your set |
| 429 | `Throttled` | Honor the `Retry-After` header; no numeric quota is published |
| 500 | `ServerError` | Retry with backoff |

Full catalog: `errors/bungalow-problem-types.yml`.
