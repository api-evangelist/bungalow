# Bungalow

Bungalow is a US residential rental platform for single-family homes, operating both a co-living
product (individual rooms in a house leased to separately screened housemates) and a whole-home
product (a house leased to a single group). Founded in 2017 by Andrew Collins and Justin McCarty,
headquartered in Miami, FL, active in roughly 23 US markets.

- Website: https://bungalow.com/
- API reference: https://fieldstone.bungalow.com/api/v1/docs/
- Live OpenAPI: https://fieldstone.bungalow.com/api/v1/open-api-schema/

## The API

Bungalow publishes a small, **anonymous**, read-mostly public REST API so that third-party listing
sites and rental portals can syndicate its inventory. Seven operations across four tags — Markets,
Listings, Showings, Applications. Verified callable with no credentials on 2026-08-01.

The spec is not linked from bungalow.com's navigation, robots.txt or sitemap. It is reachable only
via a single sentence on the About page pointing at `fieldstone.bungalow.com/api/v1/docs/`, whose
Redoc shell loads the real document from `/api/v1/open-api-schema/`.

| | |
|---|---|
| Base URL | `https://fieldstone.bungalow.com/api/v1/` |
| Auth | none (public JSON API) |
| Pagination | `limit` (default 20, max 100) / `offset`; `{results, count, next, previous}` |
| Versioning | URI path `/api/v1/`, semver within major versions |
| Errors | proprietary `{"error": {code, type, message}}` envelope — **not** RFC 9457 |
| Idempotency | none |
| Events | none outbound; one inbound partner lead-capture webhook |

Partner-gated alternatives: authenticated **Hotpads** and **Facebook Catalog** XML feeds carrying a
richer field set than the public API (MITS listed as possible future support). Request via
`integrations@bungalow.com`.

## Artifacts

| Dir | File | Method |
|---|---|---|
| `openapi/` | `bungalow-openapi-original.yml` | searched (verbatim, 105 KB, OpenAPI 3.0.2) |
| `overlays/` | `bungalow-openapi-original-overlay.yaml` | generated |
| `authentication/` | `bungalow-authentication.yml` | searched |
| `conventions/` | `bungalow-conventions.yml` | searched |
| `errors/` | `bungalow-problem-types.yml` | searched (14 error types) |
| `data-model/` | `bungalow-data-model.yml` | derived (9 entities, 13 relationships) |
| `lifecycle/` | `bungalow-lifecycle.yml` | searched |
| `conformance/` | `bungalow-conformance.yml` | derived |
| `asyncapi/` | `bungalow-webhooks.yml` | searched (inbound webhook only) |
| `mcp/` | `bungalow-mcp.yml`, `bungalow-tool-crosswalk.yml` | derived (candidate) |
| `skills/` | 3 skills + `_index.yml` | generated |
| `llms/` | `bungalow-llms.txt` | generated |
| `well-known/` | `bungalow-well-known.yml` | probed (nothing found) |
| `security/` | `bungalow-domain-security.yml` | probed |

## Known gaps in the provider's contract

1. **No error responses in the spec.** All seven operations declare only `200`, while the docs
   describe fourteen error classes in prose. Highest-leverage fix available.
2. **No `components.schemas`.** Every schema is inline and duplicated — the Market object appears in
   full four times.
3. **Path-shaped operationIds** (`/listings/properties/`) rather than verb-noun identifiers.
4. **Empty `info.version`.**
5. **No operation summaries** — descriptions only.
6. **Id wire-type divergence** — declared `integer`, returned as decimal strings.
7. No security.txt, no vulnerability disclosure program, no trust center, no status page, no
   changelog, no deprecation policy, no SDKs, no CLI, no sandbox, no DNSSEC, no CAA.
