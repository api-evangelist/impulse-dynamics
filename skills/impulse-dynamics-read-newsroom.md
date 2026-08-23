---
name: Read the Impulse Dynamics newsroom and press coverage
description: >-
  Pull company announcements and third-party media coverage from impulse-dynamics.com as structured
  JSON via the site's WordPress REST API, instead of scraping the newsroom page or parsing RSS.
api: openapi/_original/impulse-dynamics-wp-rest-openapi.yml
operations:
  - getWpV2News
  - getWpV2NewsId
  - getWpV2InTheMedia
  - getWpV2Media
  - getWpV2Search
generated: '2026-08-23'
method: generated
source: >-
  Grounded in openapi/_original/impulse-dynamics-wp-rest-openapi.yml, derived from
  https://impulse-dynamics.com/wp-json/ (fetched 2026-08-23, HTTP 200). Every operationId below was
  verified to exist verbatim in that spec.
---

# Read the Impulse Dynamics newsroom

The company keeps two distinct content streams, and they are different things — do not merge them:

| Collection | operationId | What it is |
|---|---|---|
| `news` | `getWpV2News` | First-party company announcements (mirrored at `news.impulse-dynamics.com`) |
| `in_the_media` | `getWpV2InTheMedia` | Third-party press coverage **about** the company or CCM therapy |

Attributing a third-party article to the company as its own statement is the main failure mode here.

**Base URL:** `https://impulse-dynamics.com`

## Authentication

None. Published content responds anonymously.

## Steps

1. **List announcements.** `getWpV2News` (`GET /wp-json/wp/v2/news`).
   - Sort newest-first with `orderby=date&order=desc`.
   - Window with `after=<ISO8601>` / `before=<ISO8601>` for incremental pulls.
   - Trim with `_fields=id,date,slug,title,link,excerpt`.
2. **Fetch one item.** `getWpV2NewsId` (`GET /wp-json/wp/v2/news/{id}`).
3. **List press coverage.** `getWpV2InTheMedia` (`GET /wp-json/wp/v2/in_the_media`), same parameters.
4. **Resolve images/PDFs.** `getWpV2Media` (`GET /wp-json/wp/v2/media`) for any `featured_media` id,
   or add `_embed` to the list call to inline them in one round trip.
5. **Keyword search across types.** `getWpV2Search` (`GET /wp-json/wp/v2/search`) with `search=<term>`
   when you do not know which collection holds the item.

## Conventions that apply

- **Pagination:** `page` / `per_page` (default 10, max 100); totals in `X-WP-Total` and
  `X-WP-TotalPages` response headers.
- **Rendered HTML:** `title`, `excerpt` and `content` come back as `{"rendered": "<html>"}`. Strip
  the HTML before summarising; do not treat it as plain text.
- **Errors:** WordPress envelope, branch on `code`. See `errors/impulse-dynamics-problem-types.yml`.
- **Incremental sync:** there is no cursor and no change feed. Use `after=<last-seen-date>` with
  `orderby=date`; there is no reliable way to detect edits to already-seen items.

## Cautions

- Content is regulated medical-device marketing. When summarising, preserve indication and safety
  qualifiers; do not restate efficacy claims without the context the source gives them, and do not
  turn a press release into clinical advice.
- No versioning or deprecation commitment applies to this surface.
- **Read only.** Do not attempt writes.
