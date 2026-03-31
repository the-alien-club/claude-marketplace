# Alien Open Science Plugin

A Claude Code plugin that connects AI assistants to open research infrastructure. The plugin provides access to the OpenAIRE Research Graph (600M+ research products across all disciplines) and full-text preprint collections from bioRxiv and medRxiv — through three MCP servers and a set of guided research skills.

## What It Does

The plugin registers three MCP servers that expose research data as tools Claude can call directly:

| MCP Server | Source | Content |
|------------|--------|---------|
| **openaire** | OpenAIRE Research Graph | Structured metadata — titles, abstracts, authors, citations, bibliometrics, funded projects, and cross-product relationships across 600M+ research products |
| **datacluster-medrxiv** | medRxiv preprints | Full-text clinical and health sciences preprints across 50+ medical specialities, with keyword and semantic search |
| **datacluster-biorxiv** | bioRxiv preprints | Full-text biology preprints across 50+ subject categories, with keyword and semantic search |

OpenAIRE provides the metadata layer — who published what, where, with what funding, and how it was cited. The bioRxiv and medRxiv data clusters provide the content layer — the actual text of preprints, searchable by keyword or semantic similarity.

## Skills

The plugin ships four research skills that guide Claude through common research workflows:

- **explore-openscience** — Entry point and router. Explains when to use each source and how to combine them for a given research question.
- **explore-openaire** — Detailed guide for the OpenAIRE Research Graph, with nine scenario workflows covering literature review, author profiling, citation analysis, bibliometric assessment, dataset discovery, and more.
- **explore-biorxiv** — Guide for searching and reading full-text biology preprints via the bioRxiv data cluster.
- **explore-medrxiv** — Guide for searching and reading full-text medical preprints via the medRxiv data cluster.

Skills are invoked with the `/openscience:` prefix:

```
/openscience:explore-openscience
/openscience:explore-openaire
/openscience:explore-biorxiv
/openscience:explore-medrxiv
```

## Installation

### From the marketplace (remote)

Add the marketplace and install the plugin:

```bash
/plugin marketplace add the-alien-club/mcp-plugins
/plugin install openscience@alien
/reload-plugins
```

### From a local directory

If you have the repository cloned locally, point the marketplace at the repo root:

```bash
/plugin marketplace add ./
/plugin install openscience@alien
/reload-plugins
```

The path is relative to your current working directory. If you are outside the repo, use the full path:

```bash
/plugin marketplace add /path/to/mcp-plugins
```

### Manual MCP registration

You can also register the MCP servers individually without using the plugin system:

```bash
claude mcp add openaire --transport http https://openaire.mcp.alien.club/mcp
claude mcp add datacluster-medrxiv --transport http https://medrxiv.mcp.alien.club/mcp
claude mcp add datacluster-biorxiv --transport http https://biorxiv.mcp.alien.club/mcp
```

This registers the servers but does not install the research skills.

## Repository Structure

```
.claude-plugin/
  marketplace.json            # Marketplace definition (name, owner, plugin list)
plugins/
  alien-openscience/
    .claude-plugin/
      plugin.json             # Plugin metadata (name, version, description)
    .mcp.json                 # MCP server endpoints for this environment
    skills/
      explore-openscience/
        SKILL.md              # Research router skill
      explore-openaire/
        SKILL.md              # OpenAIRE research graph skill
        scenarios/            # Nine detailed workflow guides
      explore-biorxiv/
        SKILL.md              # bioRxiv full-text search skill
      explore-medrxiv/
        SKILL.md              # medRxiv full-text search skill
```

## Managing the Plugin

After installation, use these commands to manage the plugin:

```bash
/plugin list                              # See installed plugins
/plugin marketplace update alien          # Pull latest changes
/reload-plugins                           # Reload without restarting
```
