---
name: alien-workflow-nodes
description: Discover the nodes/tools available to attach to an Alien agent and what each one does — data-access, data-manipulation, documents/AI, artifacts, audio, and the agent-structural nodes. Use when choosing a tool_type for alien_add_tool, deciding how to give an agent a capability, or explaining what the platform's nodes can do.
---

# Alien: Workflow Nodes & Tools
Nodes are the building blocks of every Alien workflow, and most of them can be
attached to an agent as a callable tool. This skill is the catalog: what exists,
what each node consumes and produces, and how to attach it. For the exhaustive
per-node parameter list, read the bundled reference file
[`reference/node-catalog.md`](../../reference/node-catalog.md).

## When to use
When you're about to call `alien_add_tool` and need the right `tool_type`, when
someone asks "can the agent read a PDF / search my data / generate a voice /
write a spreadsheet", or when you need to explain the node system.

## Always list before you attach
The live source of truth is `alien_list_agent_tools` (optional `category`
filter). It returns each attachable node's `tool_type`, `name`, `description`,
`categories`, and `params` (name, type, description, default, required) —
exactly what the web editor palette shows. Boundary and container nodes
(`http_request`, `http_response`, `agent_input`, `agent_output`, `deep_agent`,
`ai_agent`, `group`) are excluded because they are not attachable. Do not guess
a `tool_type`; read it from this tool.

## Categories at a glance
- **data-access** — read from Alien datasets. `keyword_search`, `vector_search`
  (semantic), `get_entries`, `get_entry`, `get_entry_content`, `download_entry`,
  `list_datasets`, `get_dataset`, `get_collection`, `get_weights`. These are the
  agent's window into ingested data. Note: for querying your own published data,
  attaching a **published MCP config** via `alien_add_mcp_server` is usually
  cleaner than wiring raw data-access nodes (see `alien-build-agent`).
- **data-manipulation** — reshape arrays of objects. `aggregate` (collect fields
  across items), `edit_fields` (rename/rewrite fields), `docx_extract`.
- **documents / AI** — `mistral_ocr` (OCR a file to text + images),
  `rerank_search_results` (re-rank search hits with a cross-encoder or LLM
  fallback).
- **artifacts** — produce/consume downloadable files, designed to be agent
  tools: `docx_write`/`docx_read`, `pdf_write`/`pdf_read`,
  `xlsx_write`/`xlsx_read`. A `*_write` node emits a file attached to the
  agent's response; a `*_read` node reads one back by id.
- **audio** — `voice_generation` (ElevenLabs TTS; pair with `get_weights` to
  resolve a cloned `voice_id`).
- **agent-structural** (attached, never run as flow nodes): `subagent` (nested
  specialist LLM — use `alien_add_subagent`), `mcp_server` (remote MCP tools —
  use `alien_add_mcp_server`), `ask_user` (human-in-the-loop prompt, session
  mode only), plus orchestrator-only `patch_mcp_tool` / `mcp_cache_invalidator`.

## How attachment maps to node type
- A generic node (data-access, data-manipulation, documents, artifacts, audio)
  → `alien_add_tool` with its `tool_type`. It becomes one tool whose arguments
  are the node's input schema. Optionally pin params to lock config the LLM
  can't override.
- An MCP server → `alien_add_mcp_server` (it expands into many tools).
- A subagent → `alien_add_subagent` (it's a full nested agent, not a single
  tool). `alien_list_agent_tools` flags these with an `attach_with` hint.

## Gotchas
- Data-access nodes address data in one of two modes: **global**
  (`dataset_id`/`dataset_ids`, auto-routed) or **direct** (`cluster_id` + local
  ids, discovered via `list_datasets`). When pinning params on an attached
  data-access tool, pick one mode — mixing them fails validation.
- `vector_search` is semantic (needs embeddings); `keyword_search` is literal
  with snippet context. Choose per task; `rerank_search_results` refines either.
- Artifact `*_read`/`*_write` and every generic node are cost-tracked per call;
  the cost surfaces in the tool description the agent sees.

## Next
- Attach these to an agent: `alien-build-agent`.
- Full parameter tables: [`reference/node-catalog.md`](../../reference/node-catalog.md).
- Pick the model behind the agent/subagent: `alien-ai-models`.
