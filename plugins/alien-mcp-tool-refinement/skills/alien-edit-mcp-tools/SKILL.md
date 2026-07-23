---
name: alien-edit-mcp-tools
description: Inspect and hand-edit the MCP tools generated from an Alien connector — list them to get their ids and current descriptions, then rewrite a specific tool's description, schema, annotations, or visibility. Use when you want precise manual control over one tool's metadata rather than the automated refinement loop.
---

# Alien: Edit MCP Tools
The manual, precise path to refinement: list a connector's generated tools, then
edit exactly the ones you choose. Use this when you know what a description
should say, or need to fix one tool's input schema or visibility, and don't want
to run (or pay for) the automated loop. For bulk automatic improvement, use
`alien-refine-mcp-tools`.

## When to use
"Fix the description on this one tool", "the `search` tool's schema is wrong",
"make this tool private", "the agent doesn't know param X is required — document
it". Any surgical edit to a generated tool's metadata.

## Steps
1. **List the tools** — `alien_list_mcp_tools`: `connector_id` (required),
   `enabled_only` (optional bool). Returns one entry per tool with `tool_id`,
   `tool_name`, `mcp_description`, `is_enabled`, `is_public`, and
   `external_api_endpoint_id`. You need the `tool_id` for the edit.
2. **Edit a tool** — `alien_update_mcp_tool`: `connector_id` and `tool_id`
   (both required), plus **at least one** field to change:
   - `mcp_description` — the description an MCP client sees. The main lever.
   - `description`, `summary` — UI copy.
   - `annotations` — MCP hints (`readOnlyHint`, `destructiveHint`, etc.).
   - `input_schema`, `output_schema` — full JSON Schema replacements (each must be
     a valid object schema with `type` and `properties`).
   - `tags` — string list.
   - `is_enabled`, `is_public` — visibility/availability.
   The call fails if you pass no editable field. It does **not** accept
   `toolName` — renames are blocked by design (see the safety rule below).
3. **Re-list to confirm** — `alien_list_mcp_tools` again to verify the change
   took, if you want a read-back.

## Writing a good `mcp_description`
This is the text the LLM reads when deciding whether and how to call the tool.
Make it state: what the tool does, every required parameter and what it expects,
the shape of what it returns, and any quirk (pagination, rate limits, an "unknown
max" honestly). A one-line restatement of the endpoint name is the failure mode
you're fixing.

## The safety rule
`alien_update_mcp_tool` cannot rename a tool, and you should not try to via any
other route. MCP configs reference tools by name string; a per-tool rename has no
cascade and silently drops the tool from every config that listed it. A genuine
rename must go through a *connector slug* rename, which cascades everything
together.

## After editing
No re-publish needed — the `toolName` is unchanged, so published MCP configs and
`alien_get_mcp_connection_info` stay valid. Allow up to ~5 minutes (the tool
catalogue cache TTL) for running agents to pick up the new description.

## Next
- Bulk / automatic improvement instead: `alien-refine-mcp-tools`.
- Field-by-field data model: `reference/refinement-internals.md`.
- The connector/tool-generation flow that created these tools: the
  `alien-platform-basics` plugin's `alien-external-api-to-mcp`.
