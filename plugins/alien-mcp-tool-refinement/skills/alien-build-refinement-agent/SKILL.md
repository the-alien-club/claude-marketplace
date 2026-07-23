---
name: alien-build-refinement-agent
description: Assemble and run a custom Alien agent that refines a connector's MCP tool descriptions — an orchestrator with auditor and critic subagents plus the patch_mcp_tool and mcp_cache_invalidator nodes. Use only when you need full control over the audit criteria; for normal refinement prefer the one-call alien_refine_tool_descriptions tool.
---

# Alien: Build a Refinement Agent
This builds the orchestrator → audit → critique → patch → invalidate loop by
hand, from the agent-builder tools. It's the same loop the one-call
`alien_refine_tool_descriptions` tool runs for you — assemble it yourself only
when you want to control the audit criteria, the models, or the stop conditions.

## When to use
Reach for this only when the built-in automated path isn't enough. **For normal
refinement, use `alien_refine_tool_descriptions` + `alien_get_tool_refinement_run`**
(see `alien-refine-mcp-tools`) — it's one call, no assembly, and it invalidates
the cache for you. Build your own loop when you need custom audit rules, a
specific model, or to extend the loop with extra checks.

## Advanced / validate-as-you-go
Assembling this by hand is more work and easier to get wrong than the one-call
tool. Run it against a throwaway/duplicate connector first, not a production one,
and check the result before trusting it.

## Prerequisites
- A connector with generated tools (`alien-external-api-to-mcp`).
- That connector's tools published as an MCP config, and its connection `url`
  from `alien_get_mcp_connection_info` (`alien-publish-mcp`).
- The tools to refine and their numeric `tool_id`s — get both from
  `alien_list_mcp_tools` (`connector_id`): use the `tool_name`s for the target
  MCP server's `tool_filter` (as `mcp_<tool_name>`) and the `tool_id`s for the
  `patch_mcp_tool` node.

## Assembly (uses the alien-workflow-engine plugin)
1. **Model.** `alien_list_ai_models` → a capable model `slug` (auditing needs a
   strong model).
2. **Create the orchestrator.** `alien_create_agent` — `name`, `model`, and a
   `system_prompt` that makes it a driver only: dispatch the auditor, then the
   critic, apply approved patches with `patch_mcp_tool`, call
   `mcp_cache_invalidator` once per round, then decide whether to loop. Keep the
   `workflow_id`.
3. **Add the auditor subagent.** `alien_add_subagent` — `workflow_id`, `model`, a
   `system_prompt` that makes it call every in-scope tool at least once and report
   factual gaps + proposed `mcp_description` rewrites as structured JSON, and a
   `description` the orchestrator uses to delegate. Keep `auditor_node_id`.
4. **Give the auditor live access to the target tools.** `alien_add_mcp_server` —
   `workflow_id`, `url` (the connector's published MCP config URL),
   `tool_filter` (the `mcp_<toolName>` list — clamp it to just the tools under
   refinement, or the auditor sees the whole org catalogue and cost balloons),
   `target_node_id = auditor_node_id`.
5. **Add the critic subagent.** `alien_add_subagent` — a skeptical reviewer with
   **no tools** that accepts / edits / rejects the auditor's proposals (rejecting
   unverified claims and cosmetic-only rewrites). Keep `critic_node_id`.
6. **Attach the patch tool to the orchestrator.** `alien_add_tool` —
   `workflow_id`, `tool_type = "patch_mcp_tool"`, `params = {"connector_id":
   <id>}` (this locks the connector so the LLM can't redirect patches),
   `target_node_id` defaults to the root orchestrator.
7. **Attach the cache invalidator.** `alien_add_tool` — `tool_type =
   "mcp_cache_invalidator"`, `params = {"server_url": <same MCP url>}`, on the
   root orchestrator.
8. **Verify the shape.** `alien_get_agent_definition` — confirm the auditor has
   the target MCP server, the critic has no tools, and the orchestrator has both
   `patch_mcp_tool` and `mcp_cache_invalidator`.

## Run and inspect
- `alien_run_workflow` — `workflow_id`, optional `input`. Background run; keep the
  job id.
- `alien_get_workflow_run` — poll. The orchestrator's JSON summary
  (rounds run, patches applied, stop reason) lands at
  `result.results.<http_response id>[0].results.content`. On `failed`, re-poll
  with `full: true` to see the failing node.
- The loop stops on: all tools shipped, two consecutive no-patch rounds, a
  round cap, or a patch-count budget. See `reference/refinement-internals.md`.

## Gotchas
- `patch_mcp_tool` only changes `mcpDescription` / `description` / `summary` /
  `annotations` / `inputSchema` — never `toolName`. It fails loudly on an empty
  description or an unbound connector.
- The `mcp_cache_invalidator` call each round is not optional: without it the
  next audit re-reads a stale cached catalogue and falsely reports "no findings",
  so the loop converges on nothing.
- Convergence is self-referential — the auditor judging its own prior patches
  tends to under-report. Spot-check the final descriptions yourself against the
  real API before publishing.
- Attach `patch_mcp_tool` / `mcp_cache_invalidator` only to this refinement
  orchestrator, never to a general-purpose agent.

## Next
- The concept and safety rules: `alien-refine-mcp-tools`.
- Agent-building mechanics (handles, node_ids, subagent wiring): the
  `alien-workflow-engine` plugin's `alien-build-agent`.
- Node params and stop conditions: `reference/refinement-internals.md`.
