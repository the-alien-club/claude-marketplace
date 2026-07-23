# alien-mcp-tool-refinement
When the Alien platform generates MCP tools from an external API's OpenAPI spec,
each tool's description seeds from the spec's `summary`/`description` verbatim —
usually one thin line. Thin descriptions make an LLM call the wrong tool, omit
required params, or skip the tool entirely. This plugin is about fixing that:
improving a generated tool's LLM-facing description (and, where needed, its input
schema) so agents use it correctly.

## Three ways to refine, all through the MCP
Pick by how much control you want:
1. **Automated (one call)** — `alien_refine_tool_descriptions` triggers a
   built-in audit → critique → patch loop over a connector's tools; poll
   `alien_get_tool_refinement_run` for the result. The default.
2. **Manual edit** — `alien_list_mcp_tools` to see each tool's `tool_id` and
   current description, then `alien_update_mcp_tool` to rewrite a specific one.
   Precise and cheap.
3. **Build your own loop** — assemble an orchestrator/auditor/critic agent for
   full control over the audit criteria. Advanced; #1 is the same loop packaged.

The one rule across all three: **never rename a tool** — it orphans every saved
MCP config that referenced it by name. `alien_update_mcp_tool` doesn't expose
`toolName` for exactly this reason.

## Skills
- **alien-refine-mcp-tools** — the concept, the three routes above, and the
  safety rule. Start here.
- **alien-edit-mcp-tools** — the manual path: list tools, then hand-edit a
  specific tool's description/schema/visibility with `alien_update_mcp_tool`.
- **alien-build-refinement-agent** — build a custom orchestrator/auditor/critic
  loop when the one-call automated path isn't enough. Advanced; builds on the
  `alien-workflow-engine` plugin.

## Reference
- `reference/refinement-internals.md` — the tool data model
  (`mcp_description` vs `description`), what each field controls, the
  cache-invalidation behavior, the automated loop's stop conditions, and the
  exact schemas of the refinement tools.

## Requirements
An MCP client connected to the Alien MCP server, a connector whose tools were
already generated (see the `alien-platform-basics` plugin's
`alien-external-api-to-mcp`), and `workflow:write` / `connector:write` abilities.
