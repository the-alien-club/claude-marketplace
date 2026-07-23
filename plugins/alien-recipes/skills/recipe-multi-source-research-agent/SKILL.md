---
name: recipe-multi-source-research-agent
description: End-to-end recipe to build an Alien coordinator agent with specialist subagents, each scoped to a different published source (a document dataset, an external API, another MCP). Use when the goal is one agent that reasons across several data sources by delegating to focused sub-agents.
---

# Recipe: Multi-Source Research Agent
Goal: one agent that answers questions spanning several sources by delegating to
specialist subagents — e.g. a documents subagent, an API subagent, and a
coordinator that synthesizes. This is the pattern behind the platform's own
Deep Agent preset and orchestrator examples.

## When to use
"An agent that searches both my docs and this API", "a research agent that
routes to the right source", "combine these three datasets under one agent". For
a single source, use `recipe-rag-agent-over-documents` or `recipe-api-to-agent`
instead — subagents add cost and latency you don't need for one source.

## Prerequisites
Each source must already be a published MCP config with a connection URL. Run
`recipe-publish-and-share-a-source` (or steps 1-3 of the single-source recipes)
once per source and collect each MCP server `url` before you start.

## Steps
1. **Pick models** (`alien-ai-models`). `alien_list_ai_models` → a coordinator
   model slug and (optionally) a cheaper slug for the specialist subagents.
2. **Create the coordinator** (`alien-build-agent`). `alien_create_agent` with a
   `system_prompt` describing its job as routing questions to the right
   specialist and synthesizing their answers → keep the `workflow_id`.
3. **Add one subagent per source** (`alien-build-agent`). For each source:
   - `alien_add_subagent` — `workflow_id`, `model`, a `system_prompt` scoping it
     to that one source, and a `description` the coordinator uses to decide when
     to delegate (make descriptions distinct and specific). Keep each returned
     `subagent_node_id`.
   - `alien_add_mcp_server` — `workflow_id`, the source's `url`, and
     `target_node_id = <that subagent's node_id>` so the tools attach to the
     subagent, not the coordinator.
4. **Verify the shape** (`alien-build-agent`). `alien_get_agent_definition` →
   confirm each subagent has exactly its intended source's tools and no others.
   A subagent with the wrong (or no) tools is the most common defect here.
5. **Run** (`alien-run-workflow`). `alien_run_workflow` + `alien_get_workflow_run`.
   Use async — a multi-subagent run will exceed the synchronous ~30s smoke-test
   window.

## Gotchas
- Attach each source to its subagent via `target_node_id`, not to the
  coordinator. Tools on the coordinator make it do the work itself instead of
  delegating, defeating the pattern.
- Distinct, specific subagent `description`s are what make routing work. "Handles
  documents" and "handles data" will get confused; "searches the 2024 SEC
  filings corpus" and "queries the live pricing API" will not.
- Keep the number of subagents small (the platform caps delegation depth). Group
  related sources rather than one subagent per dataset.

## Next
- Improve any API source's tool quality before wiring it in:
  `alien-mcp-tool-refinement`.
- Watch per-source consumption once it's running: `alien-analytics`.
- Debug a subagent that never gets called or always fails:
  `alien-troubleshooting`.
