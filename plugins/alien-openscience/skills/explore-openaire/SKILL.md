---
name: explore-openaire
description: OpenAIRE Research Graph Exploration Skill. Use when exploring research metadata, finding papers, analyzing citations, discovering datasets, or assessing research impact via the OpenAIRE MCP server.
---

# OpenAIRE Research Graph Exploration Skill

OpenAIRE indexes 600M+ research products with structured metadata, citation counts, and bibliometric indicators. Its unique value over WebSearch is **bibliometric authority** — you can find the most influential papers in any field and cite them with quantified impact.

## Your Most Valuable Tools (Use These First)

### 1. `find_by_influence_class` — Start here for any research question

This is OpenAIRE's killer feature. It returns papers ranked by field-normalized, time-adjusted citation impact. **Use it as your first OpenAIRE call for every synthesis question.**

```
find_by_influence_class(
  influence_class="C3",    # Top 1% — good default for most fields
  query="<2-3 key terms>",
  page_size=10
)
```

Output: ~500 chars. Returns titles, DOIs, citation counts, influence scores. Immediately tells you "these are the landmark papers."

Use C2 (top 0.1%) for very broad fields. Use C3 (top 1%) as default. C1 (top 0.01%) is very restrictive — may return 0 results for niche topics.

### 2. `search_research_products` — Broader keyword search

Complement `find_by_influence_class` with a keyword search sorted by citations for broader coverage:

```
search_research_products(
  query="<2-4 key terms>",
  sort_by="citationCount DESC",
  page_size=10,
  detail="standard"
)
```

**Critical: OpenAIRE uses AND logic.** More terms = fewer results (opposite of Google). If you get 0 results, drop terms. `"ADC linker toxicity"` works; `"antibody drug conjugate cleavable linker stability toxicity potency"` returns 0.

### 3. `explore_research_relationships` — Trace citations

Find who cites a known paper:

```
explore_research_relationships(
  target_pid="<doi>",          # Use target_pid, NOT doi
  relationship_type="cites",
  page_size=20
)
```

Always use `target_pid` for incoming citations. The `doi` parameter finds outgoing refs, which are sparsely indexed.

## Secondary Tools (Use When Needed)

| Tool | Use when | Notes |
|------|----------|-------|
| `find_by_popularity_class` | Finding currently trending work | Recent citation velocity, not all-time impact |
| `find_by_impulse_class` | Finding papers with early momentum | Good for identifying rising work |
| `get_research_product_details` | Deep-dive on a specific DOI | Returns all 4 bibliometric classes (~1500 chars) |
| `analyze_research_trends` | Publication volume over time | Small output (~600 chars), always safe |
| `get_author_profile` | Researcher's publication history | Use `author_name=`, NOT `orcid=` (sparse) |
| `search_datasets` | Finding research datasets | Returns `datasets[]` not `results[]` |
| `get_citation_network` | Visualizing citation structure | Keep `max_nodes` < 100 in main context |

## Research Scenario Guides

For complex multi-step workflows, read the relevant scenario guide:

| Scenario | Local file |
|----------|------------|
| Literature review | `scenarios/literature-review.md` |
| Bibliometric assessment | `scenarios/bibliometric-assessment.md` |
| Find primary publication | `scenarios/find-primary-publication.md` |
| Co-citation analysis | `scenarios/co-citation-analysis.md` |
| Author landscape | `scenarios/author-landscape.md` |
| Cross-domain discovery | `scenarios/cross-domain-discovery.md` |
| Dataset discovery | `scenarios/dataset-discovery.md` |
| Project impact | `scenarios/project-impact.md` |
| Assess dataset relevance | `scenarios/assess-dataset-relevance.md` |

## Context Budget and Delegation

**Always delegate these to subagents** (context bombs):

| Tool | Why | Size |
|------|-----|------|
| `get_project_outputs` | 30K+ chars | Always delegate |
| `analyze_coauthorship_network` | 15K+ chars | Always delegate |
| `get_author_profile` (limit>30) | Scales linearly | Delegate |
| `get_research_product_details` for 3+ DOIs | Parallel subagents | One per DOI |

### Detail Level

- `detail="minimal"` for routing (~100 chars/result)
- `detail="standard"` for citing in answers (~500 chars/result) — **use this as default**
- `detail="full"` only for single-item deep dives (~800 chars/result)

## Bibliometric Classes Reference

| Class | Percentile | Meaning |
|-------|-----------|---------|
| C1 | Top 0.01% | Landmark/foundational |
| C2 | Top 0.1% | Highly notable |
| C3 | Top 1% | Field-leading |
| C4 | Top 10% | Above average |
| C5 | Bottom 90% | Average |

Four dimensions: **influence** (field-normalized, time-adjusted), **popularity** (recent velocity), **impulse** (early momentum), **citation_count** (absolute).

## FOS Codes

Use full label: `"01 natural sciences"`, `"02 engineering and technology"`, `"03 medical and health sciences"`, `"0102 computer and information sciences"`, `"0301 basic medicine"`, `"0106 biological sciences"`.

## Error Patterns

| Problem | Cause | Fix |
|---------|-------|-----|
| 0 results from search | Too many AND terms | Drop to 2-3 key terms |
| 0 from `explore_research_relationships(doi=)` | Outgoing refs sparse | Use `target_pid=` instead |
| 0 from `find_datasets_by_topic` | Sparse pub-dataset links | Use `search_research_products(type=["dataset"])` |
| 0 from `get_author_profile(orcid=)` | ORCID sparse | Use `author_name=` |
| 0 from `search_persons` | Person index very sparse | Use `get_author_profile(author_name=)` |

## Validation Constraints

| Tool | Parameter | Constraint |
|------|-----------|-----------|
| `find_datasets_by_topic` | `max_publications` | Minimum 5 |
| `build_subgraph_from_dois` | `dois` | 2-100 DOIs |
| `discover_by_subject` / `discover_by_coauthors` | `dois` | 1-10 DOIs |
| `get_citation_network` | `depth` | 1-3 (depth 2+ slow) |
| `analyze_coauthorship_network` | `max_depth` | 1 or 2 only |

## Sort Options

`search_research_products`: `relevance`, `publicationDate`, `citationCount`, `influence`, `popularity`, `impulse` — each with `ASC` or `DESC`.
