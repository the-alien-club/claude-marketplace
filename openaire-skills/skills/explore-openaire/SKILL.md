---
name: explore-openaire
description: OpenAIRE Research Graph Exploration Skill. Use when exploring research metadata, finding papers, analyzing citations, discovering datasets, or assessing research impact via the OpenAIRE MCP server.
---

# OpenAIRE Research Graph Exploration Skill

You are exploring research metadata through the OpenAIRE MCP server. This server exposes ~28 tools across four API layers: Graph API v1 (organizations, projects, persons, data sources), Graph API v2 (research products, bibliometric filters, datasets), ScholeXplorer (cross-product relationships), and composite/analytical tools (citation networks, author profiles, trend analysis).

## Research Scenarios

When the user's intent matches a scenario below, **you MUST read the scenario guide** before proceeding. Each guide contains domain context, step-by-step workflows, and tool-specific gotchas. Access via either method:

- **MCP resource** (preferred): Use `ReadMcpResourceTool` with `server="openaire-local"` and the URI below.
- **Local file**: Use the `Read` tool with the file path below (relative to this skill's directory).

| Scenario | When to use | MCP Resource URI | Local file |
|----------|-------------|------------------|------------|
| **Literature review** | Surveying a field, finding key papers | `openaire://scenario/literature-review` | `scenarios/literature-review.md` |
| **Author landscape** | Exploring a researcher's work and collaborations | `openaire://scenario/author-landscape` | `scenarios/author-landscape.md` |
| **Project impact** | Assessing outputs of a funded project | `openaire://scenario/project-impact` | `scenarios/project-impact.md` |
| **Cross-domain discovery** | Finding methods/data from outside home field | `openaire://scenario/cross-domain-discovery` | `scenarios/cross-domain-discovery.md` |
| **Dataset discovery** | Finding research datasets | `openaire://scenario/dataset-discovery` | `scenarios/dataset-discovery.md` |
| **Co-citation analysis** | Revealing intellectual structure via citation patterns | `openaire://scenario/co-citation-analysis` | `scenarios/co-citation-analysis.md` |
| **Bibliometric assessment** | Finding landmark, trending, or high-impact papers | `openaire://scenario/bibliometric-assessment` | `scenarios/bibliometric-assessment.md` |
| **Find primary publication** | Locating the canonical paper for a tool/method/dataset | `openaire://scenario/find-primary-publication` | `scenarios/find-primary-publication.md` |
| **Assess dataset relevance** | Evaluating dataset suitability for research | `openaire://scenario/assess-dataset-relevance` | `scenarios/assess-dataset-relevance.md` |

To discover all available scenarios and vocabulary resources at runtime, call `ListMcpResourcesTool` with `server="openaire-local"`.

## Context Budget Awareness

### Delegation Threshold

You **MUST** use the Task tool (`subagent_type="general-purpose"`) to delegate these — never call directly in the main context:

| Tool | Always delegate? | Reason |
|------|-----------------|--------|
| `get_project_outputs` | **Yes — always** | 30K+ chars at default limits |
| `analyze_coauthorship_network` | **Yes — always** | 15K+ chars even at depth=1 |
| `get_author_profile` (limit>30) | **Yes** | Scales linearly with limit |
| `get_research_product_details` for 3+ DOIs | **Yes — parallel** | One Task per DOI in a single message |
| `get_citation_network` (max_nodes>100) | **Yes** | 6K+ chars |

Use foreground subagents only. Launch multiple Task calls in the same message for parallelism.

### Subagent Templates

**Product summary:**
```
Use mcp__openaire-local__openaire_get_research_product_details to get metadata
for DOI <doi>. Summarize in ~150 words: research question, methodology, key
findings, significance. Report: title, authors, year, citation count, influence
class, and your summary.
```

**Project output analysis:**
```
Use mcp__openaire-local__openaire_get_project_outputs with project_code='<code>'
and page_size=100. Summarize: total outputs by type, top 5 most-cited
publications, any datasets or software produced. Report counts and key DOIs.
```

**Coauthorship network:**
```
Use mcp__openaire-local__openaire_analyze_coauthorship_network with
author_name='<name>', max_depth=1, limit=30, min_collaborations=2.
Summarize: total collaborators, top 5 by shared papers, any cross-institutional
clusters. Report node count, edge count, and key collaborator names.
```

### Detail Level Heuristic

- `detail="minimal"` for routing and exploration (~100 chars/result)
- `detail="standard"` for assessment and comparison (~500 chars/result)
- `detail="full"` only for single-item deep dives (~800 chars/result)
- Prefer `page_size=5` for initial searches. Paginate for more.
- Use `response_format="json"` throughout (JSON and markdown are similar size; JSON is more reliable for extracting DOIs).

### Parallel Searches

OpenAIRE tools are stateless — run multiple searches in parallel when exploring multiple facets (e.g., publications + datasets + software for the same topic).

## Output Size Reference

| Size | Chars | Tools |
|------|-------|-------|
| **Tiny** | 200-600 | `analyze_research_trends`, `find_by_*_class` (5 results), search with `detail="minimal"` + `page_size=3` |
| **Small** | 600-1500 | `get_research_product_details`, `search_datasets` (5 results) |
| **Medium** | 1500-2500 | `explore_research_relationships`, `get_citation_network` (max_nodes=50), `discover_by_coauthors` |
| **Large** | 2500-4000 | `get_author_profile` (limit=20), `search_research_products` (standard, 10 results) |
| **Context bomb** | 15K-30K+ | `get_project_outputs` (default), `analyze_coauthorship_network`, `get_author_profile` (limit=100+) |

## Reference Tables

### Bibliometric Classes

| Class | Percentile | Meaning |
|-------|-----------|---------|
| C1 | Top 0.01% | Landmark/foundational |
| C2 | Top 0.1% | Highly notable |
| C3 | Top 1% | Field-leading |
| C4 | Top 10% | Above average |
| C5 | Bottom 90% | Average |

Four metrics: **influence** (field-normalized, time-adjusted), **popularity** (recent velocity), **impulse** (early momentum), **citation_count** (absolute).

### FOS Top-Level Codes

Use full label with code prefix for the `fos` parameter: `"01 natural sciences"`, `"02 engineering and technology"`, `"03 medical and health sciences"`, `"04 agricultural and veterinary sciences"`, `"05 social sciences"`, `"06 humanities and the arts"`.

Common: `"0102 computer and information sciences"`, `"0301 basic medicine"`, `"0502 economics and business"`, `"0106 biological sciences"`.

### Response Structure Differences

- Most search tools: `results[]`
- `search_datasets`: `datasets[]`
- Graph tools (`get_citation_network`, `build_subgraph_from_dois`, `analyze_coauthorship_network`): `nodes[]` + `edges[]`

## Error Patterns

| Error | Cause | Fix |
|-------|-------|-----|
| 0 results from `explore_research_relationships(doi=)` | Outgoing refs sparsely indexed | Use `target_pid=` with `relationship_type="cites"` |
| 0 results with `source_subtype`/`target_subtype` | Filter is broken | Remove these params; filter client-side |
| 0 datasets from `find_datasets_by_topic` | Sparse pub-to-dataset links | Use `search_research_products(type=["dataset"])` |
| 0 results from `get_author_profile(orcid=)` | ORCID coverage sparse | Use `author_name=` instead |
| 0 results from `search_persons` | Person index very sparse | Use `get_author_profile(author_name=)` |
| Empty from `discover_by_subject` on famous papers | Noisy SDG tags | Use keyword search instead; works better on domain-specific papers |
| `max_publications` validation error | Below minimum of 5 | Set `max_publications=5` or higher |
| `"Metadata unavailable"` in `build_subgraph_from_dois` | DOI not in OpenAIRE | Graceful degradation, not an error |

## Validation Constraints

| Tool | Parameter | Constraint |
|------|-----------|-----------|
| `find_datasets_by_topic` | `max_publications` | Minimum 5 |
| `build_subgraph_from_dois` | `dois` | 2-100 DOIs |
| `discover_by_subject` / `discover_by_coauthors` | `dois` | 1-10 DOIs |
| `get_citation_network` | `depth` | 1-3 (depth 2+ can be slow) |
| `analyze_coauthorship_network` | `max_depth` | 1 or 2 only |

## Common Sort Options

`search_research_products`: `relevance`, `publicationDate`, `citationCount`, `influence`, `popularity`, `impulse` — each with `ASC` or `DESC`.

`search_projects`: `relevance`, `startDate`, `endDate`.

Most other search tools: `relevance` only.
