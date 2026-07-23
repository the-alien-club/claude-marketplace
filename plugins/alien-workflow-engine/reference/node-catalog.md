# Alien Node Catalog (reference)
Full node reference for the Alien workflow engine. The live, authoritative list
is always `alien_list_agent_tools` (returns `tool_type` + params per node); this
file is a stable overview so you can plan before calling it. Node type strings
are the snake_case form (a camelCase alias also works, e.g. `vector_search` /
`vectorSearch`).

## How a node works
Every node has an `input_schema` and `output_schema`, a `type` string, and a
`process` step. Nodes run over an array of items; a
param can be a literal or an `isExpression` template (`{{ @nodeId.field }}`,
`$context`, `$input`, `$global`, `$meta`, `now()`) resolved per item at runtime.
Attaching a node to an agent as a tool exposes its input schema as the tool's
arguments; the node's `process()` runs on each call and never raises to the LLM
(errors come back as tool messages).

## data-access — read from Alien datasets
Two addressing modes, mutually exclusive per call:
- **global**: `dataset_id` / `dataset_ids` (auto-routed across clusters)
- **direct**: `cluster_id` + local ids (discover local ids via `list_datasets`)

| type | key inputs | outputs |
|---|---|---|
| `list_datasets` | `cluster_id` (req), `limit`, `offset` | `datasets[]`, `total`, `has_more`, `next_offset` |
| `get_dataset` | `cluster_id`, `dataset_id` (local) | `dataset` |
| `get_entries` | `dataset_id` XOR (`cluster_id`+`local_dataset_id`), `limit`, `mime_type`, `status` | `results[]` (entries) |
| `get_entry` | `entry_id`, `dataset_id` XOR `cluster_id` | `entry` (with file manifest) |
| `get_entry_content` | `entry_id`, dataset ref, `char_offset`, `char_limit` (0=all) | `text`, `total_length`, `has_more`, `next_offset` |
| `download_entry` | `entry_id`, dataset ref, `filename`, `file_type` (`original`/`processing`/`processed`) | `text_content`, `content_encoding` (utf-8/base64), `content_type`, `file_size_bytes` |
| `keyword_search` | `query`, dataset ref, `limit`≤100, `context_mode` (sentence/paragraph/section/smart), `min_score`, `tags`, `metadata_filters`, `sort` | `results[]` (hits w/ snippets), `total_count`, `has_more` |
| `vector_search` | `query`, dataset ref, `limit` (alias `k`), `score_threshold`, `entry_ids` | `documents[]` (`page_content`, `metadata`, `score`), `count` |
| `get_collection` | `collection_id` | `dataset_ids[]`, `count` |
| `get_weights` | `dataset_id`, `weight_index`, `type` (e.g. `voice_id`, `lora`) | `weight` (incl. `external_id`, local path), `count` |

## data-manipulation — reshape arrays
| type | inputs | outputs |
|---|---|---|
| `aggregate` | `items[]`, `fields[]` (`{label, to}`) | dict keyed by each `field.to` → list |
| `edit_fields` | `items[]`, `fields[]` (`{field_name, new_value}`) | `items[]` with edits |
| `docx_extract` | `file_url`, `entry` | `text`, `num_pages` |

## documents / AI
| type | inputs | outputs |
|---|---|---|
| `mistral_ocr` | `file_url`, `entry` | `text`, `images[]`, `num_pages` |
| `rerank_search_results` | `search_results[]`, `query`, `top_k`, `rerank_model`, `provider`, `use_dedicated_reranker` | `ranked_results[]`, `count`, `provider`, `model` (LLM fallback if no dedicated reranker) |

## artifacts — files as agent tools
`*_write` emits a downloadable file attached to the agent's response;
`*_read` reads one back by `artifact_id` / `file_id`.
| type | notes |
|---|---|
| `docx_write` / `docx_read` | Word documents |
| `pdf_write` / `pdf_read` | `pdf_write` requires exactly one of `html` / `markdown` |
| `xlsx_write` / `xlsx_read` | `xlsx_write` takes `sheets[]` (SheetSpec) |

## audio
| type | inputs | outputs |
|---|---|---|
| `voice_generation` | `text`≤50000, `voice_id` (from a `get_weights` `external_id`), `model` (default `eleven_turbo_v2_5`), `stability`, `similarity_boost`, `style` | `audio_base64`, `audio_format`, `character_count` |

## agent-structural (attached via tools/agents edges, never run as flow nodes)
| type | attach with | role |
|---|---|---|
| `subagent` | `alien_add_subagent` | nested specialist LLM (own model, prompt, tools), invoked via `task` |
| `mcp_server` | `alien_add_mcp_server` | remote MCP endpoint; expands into many tools. Alien-hosted MCPs need no auth token |
| `ask_user` | (preset only) | human-in-the-loop prompt; session mode only (needs checkpointer + session_id) |
| `patch_mcp_tool`, `mcp_cache_invalidator` | (orchestrator only) | MCP-refinement utilities with locked config |

## boundary / container (not attachable, listed for understanding)
- `http_request` (input) / `http_response` (output) — the outer HTTP boundary; the run's result lands in the `http_response` node's `results`.
- `ai_agent` / `group` — containers that hold an inner `{nodes, edges}` and **dissolve** at build time (spliced into the parent graph).
- `agent_input` / `agent_output` (`group_input` / `group_output`) — pass-through entry/exit of a container's inner graph; referenced as `{{ @agentInput.* }}` / `{{ @agentOutput.* }}`.
- `deep_agent` — the executable agent runtime; owns tools (`tools` edge) and subagents (`agents` edge).

## Gotchas
- Structural nodes' `process()` raises if reached by a `flow` edge — they must
  only ever be attached via `tools`/`agents` edges. The builder tools enforce
  this; only relevant if you inspect or hand-edit a raw graph.
- `rag_search` / `vector_search` are the same node (legacy alias). `k` is a
  deprecated alias for `limit` on `vector_search`.
