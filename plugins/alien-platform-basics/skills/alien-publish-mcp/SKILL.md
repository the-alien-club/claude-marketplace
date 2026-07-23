---
name: alien-publish-mcp
description: Publish a shareable Alien MCP configuration bundling cluster tools/datasets and/or external-API tools into one connectable MCP endpoint. Use whenever the user wants to publish, expose, or share an MCP config, or needs the connection URL for one that already exists.
---

# Alien: Publish MCP Config (Flow 3)
An MCP config bundles selected cluster tools (optionally scoped to specific
datasets) and external-API connector tools into a single connectable
streamable-HTTP MCP endpoint. This is how a data cluster from `alien-add-data`
or a connector from `alien-external-api-to-mcp` actually becomes something an
MCP client can attach to.

## Call sequence
1. `alien_list_available_sources` — no arguments. Lists every cluster
   (with its tools and datasets) and every external-API connector (with its
   tools) you may attach. Always call this first — you need real
   `cluster_id`/`dataset_ids`/`connector_id`/tool names for the next step, do
   not guess them.
2. `alien_create_mcp_config` — `name` and `config` required; optional
   `visibility` (`private` default, `org`, `public`) and `is_default` (serve
   this config when no `?config=` is specified). `config` uses snake_case
   keys exactly:
   ```
   {
     "clusters": [{"cluster_id": 1, "tools": ["..."], "dataset_ids": [1, 2]}],
     "external_apis": [{"connector_id": 1, "tools": ["..."]}]
   }
   ```
   `dataset_ids` is optional per cluster entry (omit to scope by tool only).
   Returns the created config including its slug (`cfg_...`). Note: `org`
   visibility requires an active organization on your token.
3. `alien_set_mcp_config_public` — `slug` required; optional `visibility`
   (defaults to `public`). Use this later to change visibility on a config
   you already created, instead of recreating it.
4. `alien_list_mcp_configs` — no arguments. Lists your visible configs with
   slug, name, visibility. Use this to recover a slug you've lost track of.
5. `alien_get_mcp_connection_info` — `slug` required. The key hand-off
   tool: returns the streamable-HTTP connection URL, the `Authorization`
   header to use, the resolved `mcp_`-prefixed tool filter, and a
   ready-to-paste client config example. Call this immediately after
   creating a config, or any time a builder needs to actually connect.

## Gotchas
- `alien_create_mcp_config`'s `config` object is snake_case
  (`cluster_id`, `dataset_ids`, `external_apis`) — this is different casing
  from many other tools' camelCase backend bodies; get the keys exactly
  right or the call is rejected.
- Tool names inside `config.clusters[].tools` / `config.external_apis[].tools`
  must be names discovered via `alien_list_available_sources`, not raw
  `alien_*` names guessed by pattern.
- `visibility: "org"` fails without an active organization on the token —
  confirm one exists via `alien_whoami` first (see `alien-getting-started`).
- The connection URL from `alien_get_mcp_connection_info` embeds the config
  slug as a query param (`?config=cfg_...`); it is not a generic
  "connect to Alien" URL — a client using it only sees the tools bundled in
  that specific config.

## Next
- Build and run an agentic workflow against the published config: see the
  `alien-workflow-engine` plugin's `alien-build-agent` and `alien-run-workflow`
  skills.
- Control who can use this config beyond the visibility flag, or price the
  underlying resources: `alien-rbac-and-pricing`.
- Track usage of the config's underlying clusters/connectors: `alien-analytics`.
- Deleting a config (cannot delete your last remaining one): `alien-cleanup`.
