---
name: alien-rbac-and-pricing
description: Control who can access Alien resources and set reporting-only pricing — coarse public/private visibility, fine-grained org/user sharing, dataset/endpoint pricing, and checking your own effective abilities. Use whenever the user wants to share, publish, restrict, price, or monetize a cluster, dataset, connector, workflow, or MCP config.
---

# Alien: RBAC & Pricing (Flow 5)
Two separate access mechanisms exist, and they don't automatically compose:
1. **Visibility** (`alien_set_visibility`) — the coarse public/private/org gate
   on a single resource. This is the mechanism that actually governs
   connector and MCP-config sharing today.
2. **Grant/Group RBAC** (`alien_share_with_org`, `alien_share_with_user`) —
   fine-grained grants. Enforcement currently applies ONLY to cluster-data
   and workflow access paths; grants on other resource types are accepted
   but not yet enforced.
Pricing (`alien_set_dataset_price`, `alien_set_endpoint_pricing`) is
REPORT-ONLY: it drives royalty/usage figures in `alien-analytics` — there is
no payout mechanism, setting a price never moves money.

## Tools
- `alien_set_visibility` — `resource_type` required (`dataset`, `connector`,
  `workflow`, `mcp_config`). Dispatches by type:
  - `dataset` / `connector` / `workflow`: also require `resource_id`
    (integer) and `is_public` (boolean).
  - `mcp_config`: instead requires `slug` (cfg_...) and `visibility`
    (`private` | `org` | `public`).
  Mixing the wrong pair (e.g. passing `slug` for a `dataset`) fails
  validation — the two identifier schemes are mutually exclusive by type.
- `alien_set_dataset_price` — `dataset_id`, `access_price` (number, >= 0)
  required. Sets the per-access price used to compute royalties
  (royalties = accesses × access price) in `alien_get_dashboard_metrics` and
  related analytics.
- `alien_set_endpoint_pricing` — `connector_id`, `endpoint_ids` (array),
  `pricing_mode`, `unit_price_cents` (integer, >= 0) all required; optional
  `unit_label`. Pricing-focused sibling of `alien_configure_endpoints` (see
  `alien-external-api-to-mcp`) — both write through the same batch-update
  endpoint, so don't set conflicting pricing in two places.
- `alien_share_with_org` — `resource_type` (`dataset` | `workflow`),
  `resource_id`, `org_slug` required; optional `access_type` (`read` default
  | `write`), `group_name`. For `resource_type: dataset` you MUST also pass
  `cluster_id` — dataset grants use the ability path `[cluster, dataset]`.
  Orchestrates create-grant → create-group → attach-grant-to-group →
  attach-org-to-group as four backend calls; it is best-effort — if a later
  step fails, earlier ones are not rolled back.
- `alien_share_with_user` — same shape as `alien_share_with_org` but takes
  `user_email` instead of `org_slug`. Same `cluster_id`-required-for-dataset
  rule, same best-effort orchestration caveat.
- `alien_check_my_abilities` — no arguments. Returns the platform-admin flag
  plus abilities held through the Grant/Group engine. Use this to confirm a
  share actually took effect, or before attempting an action you're unsure
  you have rights for.

## Gotchas
- Sharing a dataset without `cluster_id` fails validation immediately —
  dataset grants are always scoped through their owning cluster.
- Grant/Group sharing on a `connector` or `mcp_config` is not an option
  (`resource_type` for share tools is only `dataset` | `workflow`) — to
  control connector/MCP-config access, use `alien_set_visibility` instead.
- Because grant enforcement today is limited to cluster-data and workflow
  paths, sharing a dataset via `alien_share_with_org`/`_user` and then
  checking access on, say, a connector will not reflect that share — don't
  conflate the two mechanisms when debugging an access issue.
- Pricing tools never trigger billing; treat `access_price` and
  `unit_price_cents` purely as inputs to the analytics reports in
  `alien-analytics`.

## Next
- See the royalty/usage numbers these settings drive: `alien-analytics`.
- Publish or adjust the MCP config whose visibility this controls: `alien-publish-mcp`.
- Confirm identity/org before sharing on someone else's behalf: `alien-getting-started`.
