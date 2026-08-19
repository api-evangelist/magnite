---
name: Create and maintain SpringServe demand tags
description: List, create, duplicate, update and bulk-edit the demand tags that connect buyers to a publisher's inventory in SpringServe — the core write surface of the ad server, with no idempotency safety net.
api: openapi/magnite-springserve-v1-openapi.yml
base_url: https://console.springserve.com
operations: [demand_tags_get, demand_tags_new_get, demand_tags_post, demand_tags_id_get, demand_tags_id_patch, demand_tags_id_delete, demand_tags_id_duplicate_get, demand_tags_bulk_update_same_attributes_post, demand_tags_bulk_update_many_post, demand_tags_id_creatives_summary_get, demand_tags_macro_suggester_post, demand_tags_permissions_get, demand_tags_download_get]
---

# Create and maintain demand tags

A demand tag is how buyer demand is attached to inventory in the SpringServe ad server. This is
the highest-consequence write surface in the API: a bad demand tag changes what actually serves
and what a publisher earns. Authenticate and confirm the active account first
(`magnite-authenticate-and-set-account.md`).

## Before writing

- Check your rights: `GET /api/v1/demand_tags/permissions` (`demand_tags_permissions_get`) for
  the collection, `demand_tags_id_permissions_get` for one tag.
- **There is no idempotency key.** A retried create makes a duplicate demand tag that will serve
  alongside the original. Always re-list before retrying a write that did not return cleanly.

## Reading

- `GET /api/v1/demand_tags` (`demand_tags_get`) lists them, filterable by `date_range`,
  `currency`, `demand_label_id` and `demand_tag_status`. Paginate with `page` and `per`; the
  default page is 50 objects and nothing tells you the total.
- `GET /api/v1/demand_tags/{id}` (`demand_tags_id_get`) reads one.
- `GET /api/v1/demand_tags/download` (`demand_tags_download_get`) exports the list as CSV — one
  request instead of paging, when you want the whole set.
- `GET /api/v1/demand_tags/{id}/creatives_summary`
  (`demand_tags_id_creatives_summary_get`) shows what creative is actually running on the tag.

## Creating

1. **Get the defaults.** `GET /api/v1/demand_tags/new` (`demand_tags_new_get`) returns the
   server's defaults for a new demand tag. Start from these; do not hand-build a body.
2. **Read the shipped examples.** `POST /api/v1/demand_tags` (`demand_tags_post`) declares six
   worked example bodies in the spec — `demand_tag_tag_create_simple` / `_full`,
   `demand_tag_bidder_create_simple` / `_full`, `demand_tag_line_item_create_simple` / `_full`.
   Those three pairs are the three tag types; pick the one matching the buyer integration.
3. **Post it.** A success is `201`. A `422` returns per-field messages under `errors` — fix and
   resend; a 422 has not created anything, so resending is safe.
4. **Or clone an existing one.** `GET /api/v1/demand_tags/{id}/duplicate`
   (`demand_tags_id_duplicate_get`) copies a known-good tag, which is usually safer than
   authoring a new body from scratch.

## Updating

- One tag: `PATCH /api/v1/demand_tags/{id}` (`demand_tags_id_patch`).
- Same change across many: `POST /api/v1/demand_tags/bulk_update`
  (`demand_tags_bulk_update_same_attributes_post`).
- Different changes across many: `POST /api/v1/demand_tags/bulk/update`
  (`demand_tags_bulk_update_many_post`).
- Deleting: `DELETE /api/v1/demand_tags/{id}` (`demand_tags_id_delete`). Confirm with a human
  first — there is no undo, and no sandbox to rehearse in.

## Useful extras

- `POST /api/v1/demand_tags/macro_suggester` (`demand_tags_macro_suggester_post`) suggests VAST
  endpoint macros for a demand endpoint rather than making you infer them.
- `GET /api/v1/demand_tags/new_from_forecast/{id}` (`demand_tags_new_from_forecast_id_get`)
  seeds a new tag from a demand forecast.

## Prove what you changed

Every create, update and delete is captured in the object audit log. After a bulk edit, verify
with `GET /api/v1/changelogs?versioned_type=DemandTag&time=today`
(`changelogs_get`) — see `magnite-audit-springserve-changes.md`.
