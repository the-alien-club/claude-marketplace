# MCP Tool Refinement — Internals (reference)
Grounding for the refinement skills: the data model, what can and cannot be
changed, the cache-invalidation requirement, the automated loop's stop
conditions, and the shipped backend feature. Use when a skill isn't enough or
when debugging a refinement run.

## The tool definition
Each generated MCP tool is one tool definition. The fields that matter for
refinement:
- **`toolName`** — the LLM-facing name shipped in `tools/list`. Unique within your
  organization. **Do not change it per-tool** (see the rename gotcha). Default is
  `{connector_slug}_{operation}`.
- **`mcpDescription`** — the description an MCP client actually sees in
  `tools/list`. **This is the primary refinement target.** Seeds from the
  endpoint's OpenAPI `description`/`summary` verbatim — i.e. a thin copy.
- **`description`** — UI display copy, not shipped in `tools/list`.
- **`summary`** — one-line UI summary.
- **`annotations`** — `readOnlyHint`, `destructiveHint`, `idempotentHint`,
  `openWorldHint`, `title`. Inferred from HTTP method at generation (GET/HEAD →
  read-only; others → destructive).
- **`inputSchema`** — the MCP `inputSchema` the client sees; a projection of the
  endpoint's parameter + request-body schema. Can be replaced wholesale during
  refinement.
- **`outputSchema`** — pre-filled from the endpoint's response schema,
  independently editable.
- **`isEnabled`, `isPublic`.**

Transport + schema truth (path, method, parameter/request/response schemas,
pricing, timeout) lives on the underlying endpoint record, not on the tool
definition.

## What `patch_mcp_tool` may change
The worker node `patch_mcp_tool` (type `patch_mcp_tool`) is the mechanism an
agent uses to edit a tool. Inputs:
- `connector_id` — **locked at compile time** via the workflow config
  (`{**kwargs, **config}`, config wins); the LLM cannot redirect the patch to
  another connector.
- `tool_id` — the tool definition's numeric id, LLM-supplied.
- `mcp_description` (min length 1) — the new LLM-facing description.
- `description`, `summary`, `annotations`, `input_schema` — optional.
Allowed patch fields are exactly `{mcpDescription, description, summary,
annotations, inputSchema}`. **`toolName` is excluded** — the node strips it
before issuing the edit, as defense-in-depth. It fails loudly (raises) on an
unbound connector, an empty description, or a malformed response — it never
fabricates empty output.

## The `toolName` rename gotcha (why renames are dangerous)
A published MCP config references its tools by **name string**, not by a stable
id. The platform cascades a rename ONLY when a connector's *slug* changes (it
rewrites every tool name under that connector AND every config that references
them, together). A rename of a single tool has **no cascade** — every saved
config listing the old name silently loses the tool. The `patch_mcp_tool` node
refuses `toolName` for exactly this reason. Safe rename = connector-slug rename
only.

## Cache invalidation (required after patching)
The platform caches each MCP server's tool catalogue for ~5 minutes. An edit does
not invalidate it. The `mcp_cache_invalidator` node
(type `mcp_cache_invalidator`, input `server_url` — also config-locked) drops
that cache for one server. Without it, the next audit (or any client) re-reads
the stale description and a refinement loop converges on nothing. Call it once
per patch round.

## After refinement — no re-publish
Because `toolName` never changes, MCP configs (which reference tools by name) and
the connection info from `alien_get_mcp_connection_info` stay valid. Only cache
invalidation is needed; there is no automatic invalidation on the PATCH itself.

## The automated loop (orchestrator / auditor / critic)
Three LLM roles:
- **Orchestrator** (`deep_agent` root) — driver only; dispatches auditor, then
  critic, applies approved patches via `patch_mcp_tool`, calls
  `mcp_cache_invalidator` once per round, decides whether to loop. Runs a
  self-check: patch-call count must equal the reported `total_patches_applied`,
  and auditor-dispatch count must equal `rounds_run`.
- **Auditor** (`subagent`, has the target `mcp_server` as its tool) — calls every
  in-scope tool at least once, finds factual gaps, proposes `mcp_description`
  rewrites as structured JSON. Fresh each round (no memory). "Ship bar": tool
  actually called, every required param documented, return shape documented,
  quirks documented or verified absent.
- **Critic** (`subagent`, no tools) — skeptical reviewer; accepts / edits /
  rejects proposals, rejecting unverified numeric claims, cosmetic-only rewrites,
  and evidence-free findings.
The auditor's `mcp_server` node uses a `tool_filter` clamped to the
`mcp_<toolName>` set under refinement — without it the auditor sees the whole org
catalogue and cost balloons.

Author each subagent's system prompt around the roles above — a probing auditor
held to the ship bar, and a skeptical critic that rejects unverified claims.

### Stop conditions
- `all_ship` — the latest audit returned zero `iterate` verdicts.
- `all_rejected` — the critic rejected every proposal this round AND the previous
  round also produced zero patches (two consecutive noise rounds).
- `round_cap` — 10 rounds completed.
- `budget` — the patch tool called ≥10 times total.

### Known quality limit
Convergence is self-referential: the auditor judging its own prior-round patches
under-reports subtler issues (field-name mismatches, pagination semantics,
"limit unknown" honesty). Spot-check final descriptions against the real API
before publishing.

## The automated feature (one call)
The Alien platform ships this identical loop as a built-in feature you trigger
with a single MCP call — `alien_refine_tool_descriptions`. It audits the
connector's enabled tools, critiques and applies improved descriptions, and
invalidates the cache — no agent assembly required. It runs in the background and
writes back; a concurrency guard prevents two refinement runs on the same
connector at once (a second trigger returns "already in progress"). Poll it with
`alien_get_tool_refinement_run`, whose result lands in `parsed_result`. This is
the default refinement path; build the loop by hand only for custom control.

## MCP tool reference (the refinement surface)
All four are Flow 2 tools. `connector_id` and `tool_id` are integers.

| Tool | Inputs | Returns / notes |
|---|---|---|
| `alien_list_mcp_tools` | `connector_id` (req), `enabled_only?` | Per tool: `tool_id`, `tool_name`, `mcp_description`, `is_enabled`, `is_public`, `external_api_endpoint_id`. `enabled_only` is filtered client-side. |
| `alien_update_mcp_tool` | `connector_id` (req), `tool_id` (req), + ≥1 of `mcp_description`, `description`, `summary`, `annotations`, `input_schema`, `output_schema`, `tags`, `is_enabled`, `is_public` | Edits one tool in place. Rejects a call with no editable field. **Does not accept `toolName`** — renames are blocked by design. |
| `alien_refine_tool_descriptions` | `connector_id` (req) | Triggers the automated audit→critique→patch loop; returns the created run. Writes back; 409/"already in progress" if a run is active. |
| `alien_get_tool_refinement_run` | `connector_id` (req), `run_id?` (poll one), `limit?` (list recent) | Poll a run (result in `parsed_result`, plus job timing) or list recent runs. |
