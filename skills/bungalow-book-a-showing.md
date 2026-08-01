---
name: Book a Bungalow property showing
description: >-
  Find a property, read its bookable timeslot grid, and create an in-person or virtual showing —
  handling the taken-slot race, the missing idempotency contract, and the human-confirmation
  requirement that comes with booking a real appointment.
api: openapi/bungalow-openapi-original.yml
base_url: https://fieldstone.bungalow.com/api/v1/
operations:
  - "/markets/"
  - "/listings/properties/"
  - "/listings/showings/availability/{id}/"
  - "/listings/showings/"
generated: '2026-08-01'
method: generated
source: https://fieldstone.bungalow.com/api/v1/docs/
---

# Book a Bungalow property showing

## Before you start

This skill **writes**. `POST /listings/showings/` creates a real appointment with a Bungalow showing
agent and triggers messaging to the attendee. There is **no idempotency key** anywhere in this API,
so a blind retry books a second showing. Get explicit human confirmation of the property, the
timeslot and the attendee's contact details before the POST.

Authentication: none. The API is anonymous.

## Step 1 — resolve the property

    GET /markets/?limit=100
    GET /listings/properties/?market__slug=<slug>&limit=100

Operations: `/markets/`, `/listings/properties/`

Take the property `id`. Keep it as a **string** — ids are declared integer but returned as decimal
strings.

Useful pre-filters on the listing record: `showings_available`, `showing_today`,
`automated_tour_compatible`, `is_meet_and_greet_available`, `showdigs_showing_enabled`.

## Step 2 — read the availability grid

    GET /listings/showings/availability/{id}/

Operation: `/listings/showings/availability/{id}/`

`{id}` is the **property** id, not a showing id.

Optional parameters:

- `is_virtual=true` — returns timeslots for virtual showings; this changes the delta for the first
  available slot.
- `is_soon` — `true` shows all timeslots, `false` shows the less popular ones.
- `limit` / `offset` — standard pagination.

The response gives you `timezone_name`, `timezone_utc_offset`, and `availability_periods[]`, each
with `day_start` and a `timeslots[]` array. Each timeslot carries `start_time`, `end_time`,
`job_id` and `resource_ids`. There may also be an `instant_chat_available` flag with an
`instant_chat_timeslot`.

Render times in the market's timezone, not the caller's.

## Step 3 — confirm with the human, then book

    POST /listings/showings/
    Content-Type: application/json

    {
      "property": "<property id>",
      "start_time": "<timeslot.start_time>",
      "end_time": "<timeslot.end_time>",
      "user_name": "...",
      "user_email": "...",
      "user_phonenumber": "...",
      "is_virtual": false
    }

Operation: `/listings/showings/`

Required: `property`, `start_time`, `end_time`. Copy `start_time`/`end_time` **verbatim** from the
timeslot you selected.

Optional and worth setting:

- `job_id` — pass the timeslot's `job_id` to join an existing showing job rather than opening a new
  one (this is how group showings are batched).
- `is_virtual` — video links are sent out and reminder copy changes.
- `is_group` — group-based rather than individual showings for this property.
- `user_short_bio` — helps the co-living matching conversation.
- `utm_source`, `utm_campaign`, `utm_medium`, `utm_term`, `utm_content`, `gclid` — attribution.

The endpoint also accepts `application/x-www-form-urlencoded` and `multipart/form-data`.

The response returns `id`, `video_link` (for virtual showings), `fsa_name` and `fsa_photo` (the
showing agent), `market_contact_email`, `attendee_name`, `attendee_email` and `showing_start_time`.
Surface `video_link` and the agent's name back to the human.

## Handling the taken-slot race

Availability is read-then-write with no reservation, so a slot can go between step 2 and step 3.
Bungalow signals this explicitly:

    {
      "error": {
        "code": 410,
        "type": "APIGoneError",
        "message": "The requested resource no longer exists.",
        "non_field_errors": ["This timeslot is no longer available. Please select another time."]
      }
    }

On `410` — **re-fetch the availability grid, present the new options, and ask again.** Never loop
retrying the same slot.

On a network timeout with no response, do **not** re-POST. Re-read the availability grid first; if
your slot has disappeared, the booking probably succeeded.

## Other errors

| Status | `type` | What to do |
|---|---|---|
| 400 | `ValidationError` | Inspect `field_errors` — most often a malformed `start_time`/`end_time` |
| 404 | `NotFound` | Bad property id |
| 409 | `Conflict` | Dereference `conflict.location` — you already created this resource |
| 415 | `UnsupportedMediaType` | Use JSON, form-urlencoded or multipart |
| 429 | `Throttled` | Honor `Retry-After` |

Full catalog: `errors/bungalow-problem-types.yml`.
