---
name: Authenticate to SpringServe and select the working account
description: Mint a SpringServe API token, confirm the session, and set the active account before doing anything else. Every other SpringServe skill depends on this one, because the API resolves requests against server-side active-account state rather than an account parameter.
api: openapi/magnite-springserve-v1-openapi.yml
base_url: https://console.springserve.com
operations: [auth_post, session_get, accounts_current_get, accounts_displayable_get, accounts_id_set_current_post, accounts_permissions_get]
---

# Authenticate and select the working account

## Why this comes first

The SpringServe UI API does not take an account id on most requests. It executes every call in
the context of the **currently active account** for the authenticated user, which is server-side
state. The same token, sending the identical request, returns different data depending on which
account was made active by an earlier call. Do not skip this skill and do not assume the default
active account is the one the user meant.

## Steps

1. **Mint a token.** `POST /api/v1/auth` (`auth_post`) with a
   `application/x-www-form-urlencoded` body of `email` and `password`. The response carries the
   API token. There is no OAuth flow and no client-credentials grant — this is a credential
   exchange, so treat the password as a secret that never enters a log, a prompt, or a tool
   argument that gets echoed back.
2. **Send it on every subsequent call** as either
   `Authorization: <token>` (scheme `api_key`) or `Authorization: Bearer <token>` (scheme
   `bearer_token`, a JWT). Both are declared in the spec; pick one and be consistent.
3. **Confirm the session** with `GET /api/v1/session` (`session_get`). A 401 here returns
   `{"error":"user not authorized"}` — the token is wrong or expired, not the request.
4. **Read the active account** with `GET /api/v1/accounts/current`
   (`accounts_current_get`). Report it to the user before you write anything.
5. **If it is the wrong account**, list what is selectable with
   `GET /api/v1/accounts/displayable` (`accounts_displayable_get`), then switch with
   `POST /api/v1/accounts/{id}/set_current` (`accounts_id_set_current_post`). For a Magnite DV+
   identifier instead of a SpringServe id, use
   `accounts_dvplus_dvplus_id_set_current_post`.
6. **Check what you are allowed to do** with `GET /api/v1/accounts/permissions`
   (`accounts_permissions_get`) before attempting writes. Most collections also expose their own
   `..._permissions_get` operation.
7. **Close out** with `DELETE /api/v1/session` (`session_delete`) when the task is done and the
   token was minted for this task alone.

## Rules that apply to every SpringServe call

- **No idempotency.** There is no `Idempotency-Key` header anywhere in this API. A retried
  `POST` creates a second object. If a write times out, do **not** blindly retry — re-read the
  collection and check whether the object already exists.
- **Rate limits are documented but unsignalled.** 240 requests/minute per account overall,
  10/minute against the reporting endpoint, 3/minute for reporting by domain or app bundle. No
  `X-RateLimit-*` or `Retry-After` headers come back, so budget locally; you will not be warned.
- **Errors are not RFC 9457.** Expect `{"error": "<message>"}`, or on validation failure
  `{"status": <int>, "error": "<summary>", "errors": {"<field>": ["<message>"]}}` with HTTP 422.
- **Every response carries `x-request-id`.** Capture it and quote it when reporting a failure to
  Magnite support; it is the only correlation handle this API offers.
- **Two live versions.** `/api/v1` is current but does not yet cover every `/api/v0` endpoint. If
  a resource is missing from v1, check `openapi/magnite-springserve-v0-openapi.yml` before
  concluding it does not exist.
