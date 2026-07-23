# Alien Intelligence — Claude Code Marketplace

A Claude Code plugin marketplace for the Alien platform. It bundles two kinds of
plugin: a ready-to-use **open-science research toolkit** backed by hosted MCP
servers, and a set of **build-on-Alien** plugins that teach an agent how to
operate the Alien platform through its MCP — ingest data, wrap APIs as tools,
publish MCP configs, and build and run agentic workflows.

## Plugins

### Research toolkit (works out of the box)

| Plugin | What it gives you |
|--------|-------------------|
| **openscience** | Access to the OpenAIRE Research Graph (600M+ research products) and full-text bioRxiv/medRxiv preprints through three hosted MCP servers, plus guided research skills. Bundles its own MCP endpoints — no setup beyond installing the plugin. |

### Build on Alien (require the Alien MCP server + your access token)

| Plugin | Install when you want to… |
|--------|---------------------------|
| **platform-basics** | Operate the platform: ingest documents, wrap an external API as MCP tools, publish/share an MCP config, set pricing, read analytics, clean up. |
| **workflow-engine** | Build and run agents: compose nodes into an agent, attach tools/MCP servers/subagents, pick a model, and execute via an OpenAI-compatible Responses API. |
| **mcp-tool-refinement** | Improve the MCP tools generated from an API so agents call them correctly. |
| **recipes** | Follow an end-to-end cookbook: RAG-over-documents, API-to-agent, publish-and-share, multi-source research agent. |
| **troubleshooting** | Diagnose and fix a failing Alien operation via the MCP. |

The build-on-Alien plugins document the `alien_*` tools exposed by the Alien MCP
server; they do not stand up that server. Every call acts as the authenticated
caller (per-request token relay of your own `oat_...` token), so a read-only
token can inspect but not create.

## Installation

Add the marketplace, then install the plugins you want.

### From the marketplace (remote)

```bash
/plugin marketplace add the-alien-club/claude-marketplace
/plugin install openscience@alien
/plugin install platform-basics@alien
/plugin install workflow-engine@alien
/reload-plugins
```

### From a local checkout

```bash
/plugin marketplace add ./
/plugin install workflow-engine@alien
/reload-plugins
```

The path is relative to your current working directory; use a full path if you
are outside the repo.

### Connecting the Alien MCP server

The **openscience** plugin bundles its MCP endpoints, so it works as soon as it
is installed. The build-on-Alien plugins expect the Alien platform MCP server to
be connected to your client and authenticated with your access token — the
skills drive the `alien_*` tools that server exposes.

## Skills

**openscience** — `explore-openscience` (router), `explore-openaire` (+ nine
scenario workflows), `explore-biorxiv`, `explore-medrxiv`. Invoked with the
`/openscience:` prefix.

**platform-basics** — `alien-getting-started`, `alien-add-data`,
`alien-external-api-to-mcp`, `alien-publish-mcp`, `alien-rbac-and-pricing`,
`alien-analytics`, `alien-cleanup`.

**workflow-engine** — `alien-workflow-engine` (overview), `alien-build-agent`,
`alien-workflow-nodes`, `alien-ai-models`, `alien-run-workflow`.

**mcp-tool-refinement** — `alien-refine-mcp-tools`, `alien-build-refinement-agent`.

**recipes** — `recipe-rag-agent-over-documents`, `recipe-api-to-agent`,
`recipe-publish-and-share-a-source`, `recipe-multi-source-research-agent`.

**troubleshooting** — `alien-troubleshooting`.

Several build-on-Alien plugins bundle `reference/` docs (a node catalog, workflow
architecture, refinement internals, and a failure catalog) that their skills link
to for depth. Skills activate automatically when a task matches their
description — you don't call them by hand.

## Repository structure

```
.claude-plugin/
  marketplace.json              # Marketplace definition (name "alien", plugin list)
plugins/
  alien-openscience/            # openscience — hosted research MCP + skills (.mcp.json)
  alien-platform-basics/        # platform-basics
  alien-workflow-engine/        # workflow-engine (+ reference/)
  alien-mcp-tool-refinement/    # mcp-tool-refinement (+ reference/)
  alien-recipes/                # recipes
  alien-troubleshooting/        # troubleshooting (+ reference/)
```

Each plugin has its own `README.md` and, under `.claude-plugin/plugin.json`, its
metadata. Directories keep the `alien-` prefix; plugin names (used for
`install` and the `/name:` skill prefix) drop it.

## Managing plugins

```bash
/plugin list                              # See installed plugins
/plugin marketplace update alien          # Pull latest changes
/reload-plugins                           # Reload without restarting
```
