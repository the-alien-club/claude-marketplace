---
name: explore-openscience
description: Open Science Research Toolkit — strategy guide for searching across OpenAIRE, bioRxiv, medRxiv, and web search. Load this first for any research question.
---

# Open Science Research Toolkit

You have access to multiple complementary research sources. Each has different strengths — understand what they offer, start where it makes sense, and broaden if your initial results are insufficient.

## What Each Source Offers

**OpenAIRE** — Structured metadata graph covering all fields of science: 600M+ research products with titles, abstracts, authors, citations, bibliometric indicators, projects, and cross-product relationships. Strong for discovery, impact assessment, and navigating the scholarly record. No full text.

**bioRxiv / medRxiv** — Full-text preprint document stores with keyword and semantic search. Use when you need what's actually written in a paper — methods, experimental details, results, discussion. Load the per-source skill (`/alien-openscience:explore-biorxiv`, `/alien-openscience:explore-medrxiv`) for dataset details and search strategies.

**WebSearch** — Broadest but least structured. Complements MCP sources by reaching content they don't index — specific URLs, institutional pages, very recent work, or domains not yet covered by the data clusters.

## How to Think About Source Selection

No rigid routing — use your judgement based on what the question needs:

- **Looking up a specific paper by title or DOI?** OpenAIRE is structured for this.
- **Need the actual content of a paper?** Dataclusters have full text. Try them.
- **Assessing impact or tracing citations?** OpenAIRE has bibliometrics and citation networks.
- **Exploring a topic broadly?** Start with whichever source feels most natural, broaden if results are thin.

The key insight: **no single source is reliably sufficient**. OpenAIRE has metadata but not full text. Dataclusters have full text but not everything. When your first source doesn't give you enough, try another — don't keep refining the same query in the same source.

## Broadening When Results Are Thin

If your initial search returns sparse, irrelevant, or no results:

1. **Try a different source** — metadata search and full-text search surface different things. A paper missed by one may be found by the other.
2. **Try a different search mode** — keyword search is precise but brittle; vector/semantic search finds conceptually related content even with different terminology.
3. **Search multiple sources in parallel** when the topic is broad or unfamiliar. This is efficient when warranted, not a default for every query.
4. **WebSearch** fills gaps — if MCP sources don't have what you need, web search likely will. It also complements MCP results with additional context (published versions, institutional pages, news coverage).

## Combining Results Across Sources

When you get results from multiple sources:

- Deduplicate — the same paper may appear across sources
- Enrich — add citation data from OpenAIRE to papers found via full-text search, or read full text of high-impact papers discovered via metadata
- Present the strongest evidence regardless of which source it came from

## Context Budget

| Operation | Size | Action |
|-----------|------|--------|
| Any search with limit=5-10 | 500-2000 chars | Safe in main context |
| `list_datasets` | Small with `response_format="markdown"` | Safe. Never use `include_schema=true` (120K+) |
| OpenAIRE `get_project_outputs` | 30K+ chars | Always delegate to subagent |
| OpenAIRE `analyze_coauthorship_network` | 15K+ chars | Always delegate to subagent |
| Datacluster `get_entry_content` (full paper) | 10K-100K chars | Paginate with `char_offset`/`char_limit`, or delegate |
| OpenAIRE `get_author_profile` (limit>30) | Scales linearly | Delegate to subagent |

## Per-Source Skills

For detailed tool routing, error patterns, and search strategies:

- `/alien-openscience:explore-openaire` — OpenAIRE Research Graph (metadata, citations, bibliometrics)
- `/alien-openscience:explore-biorxiv` — bioRxiv preprint data cluster (biology full-text)
- `/alien-openscience:explore-medrxiv` — medRxiv preprint data cluster (medical/health full-text)

Load the relevant skill when going deep with one source. This overview skill is sufficient for most research tasks.
