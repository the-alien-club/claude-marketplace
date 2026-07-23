---
name: alien-workflow-engine
description: Understand the Alien workflow engine before building or running an agent — the node/DAG model, how nodes become tools, the agent-building loop, and how runs execute via the OpenAI Responses API. Use this to orient whenever a task involves building, editing, or running an Alien agent or workflow, or when you need to know what nodes/tools exist.
---

# Alien: Workflow Engine Overview
The Alien platform runs **workflows**: directed acyclic graphs of **nodes**
executed in topological order. An **agent** is a
specific, blessed workflow shape — an LLM node (`deep_agent`) with tools and
nested specialist LLMs (`subagent`s) attached — that the platform also serves
as an OpenAI-compatible `/responses` endpoint. Everything you build through the
Alien MCP server is one of these two things: a graph, or an agent-shaped graph.

## When to use
Read this first for any task that says "build an agent", "wire up a workflow",
"give the agent a tool", "connect my MCP to an agent", or "run the workflow".
It is the map; the other skills in this plugin are the territory.

## The mental model
- A workflow is `{"nodes": [...], "edges": [...]}`. Each node has a `type` (e.g.
  `deep_agent`, `vector_search`, `mcp_server`), a `data.params` block, and I/O
  schemas.
- Edges carry a handle pair. `outputs → inputs` is a **flow** edge — it defines
  data flow and execution order. Any other handle (`tools`, `agents`) is a
  **structural** edge that attaches a tool or subagent to an agent node and does
  NOT affect execution order.
- Two node families never run in the topological loop: **container** nodes
  (`ai_agent`, `group`) dissolve at build time and splice their inner graph in;
  **structural** nodes (`subagent`, `mcp_server`, `ask_user`, `patch_mcp_tool`)
  only exist to be attached via `tools`/`agents` edges and are consumed by the
  agent compiler, never executed directly. Reaching one via a `flow` edge fails
  the job.
- Data passes between nodes through template expressions: `{{ @nodeId.field }}`
  reads another node's output, plus `$context`, `$input`, `$global`, `$meta`,
  and `now()`. A param marked `isExpression: true` is resolved at runtime.

## Nodes are usable as tools
Any non-boundary node can be attached to an agent as a callable tool — the
platform wraps the node behind a tool whose argument schema is the node's own
input schema. So `vector_search` becomes a
`vector_search_tool` the LLM can call with a `query`. An `mcp_server` node is
different: it expands into *many* remote tools (one per MCP tool). A `subagent`
is different again: it is a full nested agent with its own model, prompt, and
tools, invoked through a `task` tool. See `alien-workflow-nodes` for the full
catalog and `alien-build-agent` for how attachment works.

## The four things you do here
1. **Discover** what exists — models (`alien-ai-models`), attachable tools
   (`alien_list_agent_tools`), and presets (`alien_list_agent_presets`).
2. **Build** an agent — clone a preset, then attach tools / MCP servers /
   subagents. This is `alien-build-agent`, the core loop.
3. **Run** it — trigger async and poll for the result, or smoke-test
   synchronously, or hand a real client the connection info. This is
   `alien-run-workflow`.
4. **Wire it to data** — attach a published Alien MCP config (from the
   `alien-platform-basics` plugin's `alien-publish-mcp` skill) as an
   `mcp_server` tool so the agent can query your datasets/endpoints.

## Gotchas
- Never hand-build an agent graph node-by-node. The session wiring
  (`session_id` / `user_prompt` expression bindings on `deep_agent`) is subtle;
  a hand-built graph passes schema validation but fails at runtime with
  `worker_disconnected`. Always clone a preset via `alien_create_agent`.

## Next
- Build or edit an agent: `alien-build-agent`.
- Full node/tool catalog: `alien-workflow-nodes`.
- Run and collect results: `alien-run-workflow`.
- Pick a model: `alien-ai-models`.
- Get data or an MCP config to attach first: the `alien-platform-basics`
  plugin (`alien-add-data`, `alien-external-api-to-mcp`, `alien-publish-mcp`).
