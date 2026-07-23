---
name: alien-analytics
description: Pull usage and consumption analytics for your Alien organization — dashboard totals, top consumers, top datasets, workflow compute spend, and external-API call-log usage. Use whenever the user asks about usage, consumption, royalties, top consumers/datasets, or how much a workflow/connector is costing or earning.
---

# Alien: Analytics (Flow 6)
All analytics here are org-scoped (your organization's data, from
`alien_whoami`) and any royalty/price figures are REPORT-ONLY — computed from
access counts × price, with no actual payout mechanism (see
`alien-rbac-and-pricing` for where those prices are set).

## Tools
- `alien_get_dashboard_metrics` — optional `period` (`live` | `daily` |
  `weekly` | `monthly`, default `monthly`, case-sensitive), `start`/`end`
  (ISO 8601). Returns total royalties, total consumers, and a time series.
  **`start`/`end` are validated but NOT honored** — each `period` returns a
  fixed trailing window ending now (`live` ~ last hour, `daily` ~ last 2
  days, `weekly` ~ last 2 weeks, `monthly` ~ last 2 months); an arbitrary
  historical range cannot be queried yet.
- `alien_get_top_consumers` — same `period`/`start`/`end` shape. Leaderboard
  of the top consumers of your organization's data. `start`/`end` behavior
  should be assumed to have the same fixed-window limitation as
  `alien_get_dashboard_metrics` unless verified otherwise for this endpoint.
- `alien_get_top_datasets` — same shape. Leaderboard of your organization's
  most-consumed datasets.
- `alien_get_workflow_consumption` — same shape. This is COMPUTE cost from
  running workflows/agents — a distinct figure from dataset royalties above,
  do not conflate the two when reporting numbers to a user.
- `alien_get_external_api_usage` — optional `period` (same enum/default) and
  `group_by` (`CONNECTOR` default | `ENDPOINT` | `TIME` | `STATUS` |
  `SOURCE_TYPE`). Goes through the backend's GraphQL endpoint internally;
  returns per-group request counts, distinct consumers, avg/p95 response
  time, billed units, and total price in cents. Prices here are also
  report-only.

## Gotchas
- `period` is case-sensitive lowercase (`monthly`, not `Monthly`) across the
  REST-backed tools; `alien_get_external_api_usage` accepts the same
  lowercase input but maps it to an uppercase GraphQL enum internally — you
  never need to pass uppercase yourself.
- Don't ask for a custom historical window via `start`/`end` on
  `alien_get_dashboard_metrics` expecting it to be honored — it currently
  is not; if a user needs a specific historical range, say so plainly rather
  than presenting the fixed-window result as if it were that range.
- Workflow compute spend (`alien_get_workflow_consumption`) and dataset
  royalties (`alien_get_dashboard_metrics`) are unrelated figures from
  different subsystems — never add them together.

## Next
- Where the prices behind these royalty figures are set: `alien-rbac-and-pricing`.
- Confirm which organization these numbers belong to: `alien-getting-started`.
