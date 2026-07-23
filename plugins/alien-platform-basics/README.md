# alien-platform-basics
Skills for the Alien platform control-plane, exposed through the
Alien MCP server. Every tool is prefixed `alien_` and acts as the
authenticated caller (per-request token relay of the caller's own `oat_...`
token) — there is no separate service identity.

## Skills
- `alien-getting-started` — orientation: identity, abilities, cluster discovery. Start here.
- `alien-add-data` — Flow 1: provision a cluster, create a dataset, wire a pipeline preset, upload documents, poll ingestion.
- `alien-external-api-to-mcp` — Flow 2: register an external REST API as a connector and turn its endpoints into MCP tools.
- `alien-publish-mcp` — Flow 3: bundle cluster and connector tools into a shareable Alien MCP configuration.
- `alien-rbac-and-pricing` — Flow 5: visibility, org/user sharing, and reporting-only pricing.
- `alien-analytics` — Flow 6: dashboard metrics, consumer/dataset leaderboards, workflow compute spend, external-API usage.
- `alien-cleanup` — lifecycle deletes, in cascade-safe order.

## Next
Once documents are ingested and/or an MCP config is published, hand off to the
`alien-workflow-engine` plugin's skills (`alien-build-agent`,
`alien-workflow-nodes`, `alien-run-workflow`) to build and run an agentic
workflow wired to that config.
