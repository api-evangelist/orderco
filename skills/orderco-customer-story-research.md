---
name: Research Order.co customer and vendor stories by industry
description: Pull Order.co's published case studies, filtered by its first-party industry and use-case taxonomies, through the public, anonymous Order.co Content API.
api: openapi/orderco-content-openapi.yml
operations:
  - listIndustry
  - listUseCase
  - listCustomerStory
  - getCustomerStory
  - listVendorStory
  - listTestimonials
  - listTypes
generated: '2026-08-04'
method: generated
source: openapi/orderco-content-openapi.yml
---

# Research Order.co customer and vendor stories by industry

Order.co registered its own WordPress taxonomies — `industry` and `use_case` — and tagged its
case studies with them. That turns a marketing page into a queryable dataset: "every Order.co
customer story in hospitality", answered in two calls, anonymously.

Base URL: `https://www.order.co/wp-json`

## Steps

1. **Get the vocabulary first.** Call `listIndustry` (`GET /wp/v2/industry?per_page=100`) and
   `listUseCase` (`GET /wp/v2/use_case?per_page=100`). Each term returns `id`, `slug`, `name`,
   `count` and `link`. There were 11 industry terms and 6 use-case terms at capture time. You need
   the numeric `id` for the next step — the filter takes ids, not slugs.

2. **Filter the stories.** Call `listCustomerStory`
   (`GET /wp/v2/customer_story?industry=<id>&per_page=100&_fields=id,slug,title,link,date,industry,use_case`).
   36 stories were published at capture time. `_fields` matters: an unfiltered record carries the
   full rendered HTML and runs to several kilobytes each.

3. **Read one in full.** Call `getCustomerStory` (`GET /wp/v2/customer_story/{id}`) once you have
   picked one, or add `_embed=1` to step 2 to inline the linked terms and featured media in a
   single round trip.

4. **Do the same on the supply side.** Call `listVendorStory`
   (`GET /wp/v2/vendor_story?industry=<id>`). Only 6 exist, and they carry `industry` but not
   `use_case`.

5. **Add the quotes.** Call `listTestimonials` (`GET /wp/v2/testimonials?per_page=100`) — 142
   records, untaxonomised, so filter client-side by matching company names against the stories
   you already have.

6. **Confirm the shape before you rely on it.** Call `listTypes` (`GET /wp/v2/types`) to re-read
   which post types exist and which taxonomies each supports. This is Order.co's own live
   declaration and it is the source `openapi/orderco-content-openapi.yml` was generated from — if
   it disagrees with this skill, believe it, not this file.

## Conventions and errors

- Anonymous `GET` only. Write routes return `401 rest_forbidden`.
- Pagination is `page` + `per_page` (max 100, enforced — `per_page=999` returns
  `400 rest_invalid_param`). Read `X-WP-Total` and `X-WP-TotalPages`, or follow the RFC 8288
  `Link: rel="next"` chain.
- Requesting a page past the end returns `400 rest_post_invalid_page_number`.
- An id that exists as one post type will not exist as another: `GET /wp/v2/customer_story/1`
  returns `404 rest_post_invalid_id`.
- Errors are the WordPress `{code, message, data.status}` object, not RFC 9457 problem+json. See
  `errors/orderco-problem-types.yml`.

## Do not use

`GET /cn/v1/resources` returns the entire corpus unpaginated — 8.9 MB at capture time — and
duplicates what the `wp/v2` routes serve with paging and field selection. `GET /cn/v1/press-releases`
returns rendered HTML inside a JSON string rather than records.
