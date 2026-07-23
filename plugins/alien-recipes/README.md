# alien-recipes
Outcome-oriented cookbook for the Alien platform. The `alien-platform-basics`
and `alien-workflow-engine` plugins document each `alien_*` tool in isolation;
this plugin chains them into complete, common outcomes so an agent goes from
goal to working result without re-deriving the sequence each time.

Each recipe is a single skill that names the exact tools, the order, the params
that carry state between steps, the polling points, and how to verify success.
Recipes reference the atomic skills for per-tool detail rather than repeating it.

## Recipes
- **recipe-rag-agent-over-documents** — from raw documents to an agent that
  answers questions over them: ingest → publish an MCP config → build an agent
  wired to it → run.
- **recipe-api-to-agent** — from an external REST API's OpenAPI spec to an agent
  that can call it: connector → endpoints → generated MCP tools → publish →
  attach to an agent.
- **recipe-publish-and-share-a-source** — take data or an API you already have
  and turn it into a shareable, optionally priced MCP source for your org or
  another user.
- **recipe-multi-source-research-agent** — a coordinator agent with specialist
  subagents, each scoped to a different published source.

## Requirements
An MCP client connected to the Alien MCP server with a token that
has write + run abilities. Recipes assume the atomic skills from
`alien-platform-basics` and `alien-workflow-engine` are available for detail.
