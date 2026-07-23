---
name: alien-getting-started
description: Orient on the Alien platform before doing anything else — confirm identity, organization, and write access, and discover existing clusters. Use this FIRST in any session that will call alien_* tools, before creating clusters, connectors, MCP configs, or workflows.
---

# Alien: Getting Started
Every `alien_*` tool call is a per-request token relay: the MCP server forwards
the caller's own `oat_...` token (or OAuth JWT) straight to the Alien backend.
There is no separate service identity — you act as whichever user/org the
token belongs to, with exactly that user's abilities. A read-only ("api-key")
scoped token cannot create clusters, connectors, or workflows; it will fail
with a permission error on any write call.

## When to use
Invoke this skill at the start of any task that touches the Alien platform,
and before any destructive or write-heavy flow (`alien-add-data`,
`alien-external-api-to-mcp`, `alien-publish-mcp`, `alien-cleanup`). Never
assume an organization, cluster, or write permission — verify it first.

## Call sequence
1. `alien_whoami` — no arguments. Returns the caller's identity, current
   organization, and platform abilities (what the token can read/write).
   Confirm the organization is the one you expect, and that write abilities
   are present before attempting any create/update/delete call downstream.
2. `alien_list_clusters` — no arguments. Lists the data clusters visible to
   the caller's organization, with status. Use this to find an existing
   cluster id before creating datasets or documents, or to confirm a
   newly-created cluster is provisioning.
3. `alien_get_cluster` — `cluster_id` (integer, required). Fetch one
   cluster's full detail and status by id. Use after `alien_list_clusters`
   once you have a specific id, to check readiness before creating datasets
   in it.

## Gotchas
- `alien_whoami` and `alien_check_my_abilities` (see `alien-rbac-and-pricing`)
  hit the same backend endpoint (`/users/me/abilities`) — either works for a
  quick permission check, but `alien_whoami` is the conventional first call.
- A cluster listed by `alien_list_clusters` may still be provisioning. Don't
  create datasets against a cluster until its status is ready — see
  `alien-add-data` for the provisioning-status poll.
- There is no "create organization" tool here; the organization is fixed by
  the token. If the wrong organization shows up in `alien_whoami`, the fix is
  a different token, not a platform call.

## Next
- Adding documents to (or provisioning) a cluster: `alien-add-data`.
- Wrapping an external REST API as MCP tools: `alien-external-api-to-mcp`.
- Publishing a shareable MCP config from what already exists: `alien-publish-mcp`.
- Checking what you're allowed to do or share: `alien-rbac-and-pricing`.
- Usage numbers for existing clusters/connectors: `alien-analytics`.
- Building an agent once you have data or an MCP config: see the
  `alien-workflow-engine` plugin's `alien-build-agent` skill.
