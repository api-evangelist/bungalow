# Bungalow

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
