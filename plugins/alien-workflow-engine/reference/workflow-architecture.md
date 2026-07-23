# Alien Workflow Architecture (reference)
Deeper background for the workflow engine, for when a skill isn't enough. This
describes how the platform executes a graph and how the `alien_*` builder tools
edit it. You rarely need this to *use* the
MCP — the builder tools hide it — but it explains why things are shaped the way
they are.

## Graph representation
A workflow is `{"nodes": [...], "edges": [...]}`.

A node envelope:
```json
{
  "id": "vectorSearch",
  "type": "vector_search",
  "position": {"x": 0, "y": 0},
  "data": {
    "label": "Vector Search",
    "isTool": true,
    "isInput": false, "isOutput": false,
    "handles": ["tool"],
    "params": { "query": {"value": "", "isExpression": false, "isAttachedToInputNode": false} },
    "schema": { "input": { }, "output": { } },
    "workflow": { "nodes": [], "edges": [] }
  }
}
```
Every param is wrapped in an envelope `{value, isExpression, isAttachedToInputNode}`.
`isExpression: true` means `value` is a `{{ ... }}` template resolved at runtime.
`data.workflow` is present only on container nodes (`ai_agent`, `group`).

An edge:
```json
{"id": "e1", "source": "nodeA", "sourceHandle": "outputs", "target": "nodeB", "targetHandle": "inputs"}
```
The handle pair determines the edge type:
- `outputs → inputs` → **flow** edge — drives data flow and topological
  execution order.
- any other source handle (`tools`, `agents`) → **structural** edge — attaches a
  tool/subagent to an agent node, ignored for execution order.

Canonical attach wiring (plural source handle → singular target handle):
```
deep_agent.tools  → toolNode.tool        (native tool node OR mcp_server)
deep_agent.agents → subagent.agent
subagent.tools    → toolNode.tool        (a subagent's own tools)
```

## Execution pipeline
1. **Build.** The engine turns `{nodes, edges}` into one directed graph. Container
   nodes (those with `data.workflow`) are **dissolved**: their inner graph is
   built recursively and spliced in, and edges touching the container are
   re-stitched to its entry/exit nodes. The container node itself is never
   executed. A valid graph needs ≥1 flow edge and an acyclic flow subgraph.
2. **Order.** Execution order is a topological sort over **flow edges only**
   (structural edges never affect order). A dissolved container's id is aliased
   back onto its exit node's output so `{{ @aiAgent.value }}` still resolves.
3. **Execute.** Each node's item count comes from its flow-predecessors' output
   lengths; the node runs, and its `inputs`/`outputs`/`errors` are recorded into
   the shared run context.
4. Nodes flagged `data.isOutput: true` are collected into the job's
   `result.results` — for an agent that is the `http_response` node.

## Template resolution
The engine resolves `{{ ... }}` against the run context per array index:
- `@nodeId.field` — another node's output (broadcasts a length-1 output to every
  item; errors on incompatible fan-out lengths).
- `$context.field` — job-level (e.g. `jobId`).
- `$input.field` — the initial `http_request` payload.
- `$global.field` — platform config/constants.
- `$meta.nodeId.field` — timing/counts.
- `now()` — current ISO8601 timestamp.
Field lookup handles dot paths, `field[0]` indices, and camelCase↔snake_case.

## Nodes as tools (the two mechanisms)
- **Generic node → single tool**: a node is wrapped as `{type}_tool` whose args
  schema is the node's input schema. A fresh instance runs per call; failures
  return as tool messages, never exceptions. Pinning params at attach-time locks
  those values so the LLM's kwargs can't override them.
- **Agent compilation**: the `deep_agent`'s connected subgraph (via non-flow
  edges) is compiled into a real nested agent. `subagent` nodes become full
  nested agents (own model/prompt/tools, invoked via a `task` tool); `mcp_server`
  nodes expand into many remote tools (an unreachable server is logged and
  skipped, not fatal); everything else becomes a single-call tool.

## The agent shape
```
Outer boundary:  http_request → ai_agent → http_response
ai_agent.data.workflow (inner):  agent_input → deep_agent → agent_output
```
`alien_create_agent` clones the "Starter Agent" preset (id 111) and patches only
`model`, `system_prompt`, and optionally `sandbox` on the inner `deep_agent`,
leaving the session-param expression bindings (`session_id`/`user_prompt` bound
to `@agentInput-*`) untouched. That preserved wiring is exactly why hand-built
graphs fail at runtime and cloning is mandatory. The builder tools
(`alien_add_tool`, `alien_add_mcp_server`, `alien_add_subagent`,
`alien_remove_agent_component`) do stateless read-modify-write on `{nodes,
edges}` and address parts of the graph by the `node_id`s that
`alien_get_agent_definition` surfaces.
