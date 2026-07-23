---
name: recipe-api-to-agent
description: End-to-end recipe to turn an external REST API (via its OpenAPI spec) into an Alien agent that can call it — create a connector, import and configure endpoints, generate MCP tools, publish, and wire the tools into an agent. Use when the goal is "make an agent that can use this API".
---

# Recipe: External API to Agent
Goal: start from an external REST API's OpenAPI spec and end with an agent that
can call that API as tools. Chains the connector flow into the workflow engine.

## When to use
"Give an agent access to this API", "wrap this OpenAPI spec so my agent can call
it", "let the agent hit our internal service". If you only need the MCP tools
(no agent), stop after step 4 and see `alien-external-api-to-mcp`.

## Steps
1. **Orient** (`alien-getting-started`). `alien_whoami` — confirm write ability.
2. **Create the connector and import endpoints** (`alien-external-api-to-mcp`).
   - `alien_create_connector` for the API's base URL/auth → keep the
     `connector_id`.
   - `alien_import_endpoints_from_openapi` with the spec → imports endpoints as
     candidates.
   - `alien_list_endpoints` on the connector → review what was imported.
3. **Configure and test the endpoints** (`alien-external-api-to-mcp`).
   - `alien_configure_endpoints` — enable the endpoints you want as tools and
     set/clean their descriptions and parameters. Good descriptions here decide
     whether the LLM calls the tool correctly later.
   - `alien_test_endpoint` on each enabled endpoint → confirm it returns real
     data before generating tools from it.
4. **Generate MCP tools** (`alien-external-api-to-mcp`).
   - `alien_generate_mcp_tools` → materializes the enabled endpoints as MCP tool
     definitions. If the generated descriptions/schemas are weak, refine them
     before publishing — tool quality directly drives agent accuracy. Quickest
     path: `alien_refine_tool_descriptions` (`connector_id`) then poll
     `alien_get_tool_refinement_run`; for surgical edits use `alien_list_mcp_tools`
     → `alien_update_mcp_tool`. See the `alien-mcp-tool-refinement` plugin.
5. **Publish as an MCP config** (`alien-publish-mcp`).
   - `alien_create_mcp_config` selecting the connector's tools → keep the id.
   - `alien_get_mcp_connection_info` → keep the MCP server `url`.
6. **Build the agent** (`alien-build-agent`, `alien-ai-models`).
   - `alien_list_ai_models` → model `slug`.
   - `alien_create_agent` → `workflow_id`.
   - `alien_add_mcp_server` with the `url` from step 5, optionally `tool_filter`
     to expose only a subset of the API's tools.
7. **Run and verify** (`alien-run-workflow`). Same as any agent:
   `alien_run_workflow` + `alien_get_workflow_run`, or
   `alien_get_agent_connection_info` for a real client.

## Gotchas
- Don't skip `alien_test_endpoint`. Generating tools from an endpoint that 401s
  or returns nothing yields a tool that always fails at agent runtime.
- Endpoint descriptions are the tool descriptions the LLM sees — vague ones
  cause wrong or skipped tool calls. Invest here or use the refinement plugin.
- Renaming a generated tool after publishing can orphan saved MCP configs — see
  the `alien-mcp-tool-refinement` plugin for the safe way.

## Next
- Improve generated tool quality: `alien-mcp-tool-refinement`.
- Combine this API source with a document source in one agent:
  `recipe-multi-source-research-agent`.
- Share or price the API: `alien-rbac-and-pricing`.
