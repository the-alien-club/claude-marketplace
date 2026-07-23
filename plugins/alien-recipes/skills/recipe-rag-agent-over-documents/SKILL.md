---
name: recipe-rag-agent-over-documents
description: End-to-end recipe to build an Alien agent that answers questions over a set of documents — ingest the documents, publish them as an MCP config, build an agent wired to it, and run it. Use when the goal is "make an agent that knows about these documents/PDFs/files" rather than a single tool call.
---

# Recipe: RAG Agent over Documents
Goal: hand the platform some documents and end with a running agent that answers
questions grounded in them. This chains four flows. Each step names the atomic
skill to consult for parameter detail.

## When to use
"Build a chatbot over these PDFs", "make an agent that knows my docs", "RAG over
this corpus". If you only need to *ingest* documents (no agent), stop after
step 2 and see `alien-add-data`.

## Steps
1. **Orient** (`alien-getting-started`). `alien_whoami` — confirm the org and
   that the token has write abilities. `alien_list_clusters` — reuse an existing
   ready cluster if there is one; only create a cluster if none fits.
2. **Ingest the documents** (`alien-add-data`).
   - `alien_create_cluster` only if you need a new one, then poll
     `alien_get_cluster_provisioning_status` until ready.
   - `alien_create_dataset` in the cluster → keep the `dataset_id`.
   - `alien_apply_pipeline_preset` on the dataset (chunk + embed, so vector
     search works).
   - `alien_add_document` per document → each returns an ingestion handle.
   - `alien_get_ingestion_status` — poll until every document is processed.
     Do not proceed while anything is still ingesting; an agent over half-indexed
     data gives wrong answers silently.
3. **Publish the dataset as an MCP config** (`alien-publish-mcp`).
   - `alien_list_available_sources` — confirm the dataset shows up as a source.
   - `alien_create_mcp_config` selecting that dataset → keep the config id.
   - `alien_get_mcp_connection_info` → keep the MCP server `url`.
4. **Build the agent** (`alien-build-agent`, `alien-ai-models`).
   - `alien_list_ai_models` → pick a model `slug`.
   - `alien_create_agent` with `name`, `model`, and a `system_prompt` that says
     it answers strictly from the connected data → keep the `workflow_id`.
   - `alien_add_mcp_server` with `workflow_id` and the `url` from step 3. No auth
     token needed — it is an Alien-hosted MCP, auto-injected.
   - Optional: `alien_add_subagent` scoped to retrieval, then `alien_add_tool` /
     `alien_add_mcp_server` with `target_node_id = <subagent_node_id>` so the
     coordinator delegates search (mirrors the seeded Deep Agent preset).
5. **Run and verify** (`alien-run-workflow`).
   - `alien_run_agent` once with a question you know the answer to — a quick
     liveness check (accept that a multi-tool run may hit the ~30s timeout).
   - For real use: `alien_run_workflow` (async) + `alien_get_workflow_run`
     (poll). The answer lands at
     `result.results.<http_response id>[0].results.content`.
   - Hand a real client the wiring with `alien_get_agent_connection_info`.

## Gotchas
- Skipping `alien_apply_pipeline_preset` means no embeddings, so the agent's
  vector search returns nothing — the agent will look "dumb" for no obvious
  reason.
- If the agent answers from general knowledge instead of the data, tighten the
  `system_prompt` and confirm the MCP server actually attached via
  `alien_get_agent_definition`.
- A fresh agent that fails with `worker_disconnected` was not built by
  `alien_create_agent` — see `alien-troubleshooting`.

## Next
- Share or price the agent/source: `alien-rbac-and-pricing`.
- Improve retrieval quality if the source is an API, not documents:
  `recipe-api-to-agent` + the `alien-mcp-tool-refinement` plugin.
