---
name: Find CCM therapy clinics from the Impulse Dynamics provider directory
description: >-
  Read the clinic/provider directory behind impulse-dynamics.com/find-provider/ through the site's
  WordPress REST API, filtered by country or US state, without scraping the rendered page.
api: openapi/_original/impulse-dynamics-wp-rest-openapi.yml
operations:
  - getWpV2ClinicLocator
  - getWpV2ClinicLocatorId
  - getWpV2ClinicCountry
  - getWpV2StateName
generated: '2026-08-23'
method: generated
source: >-
  Grounded in openapi/_original/impulse-dynamics-wp-rest-openapi.yml, derived from
  https://impulse-dynamics.com/wp-json/ (fetched 2026-08-23, HTTP 200). Every operationId below was
  verified to exist verbatim in that spec.
---

# Find CCM therapy clinics

Impulse Dynamics publishes a directory of clinics and providers that offer Cardiac Contractility
Modulation (CCM) therapy with the Optimizer device. The human page is
`https://impulse-dynamics.com/find-provider/`. The same data is available as JSON from the site's
WordPress REST API, which is the surface you should use.

**Base URL:** `https://impulse-dynamics.com`

## Authentication

None required. These are read operations on published content and respond anonymously. Do not send
credentials.

## Steps

1. **Resolve the taxonomy first.** Call `getWpV2ClinicCountry`
   (`GET /wp-json/wp/v2/clinic_country`) and `getWpV2StateName`
   (`GET /wp-json/wp/v2/state_name`) to get the term IDs and slugs for countries and US states. You
   need the integer term ID to filter — the directory is classified by these two taxonomies, not by
   a free-text address field.

2. **List the clinics.** Call `getWpV2ClinicLocator` (`GET /wp-json/wp/v2/clinic-locator`).
   - Filter with `clinic_country=<termId>` and/or `state_name=<termId>`.
   - Page with `page` and `per_page`. `per_page` defaults to 10 and is **hard-capped at 100** — read
     `X-WP-Total` and `X-WP-TotalPages` from the response headers to know when you are done. The body
     is a bare JSON array; the counts are only in the headers.
   - Narrow the payload with `_fields=id,slug,title,link,clinic_country,state_name` rather than
     pulling full rendered HTML for every entry.

3. **Fetch one clinic.** Call `getWpV2ClinicLocatorId`
   (`GET /wp-json/wp/v2/clinic-locator/{id}`) for a single record.

## Conventions that apply

- **Pagination:** `page` / `per_page` (max 100) / `offset`. Totals in `X-WP-Total`,
  `X-WP-TotalPages`; `Link` header carries `rel="next"`.
- **Errors:** the WordPress envelope `{"code":…, "message":…, "data":{"status":…}}`, not
  RFC 9457 problem+json. **Branch on `code`, never on `message`** — message is human prose.
  See `errors/impulse-dynamics-problem-types.yml`.
- **No rate-limit headers.** This host returns no `RateLimit-*` or `Retry-After` header, so you get
  no budget signal. Throttle yourself conservatively — the origin sits behind a managed WordPress
  edge that may throttle without telling you.
- **No request-id.** There is no correlation header to quote if a call fails.

## Cautions

- This directory is marketing content maintained by the company, not a regulated registry. Do not
  present it as a certified or exhaustive list of CCM providers, and do not use it to make a clinical
  referral decision on a patient's behalf.
- The contract carries no versioning, deprecation or SLA commitment (see
  `lifecycle/impulse-dynamics-lifecycle.yml`). Routes can change whenever the site is updated.
- **Read only.** Write operations exist on this post type but require authentication you should not
  attempt to obtain.
