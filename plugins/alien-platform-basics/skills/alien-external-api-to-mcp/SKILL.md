---
name: alien-external-api-to-mcp
description: Turn an external REST API into Alien MCP tools — register a connector, import its OpenAPI spec, review and configure endpoints, test one live, then generate MCP tools. Use whenever the user wants to wrap, connect, or expose a third-party API (e.g. OpenAlex, an internal REST service) as tools.
---

# Alien: External API → MCP (Flow 2)
A connector holds the base URL and shared auth for an upstream HTTP API.
Endpoints are imported from that API's OpenAPI document, then selectively
enabled/priced/published, and finally turned into MCP tool definitions that
can be attached to an MCP config (see `alien-publish-mcp`).

## Call sequence
1. `alien_create_connector` — `name`, `base_url`, `auth_type` required.
   `auth_type` is one of `none`, `api-key`, `api-key-query`, `bearer-token`,
   `oauth2-client-credentials`:
   - `api-key` / `api-key-query` / `bearer-token` need `auth_credential`
     (the secret) and, for the api-key variants, `auth_param_name` (the
     header or query param name). `auth_scheme` optionally prefixes a
     header value (e.g. `Bearer`).
   - `oauth2-client-credentials` needs `oauth_token_url`,
     `oauth_client_id`, `oauth_client_secret`; optional `oauth_scope`,
     `oauth_client_auth_method`.
   Optional regardless of auth type: `description`, `default_headers`
   (object), `default_timeout` (ms, 1000–120000).
2. `alien_import_endpoints_from_openapi` — `connector_id`, `spec` required
   (`spec` is a JSON object or a raw JSON/YAML string — the upstream OpenAPI
   document). Optional `base_url` override when the spec's declared server
   URL is missing/relative/wrong, and `generate_defaults` (bool) to also seed
   default MCP tool metadata during import.
3. `alien_list_endpoints` — `connector_id` required. Review the imported
   endpoints and collect the `endpoint_id`s you need for the next steps.
4. `alien_configure_endpoints` — `connector_id`, `endpoint_ids` (1–500)
   required, plus at least one of `is_enabled`, `is_public`, `pricing_mode`,
   `unit_price_cents` — the call is rejected if none are set. Public proxy
   access to an endpoint requires BOTH the endpoint and the connector itself
   to be public (connector visibility is set separately, see
   `alien-rbac-and-pricing`).
5. `alien_test_endpoint` — `connector_id`, `endpoint_id` required; optional
   `request_params` (query params object), `request_body` (object).
   Live-probes the endpoint against the real upstream API to verify auth and
   shape before trusting it. Do this before generating tools for an endpoint
   you're unsure about.
6. `alien_generate_mcp_tools` — `connector_id` required; optional
   `endpoint_ids` to restrict generation to a subset (omit to generate
   defaults for all eligible endpoints). Produces the MCP tool definitions
   that `alien-publish-mcp` will reference by name.

## Gotchas
- Steps 4–6 need enabled/public endpoints to have any effect downstream —
  running `alien_generate_mcp_tools` on a disabled endpoint won't make it
  reachable through the public proxy.
- `pricing_mode` and `unit_price_cents` set here are the SAME batch-update
  path used by `alien_set_endpoint_pricing` (see `alien-rbac-and-pricing`) —
  you can set pricing in this call or defer it to that skill; don't do both
  inconsistently.
- `alien_import_endpoints_from_openapi` passes `spec` through to the backend
  largely unchanged; malformed OpenAPI documents fail at import, not later.
- Requires organization role MEMBER or higher and a write-capable token for
  `alien_create_connector`.

## Next
- Bundle the generated tools (plus optional cluster tools) into a shareable
  config: `alien-publish-mcp`.
- Control connector/endpoint visibility and pricing further: `alien-rbac-and-pricing`.
- See how this connector is being used: `alien-analytics`.
- Removing a connector entirely (cascades to its endpoints and tools): `alien-cleanup`.
