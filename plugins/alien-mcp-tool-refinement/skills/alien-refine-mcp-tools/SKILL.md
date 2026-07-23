---
name: alien-refine-mcp-tools
description: Improve the descriptions and schemas of the MCP tools the Alien platform generated from an external API, so an agent calls them correctly. Use when generated tools have thin one-line descriptions, when an agent picks the wrong API tool or omits required params, or when someone asks to refine, clean up, or improve generated MCP tool metadata.
---

# Alien: Refine MCP Tools
A generated MCP tool's LLM-facing description (`mcpDescription`) seeds from the
OpenAPI spec's `summary`/`description` verbatim — typically one thin line, often
missing required params, return shape, and quirks. That is what makes an agent
call the wrong tool or call it wrong. Refinement rewrites those descriptions
(and, where needed, the input schema) against the tool's real behavior.

## When to use
Generated tools exist (via `alien-external-api-to-mcp`) but agents use them
poorly, descriptions are one-liners, or you're asked to "clean up the tools"
before publishing. Refine *before* you publish and share (see the
`alien-recipes` plugin's `recipe-api-to-agent`) — quality here drives every
downstream agent's accuracy.

## Three ways to refine, all through the MCP
Pick by how much control you want:
1. **Automated (one call)** — let the platform do it. `alien_refine_tool_descriptions`
   (`connector_id`) triggers a built-in audit → critique → patch loop over the
   connector's tools, then poll `alien_get_tool_refinement_run` for the result.
   This is the default; it is the least effort and handles cache invalidation
   itself. It writes back and costs money/time — run it on a connector you own.
2. **Manual edit** — inspect and hand-edit specific tools. `alien_list_mcp_tools`
   to see each tool's `tool_id` and current `mcp_description`, then
   `alien_update_mcp_tool` to rewrite one. Use when you know exactly what a
   description should say and want precise, cheap control. See `alien-edit-mcp-tools`.
3. **Build your own loop** — assemble an orchestrator/auditor/critic agent
   yourself for full control over the audit criteria. Advanced; the automated
   path (#1) is the same loop packaged. See `alien-build-refinement-agent`.

Start with #1 unless you have a specific reason to hand-edit or customize.

## What refinement may change
On each tool: `mcp_description` (the description an MCP client actually sees — the
important one), `description` and `summary` (UI copy), `annotations`,
`input_schema` and `output_schema` (full replacement JSON Schemas), `tags`, and
`is_enabled` / `is_public`. It must NOT change `toolName`. See
`reference/refinement-internals.md` for the field-by-field detail.

## The one safety rule
**Never rename a generated tool.** MCP configs store their tool list as plain name
strings, not references. A per-tool `toolName` change has no cascade, so every
saved MCP config that listed the old name silently loses the tool. This is why
`alien_update_mcp_tool` does not expose `toolName` at all. If a rename is
genuinely required, it must go through a *connector slug* rename (which cascades
tool names and every config that references them together) — never a per-tool
edit.

## After refining
Nothing needs re-publishing. Because `toolName` never changes, existing MCP
configs and the connection info from `alien_get_mcp_connection_info` stay valid.
The platform caches each MCP server's tool catalogue for ~5 minutes: the
automated path invalidates it for you; after a manual `alien_update_mcp_tool`,
allow up to the TTL for running agents to see the new description.

## Next
- Automated path detail + the poll loop: `reference/refinement-internals.md`.
- Hand-edit specific tools: `alien-edit-mcp-tools`.
- Build a custom refinement loop: `alien-build-refinement-agent`.
- Generate the tools in the first place: the `alien-platform-basics` plugin's
  `alien-external-api-to-mcp`.
