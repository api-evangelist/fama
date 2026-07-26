---
name: Run a Fama screening check and retrieve the report
description: >-
  Submit a social-media screening check for a candidate, wait for the report to
  reach DONE status (via completion callback or polling), then pull the summary,
  person, findings, and signed PDF.
api: https://developer.fama.io/reference/fama-rest-api
base_url: https://public-api.fama.io
auth: "Bearer token (Authorization: Bearer <token>)"
operations:
  - _get_me_me_get
  - _get_my_companies_my_companies_get
  - _get_my_check_types_my_check_types_get
  - _request_check_company__company_uuid__check_post
  - _check_status_company__company_uuid__check_get
  - _get_summary_report__report_uuid__summary_get
  - _get_person_report__report_uuid__person_get
  - _download_report_pdf_report__report_uuid__pdf_get
---

# Run a Fama screening check and retrieve the report

Use the Fama v2 REST API at `https://public-api.fama.io`. Authenticate every
request with `Authorization: Bearer <token>`. The API is rate limited to
**100 requests/minute per token** — back off on HTTP 429.

## Steps

1. **Validate the token** — `GET /me` (`_get_me_me_get`). A 200 confirms the
   bearer token is valid and tied to an active user. A 401 means refresh/obtain
   a token first.
2. **Resolve the company** — `GET /my/companies` (`_get_my_companies_my_companies_get`)
   to get the `company_uuid` you are authorized to run checks for.
3. **Pick a check type** — `GET /my/check_types` (`_get_my_check_types_my_check_types_get`)
   to choose the appropriate `check_type_uuid` (report type, lookback, sources).
4. **Submit the check** — `POST /company/{company_uuid}/check`
   (`_request_check_company__company_uuid__check_post`) with the candidate's
   required fields and the chosen check type. Capture the returned `report_uuid`.
5. **Wait for completion** — prefer the completion callback (see
   `../asyncapi/fama-webhooks.yml`): Fama POSTs `{report_uuid, full_name, email}`
   to your configured endpoint when the report reaches **DONE**. If callbacks are
   not configured, **poll** `GET /company/{company_uuid}/check`
   (`_check_status_company__company_uuid__check_get`) until the report status is
   `DONE`. There is no idempotency key — reconcile by `report_uuid`, do not blindly
   re-submit.
6. **Retrieve results** — once DONE:
   - `GET /report/{report_uuid}/summary` (`_get_summary_report__report_uuid__summary_get`) for aggregate counts.
   - `GET /report/{report_uuid}/person` (`_get_person_report__report_uuid__person_get`) for subject identity details.
   - `GET /report/{report_uuid}/pdf` (`_download_report_pdf_report__report_uuid__pdf_get`) for the signed PDF (redirect); download asynchronously, do not block your callback handler.

## Error handling

Errors use a custom JSON envelope (`message` / `detail`), not problem+json — see
`../errors/fama-error-codes.yml`. Handle 401 (refresh token), 403 (permissions),
422 (fix invalid fields via the `detail[]` array), and 429 (exponential backoff).
