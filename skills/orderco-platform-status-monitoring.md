---
name: Monitor Order.co platform status
description: Check whether the Order.co application is up, list reported components, read incident history and watch for scheduled maintenance through the public, anonymous Order.co Status API.
api: openapi/orderco-status-openapi.yml
operations:
  - getStatus
  - getSummary
  - listComponents
  - listUnresolvedIncidents
  - listIncidents
  - listUpcomingScheduledMaintenances
generated: '2026-08-04'
method: generated
source: openapi/orderco-status-openapi.yml
---

# Monitor Order.co platform status

Order.co runs an Atlassian Statuspage at `https://status.order.co/` (page id `ckzwmwkw4x3f`) and
leaves its v2 JSON API open. No key, no account, no header. This is the only operational API
Order.co publishes.

Base URL: `https://status.order.co/api/v2`

## Steps

1. **Cheapest liveness check.** Call `getStatus` (`GET /status.json`). The response is a few
   hundred bytes: `status.indicator` is one of `none`, `minor`, `major`, `critical`,
   `maintenance`, and `status.description` is the human phrasing ("All Systems Operational").
   Anything other than `none` warrants a second call.

2. **One call for everything.** Call `getSummary` (`GET /summary.json`) instead when you want the
   whole picture in a single request — page identity, every component, unresolved incidents,
   active and upcoming maintenance, and the rollup. Prefer this over fanning out.

3. **Respect the cache window.** Responses carry
   `Cache-Control: max-age=10, public, s-maxage=10, stale-while-revalidate=20` and a weak `ETag`.
   Polling faster than every ten seconds returns the same bytes. Send `If-None-Match` and handle
   `304`.

4. **Know what a component means here.** Call `listComponents` (`GET /components.json`). At the
   time this skill was generated the page had exactly **one** component,
   "Order.co Application (app.order.co)". There is no separate component for the API, for the
   accounting integrations, or for the vendor surface — so a green page does not tell you that a
   QuickBooks or NetSuite sync is healthy. Do not infer integration health from this API.

5. **Check for an active outage.** Call `listUnresolvedIncidents`
   (`GET /incidents/unresolved.json`). An empty `incidents` array means nothing is open.

6. **Read the history with a caveat.** Call `listIncidents` (`GET /incidents.json`) for the full
   record with `incident_updates` nested inside each incident, newest first. Be aware that as of
   2026-08-04 the only record on this page was the Statuspage default sample,
   "This is an example incident" (`id yn6q124l9xx6`). Treat an empty or sample-only history as
   *no signal*, not as *a clean record*.

7. **Look ahead.** Call `listUpcomingScheduledMaintenances`
   (`GET /scheduled-maintenances/upcoming.json`) before scheduling your own work. None had been
   published at capture time.

## Conventions and errors

- Every operation is a `GET`, unauthenticated, `application/json`.
- `access-control-allow-origin: *`, so this is callable directly from a browser.
- No pagination — each endpoint returns the whole collection.
- No rate-limit headers are returned. Stay inside the 10-second cache window anyway.
- Atlassian returns `atl-request-id` and `atl-traceid`; those identify a *Statuspage* request, so
  they are useful to Atlassian support, not to Order.co support.
- See `conventions/orderco-conventions.yml` and `lifecycle/orderco-lifecycle.yml`.

## Subscribing instead of polling

The page offers email, Slack and Microsoft Teams subscriptions plus
`https://status.order.co/history.atom` and `https://status.order.co/history.rss`. Unlike most
Statuspage instances it does **not** expose the generic `POST /subscriptions/webhook.json`
option, so an agent that wants push has to go through Teams or consume the Atom feed.

## Out of scope

This API says nothing about the Order.co procurement, accounts-payable, spend-management or
vendor APIs. Those exist — Order.co markets API connectivity to QuickBooks Online, NetSuite,
Sage Intacct and Workday, and issues vendor API/EDI credentials at onboarding — but none of it is
publicly documented, so there is no operation to ground a skill in.
