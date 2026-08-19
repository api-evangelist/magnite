---
name: Run a SpringServe performance report
description: Run an ad-serving performance report against SpringServe, synchronously for small pulls or asynchronously for large ones, then page or download the results — while staying inside the tightest rate limit in the API.
api: openapi/magnite-springserve-v1-openapi.yml
base_url: https://console.springserve.com
operations: [reports_post, reports_id_get, reports_download_id_get, reports_templates_get, reports_templates_post, reports_templates_id_get, scheduled_reports_get, scheduled_reports_post]
---

# Run a performance report

Authenticate and confirm the active account first — see
`magnite-authenticate-and-set-account.md`. A report always reflects the active account.

## Budget before you call

Reporting is the most rate-limited surface in the API: **10 requests/minute** to the reporting
endpoint, and only **3 requests/minute** when the report is dimensioned by domain or app bundle.
No header tells you how much budget is left. Decide the query before you start rather than
exploring interactively, and prefer one wide report over many narrow ones.

## Steps

1. **Start from a template if one fits.** `GET /api/v1/reports/templates`
   (`reports_templates_get`) lists saved report shapes; `reports_templates_id_get` reads one.
   Reusing a template is one request instead of several rounds of trial and error.
2. **Run the report.** `POST /api/v1/reports` (`reports_post`) with a JSON body of report
   parameters — date range, dimensions, metrics. The spec ships two worked bodies as
   `components.examples`: `report_create_simple` and `report_create_full`. Read them rather than
   guessing field names; the parameter schema lives in the external ref
   `openapi/ref/reports.yaml#/components/schemas/reportParameters`.
3. **Choose sync or async on the request body.** The body accepts `async: true` and
   `csv: true` booleans. Use `async: true` for anything wide or long-dated — a synchronous large
   report is the usual cause of a hung call.
4. **Fetch the rows.** `GET /api/v1/reports/{id}` (`reports_id_get`), which accepts `page`,
   `limit`, `sort` and `source` query parameters. Page size defaults to 50 objects; there is no
   total-count field and no `Link` header, so you know you are done only when a page comes back
   short or empty.
5. **Or take the whole file.** `GET /api/v1/reports/download/{id}`
   (`reports_download_id_get`), with `zipped` when the export is large. Report downloads are the
   only `text/csv` responses in the API. For a bulk pull this is one request against the
   10/minute budget instead of dozens of paged ones — prefer it.
6. **To make it recur**, create a scheduled report with `POST /api/v1/scheduled_reports`
   (`scheduled_reports_post`); list existing ones with `scheduled_reports_get`. This moves the
   work off your rate-limit budget entirely.

## Failure handling

- A `422` returns `{"status": 422, "error": "<summary>", "errors": {"<field>": ["..."]}}` — read
  `errors` to find the bad parameter; the spec also defines a `reportParameterError` schema.
- A `401` means the token or the active account changed under you. Re-run step 4 of the
  authentication skill.
- Because there is no idempotency key, a timed-out `reports_post` may still have created a
  report. List recent reports before firing a second one.

## The Python alternative

The first-party `springserve` library (PyPI, Apache 2.0, 0.8.14 released 2026-03-17) wraps this
flow, handles pagination automatically, and converts results to a Pandas dataframe via
`to_dataframe()` with the `reporting` extra installed. If the caller is running Python, that is
less code and fewer requests than driving the REST endpoints by hand.
