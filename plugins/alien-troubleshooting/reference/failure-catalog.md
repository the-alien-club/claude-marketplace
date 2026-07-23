# Alien Failure Catalog (reference)
Every common failure mode for an external builder on the Alien platform, its
signature, the MCP tool that confirms it, and the fix. Ordered roughly by how
often it bites. The `alien-troubleshooting` skill is the fast path; this is the
complete table.

## Identity & permissions
| Symptom | Cause | Confirm with | Fix |
|---|---|---|---|
| 401/403 on a write call | Token lacks the ability (`workflow:write`, `dataset` abilities, …) | `alien_check_my_abilities` | Use a token with the ability; do not switch to the admin token to mask it |
| Wrong org's data / empty listings | Token belongs to a different org; org is fixed by the token | `alien_whoami` | Use the correct token — there is no "switch org" tool |
| External-API source 401s for teammates | Connector is public but its endpoints are not | `alien_check_my_abilities`, `alien_list_endpoints` | Mark endpoints public via `configure_endpoints` / `alien-rbac-and-pricing` |
| Everything "works" only with the admin token | Admin token bypasses policy and hides real 401/403s | switch back to a normal token | Diagnose with the normal token; fix the actual ability/visibility gap |

## Agent build & run
| Symptom | Cause | Confirm with | Fix |
|---|---|---|---|
| `worker_disconnected` on first run | Agent graph missing session wiring (hand-built or badly edited) | (known signature) | Rebuild with `alien_create_agent`, re-attach via builder tools |
| Agent ignores its data / answers generically | MCP server or tools not attached, or on the wrong node | `alien_get_agent_definition` | Re-run `alien_add_mcp_server`/`alien_add_tool` with correct `target_node_id` |
| Agent has fewer tools than expected | An `mcp_server` was unreachable at compile time and skipped (logged, not fatal) | `alien_get_agent_definition` | Fix the MCP URL/availability, re-attach |
| Subagent never invoked | Vague/duplicate subagent `description`; coordinator can't route | `alien_get_agent_definition` | Give each subagent a distinct, specific `description` |
| `alien_run_agent` returns 504 / times out | ~30s synchronous ceiling on the Responses path; multi-tool run exceeds it | (expected) | Use `alien_run_workflow` + `alien_get_workflow_run` (async) |
| Model rejected on create/update | Passed a display name, not a slug | `alien_list_ai_models` | Pass a `slug` from the catalog |
| Job `status: failed` | A specific node errored | `alien_get_workflow_run` with `full: true` → per-node `errors`/traceback | Fix the named node (validation, unreachable dep, unknown type) |

## Data & ingestion
| Symptom | Cause | Confirm with | Fix |
|---|---|---|---|
| Agent/search returns nothing | Dataset never embedded (pipeline preset not applied) | check `alien_apply_pipeline_preset` was run | Apply the pipeline preset, wait for re-ingestion |
| Ingestion never finishes | A document is stuck or errored | `alien_get_ingestion_status` | Re-add the affected document; don't just wait |
| Answers are partial/wrong, no error | Agent built over a dataset still ingesting | `alien_get_ingestion_status` | Wait until all documents are processed before running |
| Creating a dataset fails on a new cluster | Cluster still provisioning | `alien_get_cluster_provisioning_status` | Poll until ready, then create datasets |

## External API → tools
| Symptom | Cause | Confirm with | Fix |
|---|---|---|---|
| Generated tool always fails at agent runtime | Tools generated from an endpoint that 401s / returns nothing | `alien_test_endpoint` before generating | Fix the endpoint/auth, re-test, regenerate |
| LLM calls the wrong tool or skips it | Weak generated tool names/descriptions | inspect via `alien_list_endpoints` | Improve descriptions (`configure_endpoints`) or use the `alien-mcp-tool-refinement` plugin |
| Saved MCP config loses a tool after a rename | Live MCP filters tools by name string; renaming orphans configs | (known gotcha) | Refine via the safe path in `alien-mcp-tool-refinement`, not a raw name PATCH |

## Publishing & sharing
| Symptom | Cause | Confirm with | Fix |
|---|---|---|---|
| Public MCP config, outsiders still 401 | Underlying endpoints/dataset still private | `alien_list_endpoints`, `alien_list_available_sources` | Match visibility at both config and source level |
| Consumer can't reach the MCP | Wrong or stale connection info | `alien_get_mcp_connection_info` | Re-fetch and hand out the current `url` |

## Notes
- Most of these do not resolve on retry — retrying a `worker_disconnected`, a
  permission denial, or a stuck ingestion just wastes time. Diagnose first.
- When a job fails, `full: true` on `alien_get_workflow_run` is the single most
  useful call — it names the failing node and carries the traceback.
