---
name: Curate SpringServe deal lists and marketplaces
description: Build and maintain the deal lists and curated marketplaces that package inventory for buyers in SpringServe, using the bulk deal operations rather than one call per deal.
api: openapi/magnite-springserve-v1-openapi.yml
base_url: https://console.springserve.com
operations: [deal_lists_get, deal_lists_post, deal_lists_deal_list_id_get, deal_lists_deal_list_id_patch, deal_lists_deal_list_id_delete, deal_lists_deal_list_id_deals_get, deal_lists_deal_list_id_deals_bulk_create_post, deal_lists_deal_list_id_deals_bulk_replace_post, deal_lists_deal_list_id_deals_bulk_delete_delete, deal_lists_bulk_update_same_attributes_post, deal_lists_permissions_get, curated_marketplaces_get, curated_marketplaces_post, curated_marketplaces_id_get, curated_marketplaces_id_patch, curated_marketplaces_id_delete, curated_marketplaces_global_get]
---

# Curate deal lists and marketplaces

Deal lists group deal IDs into a reusable package; curated marketplaces are the productised
inventory bundles built on top. Authenticate and confirm the active account first
(`magnite-authenticate-and-set-account.md`), then check
`GET /api/v1/deal_lists/permissions` (`deal_lists_permissions_get`).

## Deal lists

1. `GET /api/v1/deal_lists` (`deal_lists_get`) lists them; `page`/`per`, 50 per page by default.
2. `POST /api/v1/deal_lists` (`deal_lists_post`) creates one.
3. `GET /api/v1/deal_lists/{deal_list_id}` (`deal_lists_deal_list_id_get`) reads it;
   `PATCH` (`deal_lists_deal_list_id_patch`) renames or re-scopes it.
4. `GET /api/v1/deal_lists/{deal_list_id}/deals` (`deal_lists_deal_list_id_deals_get`) reads the
   member deals.

## Change the membership in bulk — this is the important part

Three distinct bulk operations exist, and choosing the wrong one is the common mistake:

- `POST .../deals/bulk_create` (`deal_lists_deal_list_id_deals_bulk_create_post`) — **adds**
  deals, leaving existing members alone.
- `POST .../deals/bulk_replace` (`deal_lists_deal_list_id_deals_bulk_replace_post`) —
  **replaces the entire membership**. Anything not in your payload is removed. Read the current
  membership first and confirm with a human before using this.
- `DELETE .../deals/bulk_delete` (`deal_lists_deal_list_id_deals_bulk_delete_delete`) — removes
  the named deals.

Use these instead of looping single calls: the account-wide budget is 240 requests/minute and a
per-deal loop burns it for no benefit.

## Curated marketplaces

- `GET /api/v1/curated_marketplaces` (`curated_marketplaces_get`) for the active account;
  `curated_marketplaces_global_get` to see them across all accounts you can reach.
- `POST` (`curated_marketplaces_post`), `PATCH` (`curated_marketplaces_id_patch`),
  `DELETE` (`curated_marketplaces_id_delete`) to manage one.
- `curated_marketplaces_id_permissions_get` before writing.

## Safety

No idempotency key exists on this API. A retried `bulk_create` after a timeout can double-add
deals — re-read `.../deals` before retrying. A retried `bulk_replace` is naturally idempotent in
effect (the payload defines the final state), which is the one place a retry is safe.
