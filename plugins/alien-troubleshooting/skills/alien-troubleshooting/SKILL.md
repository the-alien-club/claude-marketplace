---
name: alien-troubleshooting
description: Diagnose and fix a failing Alien platform operation via the MCP — a workflow that fails, an agent that errors or ignores its tools, ingestion that never completes, permission denials, gateway timeouts, or invalid model errors. Use whenever an alien_* call or an agent run fails and the cause is not obvious.
---

# Alien: Troubleshooting
Match the symptom, confirm the cause with a read-only MCP call, apply the fix.
The full table is in [`reference/failure-catalog.md`](../../reference/failure-catalog.md);
this skill is the fast path for the failures you actually hit.

## When to use
An `alien_*` tool returned an error, a job came back `failed`, an agent answered
without using its data, ingestion is stuck, or a call was denied. Start here
before retrying blindly — most of these do not resolve on retry.

## First move, always
`alien_whoami` (`alien-getting-started`). Confirm you are the org you think you
are and that the token carries the abilities the failing call needs. A large
share of failures are permission gaps wearing a confusing error. Do NOT reach
for the admin token to "make it work" — an admin token masks 401/403s and hides
the real problem; diagnose with the normal token first.

## Symptom → cause → fix
**Agent run fails immediately with `worker_disconnected`.**
The agent graph is missing its session wiring — it was hand-built or edited into
an invalid shape, not produced by `alien_create_agent`. Fix: rebuild with
`alien_create_agent` (it clones the working preset), then re-attach tools with
the builder tools. Never assemble an agent graph by hand. See `alien-build-agent`.

**Agent answers from general knowledge / ignores its data.**
The MCP server or tools never attached, or attached to the wrong node. Confirm
with `alien_get_agent_definition` — check the root agent (or the intended
subagent) actually lists the expected tools. If missing, re-run
`alien_add_mcp_server` / `alien_add_tool` with the correct `target_node_id`. If
present but empty results, the underlying dataset was never embedded — re-check
`alien_apply_pipeline_preset` ran (see `alien-add-data`).

**A tool call is denied (401 / 403).**
Either the token lacks the ability (`workflow:write`, `dataset` abilities, etc.)
— confirm with `alien_check_my_abilities` — or, for an external-API source, the
connector is public but its endpoints are not, so non-org callers get 401. Fix
the second by marking endpoints public (`alien-rbac-and-pricing`), not by
switching to an admin token.

**`alien_run_agent` times out (~504 / gateway timeout).**
Expected for any agent that makes multiple tool calls — the synchronous
Responses path has a ~30s ceiling. This is not a real failure. Switch to the
async path: `alien_run_workflow` + `alien_get_workflow_run`. See
`alien-run-workflow`.

**`alien_create_agent` / `alien_add_subagent` rejects the model.**
The `model` must be a valid slug from the catalog, not a display name. Run
`alien_list_ai_models` and pass a `slug` from it. See `alien-ai-models`.

**Ingestion never completes.**
Poll `alien_get_ingestion_status`. If a document is stuck or errored, the fix is
per-document (re-add it), not waiting longer. Do not build or run an agent over
a dataset that has not finished ingesting — it returns partial, wrong answers
with no error. See `alien-add-data`.

**Creating a dataset fails against a new cluster.**
The cluster is still provisioning. Poll `alien_get_cluster_provisioning_status`
until ready before creating datasets in it.

## Reading a failed job
When `alien_get_workflow_run` shows `status: failed`, call it again with
`full: true`. The per-node `errors` (with tracebacks) pinpoint which node failed
and why — an unknown node type, a validation error on a locked param, an
unreachable MCP server. That node is where to fix, not the whole graph.

## Next
- Rebuild or re-wire an agent: `alien-build-agent`.
- Full failure table with tool-by-tool signatures:
  [`reference/failure-catalog.md`](../../reference/failure-catalog.md).
- Verify permissions and sharing: `alien-rbac-and-pricing`.
