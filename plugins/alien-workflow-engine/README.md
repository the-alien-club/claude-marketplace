# alien-workflow-engine
Skills that teach an agent how to build and run **agentic workflows** on the
Alien platform through the Alien MCP server. This is the second
half of the platform journey: once you have data or a published MCP config
(see the `alien-platform-basics` plugin), these skills compose an agent out of
nodes and run it via the OpenAI-compatible Responses API.

## Skills
- **alien-workflow-engine** — orientation: the node/DAG model, how nodes become
  tools, the agent shape, and the four things you do here. Start here.
- **alien-build-agent** — the core loop: clone a preset with `alien_create_agent`,
  then attach tools, MCP servers, and subagents, inspecting and updating by
  `node_id`.
- **alien-workflow-nodes** — the catalog of attachable nodes (data-access,
  data-manipulation, documents/AI, artifacts, audio, agent-structural) and how
  each is attached.
- **alien-ai-models** — list models and get the exact `slug` for an agent or
  subagent.
- **alien-run-workflow** — run async and poll, smoke-test synchronously, or hand
  a real client the Responses-API connection info.

## Reference
- `reference/node-catalog.md` — full per-node parameter tables.
- `reference/workflow-architecture.md` — how the platform builds and executes a
  graph, and how the builder tools edit it.

## Requirements
An MCP client connected to the Alien MCP server with a valid access token that
has the `workflow:*` abilities. Building and running agents needs write/run
permission — a read-only token can only inspect.
