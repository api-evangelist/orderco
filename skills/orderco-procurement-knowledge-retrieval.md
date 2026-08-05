---
name: Retrieve Order.co procurement knowledge
description: Search and pull Order.co's published procurement, AP-automation and spend-management corpus — 873 FAQ records, 271 blog posts, ebooks, webinars and tools — as structured JSON through the public, anonymous Order.co Content API.
api: openapi/orderco-content-openapi.yml
operations:
  - search
  - listFaqs
  - getFaqs
  - listPosts
  - getPosts
  - listEbook
  - listWebinar
  - listTool
  - listCategories
generated: '2026-08-04'
method: generated
source: openapi/orderco-content-openapi.yml
---

# Retrieve Order.co procurement knowledge

Order.co's `llms.txt` points models at HTML pages. The same corpus is available as JSON, and
there is more of it: 873 discrete FAQ records that never appear as standalone pages at all. If
you are building retrieval over what Order.co says about procurement, this API is the better
source than crawling the site.

Base URL: `https://www.order.co/wp-json`

## Steps

1. **Start with search when you have a question.** Call `search`
   (`GET /wp/v2/search?search=<term>&per_page=100`). It spans every public type and returns
   `id`, `title`, `url`, `type` and `subtype` — 306 hits for `procurement` at capture time. Read
   `X-WP-Total` to size the result set before paging.

2. **Go to the FAQ corpus for direct answers.** Call `listFaqs`
   (`GET /wp/v2/faqs?search=<term>&per_page=100&_fields=id,slug,title,link,content`). 873 records.
   This is the densest structured content on the site — each record is a question and its answer,
   already atomised, which is exactly the shape a retrieval index wants. Page through with
   `page`; `X-WP-TotalPages` tells you how far.

3. **Pull long-form for context.** Call `listPosts`
   (`GET /wp/v2/posts?search=<term>&per_page=20&_fields=id,slug,title,link,date,excerpt,categories`)
   — 271 articles. Always pass `_fields`; an unfiltered post record is roughly 70 KB because
   `content.rendered` carries the full HTML. Fetch the body with `getPosts`
   (`GET /wp/v2/posts/{id}`) only for the ones you actually need.

4. **Slice by topic.** Call `listCategories` (`GET /wp/v2/categories?per_page=100`) for the 10
   editorial categories (accounts payable, procurement, spend management, purchasing process,
   finance and so on), then filter with `GET /wp/v2/posts?categories=<id>`.

5. **Add the gated assets' metadata.** Call `listEbook` (21), `listWebinar` (10) and `listTool`
   (18). These return titles, slugs and links; the assets themselves sit behind forms on the
   site, so treat these as a catalogue, not as content.

6. **Date-bound an incremental crawl.** Every collection accepts `after`, `before`,
   `modified_after` and `modified_before` as ISO 8601 date-times, plus `orderby=modified`. Store
   the newest `modified` you have seen and pass it as `modified_after` on the next run rather
   than re-reading the corpus.

## Conventions and errors

- Anonymous `GET` only; no key, no header.
- `per_page` is bounded 1–100 server-side. `per_page=999` returns `400 rest_invalid_param` with
  the exact bound in `data.params`.
- `orderby` is an enum — `author, date, id, include, modified, parent, relevance, slug,
  include_slugs, title`. An invalid value returns `400 rest_invalid_param` with detail code
  `rest_not_in_enum`.
- Responses are not edge-cached (`cf-cache-status: DYNAMIC`) and carry no rate-limit headers.
  Cloudflare fronts the origin, so throttle yourself: unsignalled limits are still limits.
- Full error catalogue: `errors/orderco-problem-types.yml`. Cross-cutting semantics:
  `conventions/orderco-conventions.yml`.

## Scope

This is Order.co's published knowledge, not Order.co's product. Nothing here touches purchase
orders, vendors, invoices, virtual cards or spend data — that API exists but is undocumented and
credentialed, so there is no operation to ground a skill in.
