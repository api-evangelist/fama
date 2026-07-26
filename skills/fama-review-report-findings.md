---
name: Review Fama report findings
description: >-
  For a completed Fama report, page through the social profiles, posts, and web
  content findings and read the aggregate summary for adjudication.
api: https://developer.fama.io/reference/fama-rest-api
base_url: https://public-api.fama.io
auth: "Bearer token (Authorization: Bearer <token>)"
operations:
  - _get_summary_report__report_uuid__summary_get
  - _get_profiles_report__report_uuid__profiles_get
  - _get_posts_report__report_uuid__posts_get
  - _get_web_content_report__report_uuid__web_content_get
  - _get_person_report__report_uuid__person_get
---

# Review Fama report findings

Once a report is `DONE`, use these v2 read endpoints against
`https://public-api.fama.io` with `Authorization: Bearer <token>`. Collection
endpoints accept pagination parameters — page until exhausted (see
`../conventions/fama-conventions.yml`). Respect the 100 req/min limit.

## Steps

1. **Read the summary** — `GET /report/{report_uuid}/summary`
   (`_get_summary_report__report_uuid__summary_get`) for totals: posts, web
   content, adverse findings, community findings, keywords, and overall results.
2. **Confirm the subject** — `GET /report/{report_uuid}/person`
   (`_get_person_report__report_uuid__person_get`) for names, emails, addresses,
   jobs, degrees, identifications, and phone numbers.
3. **List social profiles** — `GET /report/{report_uuid}/profiles`
   (`_get_profiles_report__report_uuid__profiles_get`), paging through profile URLs.
4. **Review flagged posts** — `GET /report/{report_uuid}/posts`
   (`_get_posts_report__report_uuid__posts_get`); each post carries `url`, `text`,
   `type`, `keywords`, and matched `behaviors`.
5. **Review web content** — `GET /report/{report_uuid}/web_content`
   (`_get_web_content_report__report_uuid__web_content_get`); each item carries
   `url`, `title`, `content`, and matched `behaviors`.

## Adjudication notes

Fama filters protected-class information for EEOC/FCRA-aligned adjudication —
base decisions on the returned `behaviors` categories, not raw content. Errors
follow `../errors/fama-error-codes.yml`.
