---
name: recipe-publish-and-share-a-source
description: End-to-end recipe to turn data or an API you already have on the Alien platform into a shareable, optionally priced MCP source for your organization or a specific user. Use when the goal is to distribute or monetize an existing dataset or connector, not to build an agent.
---

# Recipe: Publish and Share a Source
Goal: take something that already exists on the platform (an ingested dataset or
an API connector with generated tools) and make it a reusable MCP source others
can consume — optionally public, shared with your org, or priced.

## When to use
"Let my team use this dataset", "publish this API as a source", "put a price on
this data", "make this MCP available to another org member". This assumes the
data/connector already exists (see `recipe-rag-agent-over-documents` steps 1-2
or `recipe-api-to-agent` steps 1-4 to create one first).

## Steps
1. **Confirm what you can share** (`alien-getting-started`,
   `alien-rbac-and-pricing`). `alien_check_my_abilities` — verify you can set
   visibility and pricing on the target. You can only share what you own or have
   admin rights over.
2. **Find the source** (`alien-publish-mcp`). `alien_list_available_sources` —
   locate the dataset or connector you intend to publish, by id.
3. **Publish an MCP config** (`alien-publish-mcp`).
   - `alien_create_mcp_config` selecting the source(s) → keep the config id.
   - `alien_list_mcp_configs` → confirm it was created as expected.
4. **Set access** (`alien-rbac-and-pricing`).
   - `alien_set_visibility` on the dataset/connector as needed.
   - `alien_set_mcp_config_public` to make the MCP config itself publicly
     reachable, if that's the intent.
   - `alien_share_with_org` or `alien_share_with_user` for scoped sharing.
   - Note the two-gate rule for external-API sources: a public connector still
     needs its endpoints marked public, or non-org-members get 401s.
5. **Price it** (optional, `alien-rbac-and-pricing`).
   - `alien_set_dataset_price` for a dataset, or `alien_set_endpoint_pricing`
     for API endpoints.
6. **Hand out the connection** (`alien-publish-mcp`).
   - `alien_get_mcp_connection_info` → the MCP server `url` and auth model that
     a consumer plugs into their own MCP client or attaches to an agent via
     `alien_add_mcp_server`.

## Gotchas
- Visibility is layered: a public MCP config over private endpoints still fails
  for outsiders. Match visibility at both the config and the underlying
  source/endpoint level.
- Pricing applies to the underlying dataset/endpoints, not the MCP config
  wrapper — set it on the source.
- Sharing with a user requires that user to be resolvable in your org context.

## Next
- Watch consumption and royalties on the shared source: `alien-analytics`.
- Build your own agent on top of the published source: `recipe-api-to-agent` or
  `recipe-rag-agent-over-documents`.
