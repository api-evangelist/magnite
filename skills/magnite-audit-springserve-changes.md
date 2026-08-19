---
name: Audit who changed what in SpringServe
description: Query the SpringServe object-change log to answer "what changed, when, and who did it" across 40+ object types — the API's built-in audit trail, and the verification step after any bulk write.
api: openapi/magnite-springserve-v1-openapi.yml
base_url: https://console.springserve.com
operations: [changelogs_get, changelogs_id_get, changelogs_user_emails_get, changelogs_versioned_types_for_current_account_get, changelogs_versioned_type_versioned_type_get, changelogs_versioned_type_versioned_type_versioned_id_get]
---

# Audit who changed what

SpringServe exposes a first-class audit trail as a normal API resource. This is a **log of
object changes** — created, updated, destroyed — not a product release-notes changelog. Use it
to verify your own writes and to investigate unexplained delivery changes.

Authenticate and confirm the active account first
(`magnite-authenticate-and-set-account.md`); the log is scoped to the active account.

## Steps

1. **Find out what is auditable here.**
   `GET /api/v1/changelogs/versioned_types_for_current_account`
   (`changelogs_versioned_types_for_current_account_get`) returns the object types the active
   account actually has — Campaign, Creative, Deal, SupplyTag, DemandTag, Segment and 40-odd
   others. Do not guess a `versioned_type` string; read it from here.
2. **Query the log.** `GET /api/v1/changelogs` (`changelogs_get`) with any of:
   - `change_type` — `created`, `updated`, `destroyed`
   - `time` — `today`, `yesterday`, `last_7_days`, `last_30_days`, `last_year`
   - `user` — an email or id, **or `exclude_system_updates`**, which is the one to reach for
     first: automated system writes otherwise drown the human changes you are looking for
   - `versioned_type` — the object type from step 1
3. **Narrow to one object type** with
   `GET /api/v1/changelogs/versioned_type/{versioned_type}`
   (`changelogs_versioned_type_versioned_type_get`), or to **one specific object** with
   `GET /api/v1/changelogs/versioned_type/{versioned_type}/{versioned_id}`
   (`changelogs_versioned_type_versioned_type_versioned_id_get`). That last one is the fastest
   answer to "why did this supply tag stop delivering".
4. **Resolve the actors.** `GET /api/v1/changelogs/user_emails`
   (`changelogs_user_emails_get`) lists the users who appear in the log, so you can turn ids
   into people.
5. **Read one entry in full** with `GET /api/v1/changelogs/{id}` (`changelogs_id_get`).

## Notes

- Paginate with `page` and `per`; 50 entries per page by default, and no total count is
  returned, so keep requesting until a page comes back short.
- This is a read-only surface — safe for an agent to use without confirmation, and the right
  first move when investigating before touching anything.
- Run it immediately after any bulk demand-tag or deal-list edit to confirm the change landed on
  exactly the objects you intended.
