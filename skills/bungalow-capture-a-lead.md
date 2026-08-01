---
name: Capture a Bungalow rental lead
description: >-
  Hand a prospective renter off to Bungalow correctly — either synchronously via the public
  application-source endpoint, or via the partner lead-capture webhook — with the consent and PII
  handling that submitting a named person's details demands.
api: openapi/bungalow-openapi-original.yml
base_url: https://fieldstone.bungalow.com/api/v1/
operations:
  - "/markets/"
  - "/listings/properties/"
  - "/applications/source/"
generated: '2026-08-01'
method: generated
source: https://fieldstone.bungalow.com/api/v1/docs/
---

# Capture a Bungalow rental lead

## Before you start

This skill submits a **named individual's personal data** — name, email, sometimes phone, budget,
employer and school — into Bungalow's leasing funnel. Do not call it without that person's explicit,
informed consent. It is not idempotent: a repeated call creates a second application source.

Authentication: none. The docs are explicit — "Being signed in is not required."

## Two routes, pick the right one

| Route | Direction | Who can use it | When |
|---|---|---|---|
| `POST /applications/source/` | you → Bungalow, synchronous | anyone, anonymously | Default. Returns a public application URL you can send the renter to. |
| Lead-capture webhook | you → Bungalow, async | approved partners only | You run a listings site and Bungalow has provisioned an endpoint for you. Request via `integrations@bungalow.com`. |

## Route A — application source (public, synchronous)

### Step 1 — resolve the market (required field)

    GET /markets/?limit=100

Operation: `/markets/`

`market` is a **required** field on the application. Take the market `id` (keep it as a string).

### Step 2 — optionally resolve the property

    GET /listings/properties/?market__slug=<slug>&limit=100

Operation: `/listings/properties/`

`property` is optional but strongly worth setting when the lead came from a specific listing.

### Step 3 — submit

    POST /applications/source/
    Content-Type: application/json

    {
      "first_name": "...",
      "last_name": "...",
      "email": "...",
      "market": "<market id>",
      "property": "<property id>",
      "extra": { }
    }

Operation: `/applications/source/`

Required: `first_name`, `last_name`, `email`, `market`.
Optional: `middle_name`, `property`, `extra` (free-form).

Also accepts `application/x-www-form-urlencoded` and `multipart/form-data`.

The response is a public view of the application source carrying a `url`. **Send the renter to that
URL** — it is the continuation of their application, not an internal link.

## Route B — partner lead-capture webhook

Bungalow's ask:

> When a lead shows interest in one of our properties on your platform, in order to have the best
> possible lead experience, we ask that you send us a webhook with the lead data included.

The endpoint is provisioned per partner and is not published — request one at
`integrations@bungalow.com`. It accepts JSON or url-form-encoded bodies; casing is flexible but
`snake_case` is recommended.

Required fields: `property_id`, `name` (or `first_name` + `last_name`), `email`.
Optional fields: `phone`, `bio`, `max_budget`, `desired_move_in_date`, `student`, `work`, `school`,
`hobbies`.

There is no documented signing scheme and no documented retry policy — build your own delivery
guarantees and keep an outbox. Details: `asyncapi/bungalow-webhooks.yml`.

## Errors

| Status | `type` | What to do |
|---|---|---|
| 400 | `ValidationError` | `field_errors` names the bad field — usually `email` or a missing `market` |
| 404 | `NotFound` | Bad market or property id |
| 409 | `Conflict` | You already submitted this lead. Dereference `conflict.location`; do not re-POST |
| 429 | `Throttled` | Honor `Retry-After` |
| 500 | `ServerError` | Retry with backoff — but only once, and de-duplicate on your side |

Because there is no idempotency key, keep your own submission ledger keyed on
`(email, property_id)` so a retry storm does not flood a leasing team with duplicate leads.

Full catalog: `errors/bungalow-problem-types.yml`.
