---
name: alien-build-agent
description: Build or edit an Alien agent — create it by cloning a preset, then attach tools, MCP servers, and specialist subagents, inspecting and updating as you go. Use whenever the task is to create an agentic workflow, give an agent a tool or data source, add a subagent, or change an agent's model/system prompt.
---

# Alien: Build an Agent
An Alien agent is a workflow with a fixed outer boundary
(`http_request → ai_agent → http_response`) wrapping an inner graph
(`agent_input → deep_agent → agent_output`). You never draw this by hand. You
clone a working preset with `alien_create_agent`, then mutate it with four
composable builder tools. Each builder is a read-modify-write on the graph's
`{nodes, edges}`; you address parts of the graph by `node_id`.

## When to use
Any request to "make an agent", "build a workflow that uses my data", "add a
web-search tool to the agent", "connect my MCP to an agent", "add a research
subagent", or "change the agent's prompt/model". First orient with
`alien-workflow-engine` and pick a model with `alien-ai-models`.

## The build loop
1. **Pick a model.** `alien_list_ai_models` (see `alien-ai-models`). You need a
   model `slug` (e.g. from the verified public LLM catalog) for the next step.
2. **Create the agent.** `alien_create_agent` — required `name`, `model` (the
   slug), `system_prompt`; optional `slug` (auto-derived from name),
   `sandbox` (bool, default false — enables the OpenSandbox code-exec runtime,
   session-mode only), `is_public` (bool, default false). This clones the
   "Starter Agent" preset (id 111), patches identity + behavior, and creates a
   brand-new workflow. It returns `{workflow_id, name, slug, model, ...}` — keep
   the `workflow_id`; every builder call needs it. It does NOT return the graph.
3. **See what you built.** `alien_get_agent_definition` — `workflow_id`. Returns
   the root agent (`node_id`, model, system_prompt, sandbox, tools) and each
   subagent (`node_id`, model, description, tools). The `node_id`s here are the
   handles you pass to every add/remove call. Call this whenever you need to
   target a specific node.
4. **Attach capabilities** (any mix, repeatable):
   - **A native tool** — `alien_add_tool`: `workflow_id`, `tool_type` (from
     `alien_list_agent_tools`), optional `params` (locks config values the LLM
     cannot override — e.g. a fixed `dataset_ids`), optional `target_node_id`
     (defaults to the root `deep_agent`; pass a subagent's node_id to give the
     subagent the tool instead). Returns `added_node_id`.
   - **An MCP server** — `alien_add_mcp_server`: `workflow_id`, `url` (an MCP
     endpoint — e.g. from `alien_get_mcp_connection_info` in the
     `alien-publish-mcp` skill), optional `tool_filter` (whitelist of tool
     names), optional `target_node_id`. Expands into many remote tools at run
     time. For Alien-hosted MCP configs you do NOT pass an auth token — it is
     auto-injected. Returns `added_node_id`.
   - **A specialist subagent** — `alien_add_subagent`: `workflow_id`, `model`,
     `system_prompt`, optional `description` (what the main agent sees when
     deciding to delegate). Returns `subagent_node_id`. A subagent is a full
     nested LLM. Give it its own tools by calling `alien_add_tool` /
     `alien_add_mcp_server` again with `target_node_id = subagent_node_id`.
5. **Adjust the root agent** — `alien_update_agent`: `workflow_id` plus any of
   `model`, `system_prompt`, `sandbox`, `name`, `is_public` (at least one). This
   patches the root `deep_agent` in place. It does NOT edit subagents — to
   change a subagent's model/prompt, remove and re-add it.
6. **Remove a component** — `alien_remove_agent_component`: `workflow_id`,
   `node_id`. Cascade-removes a tool, MCP server, or subagent (removing a
   subagent also removes the tools attached to it). Structural/boundary nodes
   (`http_request`, `http_response`, `ai_agent`, `agent_input`, `agent_output`,
   `deep_agent`) are protected and cannot be removed.

## Patterns worth copying
- **RAG agent over your own data**: create the agent → `alien_add_mcp_server`
  with the URL of a published Alien MCP config (from `alien-publish-mcp`) →
  optionally add a subagent scoped to search and give *it* the MCP server, so
  the coordinator delegates retrieval. This mirrors the seeded "Deep Agent"
  preset.
- **Reviewer subagent with no tools**: add a subagent whose only job is to
  critique/judge (`description` = "reviews drafts for X"), attach no tools to
  it. The main agent delegates via `task`. This is a real in-repo orchestrator
  pattern.
- **Config-locked utility tool**: `alien_add_tool` with `params` set pins those
  values so the LLM cannot change them at call time (e.g. bind a tool to one
  `connector_id` or `dataset_ids`).

## Gotchas
- The plural-source → singular-target handle wiring (`deep_agent.tools →
  node.tool`, `deep_agent.agents → subagent.agent`) is handled for you by the
  builder tools. If you ever inspect the raw graph, that is the shape to expect.
- Give a subagent its tools by re-running `alien_add_tool` with
  `target_node_id = <subagent_node_id>` — a subagent created without tools is
  just an extra LLM voice.
- `alien_create_agent` needs the `workflow:write` ability. If it fails on
  permissions, re-check the token via `alien_whoami` (`alien-getting-started`).
- Prefer `alien_list_agent_tools` over guessing tool types — it returns the
  exact `tool_type` strings and each tool's params, mirroring the editor
  palette. See `alien-workflow-nodes`.

## Next
- Run what you built: `alien-run-workflow`.
- Browse attachable tools/nodes: `alien-workflow-nodes`.
- Publish an MCP config to attach as a data source: the `alien-platform-basics`
  plugin's `alien-publish-mcp`.
