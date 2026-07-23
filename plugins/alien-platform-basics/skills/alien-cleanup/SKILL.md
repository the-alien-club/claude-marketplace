---
name: alien-cleanup
description: Permanently delete Alien resources — workflows, connectors, MCP configs, datasets, and clusters — in a cascade-safe order. Use whenever the user wants to delete, tear down, decommission, or remove a workflow, agent, connector, dataset, or cluster. All of these are destructive and irreversible.
---

# Alien: Cleanup (Lifecycle Deletes)
Every tool below is destructive and permanent — there is no undo, and several
cascade to child resources. Confirm the target id with a list/get call
(`alien_list_clusters`, `alien_list_available_sources`, `alien_list_mcp_configs`)
before deleting anything; never guess an id.

## Delete order
When tearing down a whole setup, delete top-down through what depends on what,
not bottom-up — an MCP config or workflow left pointing at an already-deleted
cluster/connector is a dangling reference a client may still be using:
1. `alien_delete_workflow` — `workflow_id` required. Removes the workflow/agent
   and its job associations. Delete workflows before the clusters or connectors
   they consume, so nothing keeps running against infrastructure you're about
   to remove. Requires ownership and a write-capable token.
2. `alien_delete_mcp_config` — `slug` (cfg_...) required. Removes the
   published config; any client connecting via its URL stops working
   immediately. You cannot delete your last remaining config. Do this before
   deleting the connectors/clusters it bundles, so the config doesn't
   silently start erroring instead of cleanly disappearing.
3. `alien_delete_connector` — `connector_id` required. Cascades to the
   connector's imported endpoints and any generated MCP tool definitions.
   Requires org-admin role on the owning organization.
4. `alien_delete_dataset` — `dataset_id` required. Cascades to the dataset's
   entries (documents) and their processed content/embeddings. Use this when
   removing one dataset but keeping its cluster; skip it if you're about to
   delete the whole cluster anyway (step 5 already cascades to it).
5. `alien_delete_cluster` — `cluster_id` required. HIGHLY destructive:
   decommissions the cluster's tenant infrastructure and cascades to EVERY
   dataset, entry, and stored document it holds. Requires org-admin role.
   This is the last step, and the only one that removes real provisioned
   infrastructure (databases, object storage, vector store, workflow engine).

## Gotchas
- `alien_delete_cluster` cascades to all datasets in it — do not bother
  calling `alien_delete_dataset` for every dataset first unless you actually
  want to keep the cluster and only remove some of its datasets.
- `alien_delete_mcp_config` refuses to delete your last remaining config —
  if that happens, either accept the config stays or create a replacement
  first.
- `alien_delete_connector` and `alien_delete_cluster` both require org-admin
  role, which is stricter than the write-capable role needed for most create
  calls — confirm abilities with `alien_check_my_abilities` (see
  `alien-rbac-and-pricing`) before assuming the delete will succeed.
- None of these tools ask for confirmation twice — there is exactly one call
  between "id in hand" and "gone forever." Verify the id belongs to the
  resource you actually intend to remove, not a similarly-named one.

## Next
- Confirm the id and ownership before deleting: `alien-getting-started`,
  `alien-rbac-and-pricing`.
- Rebuilding after cleanup: `alien-add-data`, `alien-external-api-to-mcp`,
  `alien-publish-mcp`.
