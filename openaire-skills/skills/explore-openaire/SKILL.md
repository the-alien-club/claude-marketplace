---
name: explore-openaire
description: OpenAIRE Research Graph Exploration Skill. Use when exploring research metadata, finding papers, analyzing citations, discovering datasets, or assessing research impact via the OpenAIRE MCP server.
---

# OpenAIRE Research Graph Exploration Skill

You are exploring research metadata through the OpenAIRE MCP server. This server exposes ~30 tools across four API layers: Graph API v1 (organizations, projects, persons, data sources), Graph API v2 (research products, bibliometric filters, datasets), ScholeXplorer (cross-product relationships), and composite/analytical tools (citation networks, author profiles, trend analysis). Follow these practices to manage context efficiently and produce useful results.

## Tuning Variables

Calibrated from empirical testing against live OpenAIRE APIs (February 2026). Adjust if API behavior changes.

| Variable | Value | Rationale |
|----------|-------|-----------|
| `SEARCH_LIMIT` | `5` | Returns enough results to judge relevance without flooding context. Increase to 10-20 for broad surveys. |
| `SEARCH_DETAIL` | `"minimal"` | Minimal detail returns ~300-500 chars per call. Use `"standard"` when you need abstracts/subjects. Use `"full"` only for single-item deep dives. |
| `BIBLIOMETRIC_LIMIT` | `5` | For `find_by_*_class` tools. These return pre-filtered high-quality results; 5 suffices for landmark paper discovery. |
| `RELATIONSHIP_PAGE_SIZE` | `20` | ScholeXplorer default. Increase to 50 for comprehensive citation traversal. Max effective: 100. |
| `CITATION_NETWORK_MAX_NODES` | `50` | Caps `get_citation_network` output. At depth=1 this yields ~1500 chars. Default 200 is a context bomb (~6K+ chars). |
| `AUTHOR_PROFILE_LIMIT` | `20` | Caps publications in `get_author_profile`. Default 100 yields ~4K chars; prolific authors at 500 yield 15K+. |
| `COAUTHORSHIP_LIMIT` | `30` | Caps `analyze_coauthorship_network`. Default 100 with depth=1 yields ~15K chars (64 nodes for prolific authors). |
| `PROJECT_OUTPUT_LIMIT` | `10` | Caps `get_project_outputs`. Default 100 yields 30K+ chars. Always override. |
| `TREND_YEAR_SPAN` | `10` | For `analyze_research_trends`. Yields ~600 chars. Wider spans are fine — output scales linearly with years. |
| `PARALLEL_READ_THRESHOLD` | `3` | Number of product details at which parallel subagent delegation becomes worthwhile. |
| `SORT_DEFAULT` | `"citationCount DESC"` | Most common sort for publication searches. Used in 8/9 oracle scenarios. |

## Tool Taxonomy

### By Output Size (empirically measured, json format, default params)

| Size | Chars | Tools |
|------|-------|-------|
| **TINY** | 200-500 | `get_organization`, `get_data_source`, search_* with `detail="minimal"` + `page_size=3` |
| **SMALL** | 500-800 | `get_person`, `get_project`, `search_datasets` (3 results), `find_by_*_class` (3 results), `analyze_research_trends` |
| **MEDIUM** | 800-2500 | `explore_research_relationships`, `get_research_links`, `discover_by_subject`, `get_research_product_details`, `search_research_products` (standard detail, 5 results) |
| **LARGE** | 2500-4000 | `discover_by_coauthors`, `get_author_profile` (limit=20), `get_citation_network` (max_nodes=50) |
| **CONTEXT BOMB** | 15K-30K+ | `get_project_outputs` (default), `analyze_coauthorship_network` (prolific author), `get_author_profile` (limit=100+) |

### By API Layer

**Graph API v1** (entity metadata — organizations, projects, persons, data sources):
- `openaire_search_organizations`, `openaire_get_organization`
- `openaire_search_projects`, `openaire_get_project`
- `openaire_search_persons`, `openaire_get_person`
- `openaire_search_data_sources`, `openaire_get_data_source`
- `openaire_get_research_links` (native Graph relationship data)
- `openaire_get_relationship_types` (reference: 19 relationship types)

**Graph API v2** (research products + bibliometric filters):
- `openaire_search_research_products` (universal search, most-used tool)
- `openaire_get_research_product_details` (single-item deep dive)
- `openaire_search_datasets` (dedicated dataset search — note: uses `datasets[]` not `results[]`)
- `openaire_find_by_influence_class` (field-normalized citation impact)
- `openaire_find_by_popularity_class` (recent citation velocity)
- `openaire_find_by_impulse_class` (early citation momentum)
- `openaire_find_by_citation_count_class` (absolute citation count)

**ScholeXplorer** (cross-product relationships):
- `openaire_explore_research_relationships` (citation/supplement/version traversal)

**Composite/Analytical** (multi-step pipelines):
- `openaire_get_citation_network` (graph from seed DOI)
- `openaire_build_subgraph_from_dois` (relationship graph between DOI set)
- `openaire_discover_by_subject` (subject-similarity expansion)
- `openaire_discover_by_coauthors` (author-network expansion)
- `openaire_find_datasets_by_topic` (publication→dataset linking)
- `openaire_get_author_profile` (publications + co-author analysis)
- `openaire_analyze_coauthorship_network` (collaboration graph)
- `openaire_analyze_research_trends` (yearly publication counts)
- `openaire_get_project_outputs` (all outputs of a funded project)

## Context Management Rules

1. **Cap all composite tool parameters.** The default limits on composite tools are sized for web UIs, not LLM context windows. Always override: `max_nodes=CITATION_NETWORK_MAX_NODES`, `limit=AUTHOR_PROFILE_LIMIT`, `page_size=PROJECT_OUTPUT_LIMIT`.

2. **Delegate context bombs to subagents.** If you need full output from `get_project_outputs`, `analyze_coauthorship_network`, or `get_author_profile` (limit>50), launch a subagent to call the tool and summarize. These can exceed 30K chars. Subagents have full MCP tool access. **Note:** If running subagents in background (`run_in_background=true`), MCP tools must already be approved in the permission settings — background agents cannot surface interactive permission prompts. Run subagents in the foreground if MCP permissions haven't been pre-approved.

3. **Use `detail="minimal"` for routing, `"standard"` for assessment.** Minimal returns id+title+type (~100 chars/result). Standard adds abstract, subjects, metrics (~500 chars/result). Full adds all fields (~800 chars/result). The difference between standard and full is rarely worth the context cost.

4. **Use `response_format="json"` throughout.** Despite the DataCluster MCP where markdown saves space, OpenAIRE markdown and json are nearly identical in size. JSON is more reliable for extracting DOIs and IDs for follow-up calls.

5. **Prefer `page_size=5` for exploratory searches.** You can always paginate for more. Starting at 5 keeps initial responses under 2500 chars even with standard detail.

6. **Batch parallel searches when exploring multiple facets.** The OpenAIRE tools are stateless — multiple search calls can run in parallel without conflicts. Use this for multi-query strategies (e.g., searching the same topic across publications, datasets, and software simultaneously).

## Workflow Patterns

### Pattern 1: Search → Explore → Detail (Universal)

The most common pattern, used in 8 of 9 oracle scenarios. Start broad, narrow via relationships, then deep-dive.

```
Step 1: Find seed papers
  openaire_search_research_products(
    query="<topic>", type=["publication"],
    sort_by="citationCount DESC", page_size=SEARCH_LIMIT,
    detail="minimal"
  )

Step 2: Explore relationships from seed
  openaire_explore_research_relationships(
    target_pid="<seed_doi>",       # Use target_pid for INCOMING citations
    relationship_type="cites",      # "cites" = papers that cite the seed
    page_size=RELATIONSHIP_PAGE_SIZE
  )

Step 3: Deep-dive on promising results
  openaire_get_research_product_details(identifier="<doi>")
```

**Key:** Always use `target_pid` (not `doi`) with `relationship_type="cites"` for citation analysis. The `doi` parameter finds outgoing references, which are sparsely indexed.

### Pattern 2: Bibliometric Discovery (Landmark Papers)

For finding influential/trending/highly-cited papers in a field. The simplest pattern — often a single tool call suffices.

```
Step 1: Find landmark papers
  openaire_find_by_influence_class(
    influence_class="C1",           # Top 0.01%
    query="<topic>",
    type=["publication"],
    page_size=BIBLIOMETRIC_LIMIT
  )

Step 2 (optional): Check trending work
  openaire_find_by_popularity_class(
    popularity_class="C1",
    query="<topic>"
  )

Step 3 (optional): Deep-dive
  openaire_get_research_product_details(identifier="<doi>")
```

**Choosing the right bibliometric tool:**
| Tool | Measures | Use when |
|------|----------|----------|
| `find_by_influence_class` | Field-normalized citation impact (time-adjusted) | Finding foundational/seminal papers |
| `find_by_popularity_class` | Recent citation velocity | Finding currently trending research |
| `find_by_impulse_class` | Early citation momentum post-publication | Finding papers that made immediate impact |
| `find_by_citation_count_class` | Absolute citation count (raw) | Finding most-cited regardless of field/age |

### Pattern 3: Cross-Domain Discovery

When a researcher needs to find methods/data from outside their field.

```
Step 1: Find seed in user's domain
  openaire_search_research_products(
    query="<domain-specific terms>",
    type=["publication"], sort_by="citationCount DESC"
  )

Step 2: Build citation network from seed
  openaire_get_citation_network(
    identifier="<seed_doi>",
    direction="both", depth=1,
    max_nodes=CITATION_NETWORK_MAX_NODES
  )

Step 3: Search the cross-domain methodology
  openaire_search_research_products(
    query="<methodology terms without domain terms>",
    type=["publication"], sort_by="citationCount DESC"
  )

Step 4 (optional): Trace adoption pattern
  openaire_explore_research_relationships(
    target_pid="<cross_domain_doi>",
    relationship_type="cites"
  )
```

**Why this works:** Citation networks naturally bridge domains. Step 3 deliberately drops domain-specific terms to find the methodology in its native field.

### Pattern 4: Dataset Discovery

Finding research datasets is the weakest area of OpenAIRE. Use this cascading strategy:

```
Step 1: Try direct dataset search
  openaire_search_research_products(
    query="<topic>", type=["dataset"],
    sort_by="citationCount DESC", page_size=SEARCH_LIMIT
  )

Step 2: Try dedicated dataset search
  openaire_search_datasets(search="<topic>", page_size=SEARCH_LIMIT)

Step 3 (if Steps 1-2 insufficient): Try topic-based linking
  openaire_find_datasets_by_topic(
    query="<topic>",
    max_publications=10,        # min allowed: 5
    sort_publications_by="citationCount"
  )
  # WARNING: Almost always returns 0 datasets. ScholeXplorer pub→dataset links are sparse.

Step 4 (fallback): Search for data papers
  openaire_search_research_products(
    query="<topic> dataset",
    type=["publication"],
    instance_type=["Data Paper"],
    sort_by="citationCount DESC"
  )
```

**Known limitation:** FOS filters work for publications but produce unreliable results for datasets. Subject metadata on datasets is often empty. The `subjects` parameter on `search_datasets` has no practical effect.

### Pattern 5: Author/Collaboration Analysis

```
Step 1: Find author's work
  openaire_get_author_profile(
    author_name="<full name>",     # Prefer name over ORCID (better coverage)
    limit=AUTHOR_PROFILE_LIMIT,
    include_coauthors=true
  )

Step 2 (if needed): Build collaboration graph — DELEGATE TO SUBAGENT
  # This is a context bomb. Launch subagent:
  openaire_analyze_coauthorship_network(
    author_name="<full name>",
    max_depth=1,                   # depth=2 explodes exponentially
    limit=COAUTHORSHIP_LIMIT,
    min_collaborations=2           # Filter noise
  )
```

**Always use `author_name` over `orcid`.** ORCID-based searches frequently return empty results even for valid ORCIDs. Name search has significantly better coverage.

### Pattern 6: Project Impact Assessment

```
Step 1: Find the project
  openaire_search_projects(
    search="<project name or topic>",
    funding_short_name=["EC"],      # or "NSF", "NIH" etc.
    page_size=SEARCH_LIMIT
  )

Step 2: Get project outputs — DELEGATE TO SUBAGENT
  # Context bomb at default page_size=100
  openaire_get_project_outputs(
    project_code="<grant_code>",
    page_size=PROJECT_OUTPUT_LIMIT
  )

Step 3: Assess key outputs
  openaire_get_research_product_details(identifier="<output_doi>")
```

### Pattern 7: Research Trend Analysis

```
openaire_analyze_research_trends(
  search="<topic>",
  from_year=<start>,
  to_year=<end>,
  product_type="publication"    # or "dataset", "software", "all"
)
```

Output is always small (~600 chars). Returns yearly counts, peak year, growth statistics. Safe to call in main context.

## Tool Composition Chains

### Chains That Work Well

| Chain | Use Case | Context Cost |
|-------|----------|-------------|
| `search_research_products` → `get_research_product_details` | Find then inspect | ~800 + ~1500 chars |
| `search_research_products` → `explore_research_relationships(target_pid=)` | Find then trace citations | ~800 + ~2500 chars |
| `find_by_influence_class` → `get_research_product_details` | Landmark paper deep-dive | ~500 + ~1500 chars |
| `search_datasets` → `get_research_product_details` | Dataset metadata enrichment | ~800 + ~1500 chars |
| `search_projects` → `get_project_outputs(page_size=10)` | Project impact quick-look | ~450 + ~3000 chars |
| `analyze_research_trends` (standalone) | Publication volume over time | ~600 chars total |
| `get_relationship_types` (standalone) | Reference for valid relationship names | ~2500 chars (call once, reuse) |

### Chains That Need Workarounds

| Chain | Problem | Workaround |
|-------|---------|-----------|
| `explore_research_relationships(doi=, relationship_type="references")` | Outgoing references sparsely indexed | Use `target_pid=<doi>` + `relationship_type="cites"` instead |
| `discover_by_subject` on highly-cited papers | SDG tags cause irrelevant matches ("16. Peace & justice" on ML papers) | Works better on domain-specific papers (EEG, genomics). For landmark papers, use `search_research_products` with methodology keywords instead |
| `find_datasets_by_topic` | Almost always returns 0 datasets | Use `search_research_products(type=["dataset"])` or `search_datasets` directly |
| `build_subgraph_from_dois` | Returns 0 edges for obviously related papers (BERT→Transformer) | ScholeXplorer citation data incomplete. Use `get_citation_network` on each DOI separately |
| `explore_research_relationships` + `source_subtype`/`target_subtype` | Filter is broken — removes ALL results | Do not use these parameters. Filter publication types client-side after retrieval |

## Iterative Querying Strategies

### Strategy 1: Faceted Parallel Search

When exploring a topic across multiple dimensions, run parallel searches:

```python
# All three calls in parallel — no dependencies
search_research_products(query="transformer attention", type=["publication"], sort_by="citationCount DESC", page_size=5)
search_research_products(query="transformer attention", type=["dataset"], page_size=5)
search_research_products(query="transformer attention", type=["software"], page_size=5)
```

### Strategy 2: Citation Snowball

When a seed paper is highly relevant, expand in both directions:

```python
# Step 1: Who cites this? (incoming)
explore_research_relationships(target_pid="<doi>", relationship_type="cites", page_size=20)

# Step 2: What does it cite? (outgoing — less reliable but sometimes works)
get_citation_network(identifier="<doi>", direction="references", max_nodes=20)

# Step 3: Pick promising nodes, repeat step 1 on them
```

### Strategy 3: Bibliometric Triangulation

When assessing a paper's significance, check multiple metrics:

```python
# All four in parallel
get_research_product_details(identifier="<doi>")  # Gets influence, popularity, impulse, citation_class
# If the paper is C1 across all four → unambiguously landmark
# If C1 influence but C5 impulse → foundational but not recently trending
# If C5 influence but C1 popularity → currently hot but not yet established
```

### Strategy 4: Author Trail

When you find a relevant paper, explore the author's other work:

```python
# Step 1: Get paper details to extract author names
get_research_product_details(identifier="<doi>")

# Step 2: Search for more work by the first author
search_research_products(author_full_name=["<first_author>"], sort_by="citationCount DESC", page_size=5)

# Step 3 (if deeper needed): Full author profile via subagent
get_author_profile(author_name="<first_author>", limit=AUTHOR_PROFILE_LIMIT)
```

### Strategy 5: Filter Cascade for Datasets

Datasets have poor metadata. Try multiple approaches:

```python
# Attempt 1: Direct type filter (best coverage)
search_research_products(query="<topic>", type=["dataset"], page_size=10)

# Attempt 2: Dedicated endpoint (different index, sometimes surfaces different results)
search_datasets(search="<topic>", page_size=10)

# Attempt 3: Data papers (publications that describe datasets)
search_research_products(query="<topic> dataset", instance_type=["Data Paper"], page_size=10)
```

## Subagent Delegation Guide

Subagents have full access to OpenAIRE MCP tools. **Background subagents** (`run_in_background=true`) require MCP tool permissions to be pre-approved — they cannot prompt the user interactively. If permissions are not yet approved, run subagents in the foreground for the first invocation (which will trigger the approval prompt), then subsequent background runs will work.

| Task | Delegate? | Why |
|------|-----------|-----|
| `get_project_outputs` (full list) | **YES** | Default returns 100 items, 30K+ chars |
| `analyze_coauthorship_network` (any depth) | **YES** | Even depth=1 yields 15K+ for prolific authors |
| `get_author_profile` (limit>30) | **YES** | Scales linearly; 100 pubs = ~4K chars, 500 = ~20K |
| Reading PARALLEL_READ_THRESHOLD+ product details | **YES (parallel)** | Launch one subagent per paper for details + summary |
| `get_citation_network` (max_nodes>100) | **YES** | 200+ nodes produces 6K+ chars |
| Any search tool | **NO** | Responses are small and needed for routing decisions |
| `get_research_product_details` (single) | **NO** | ~1500 chars, manageable |
| `analyze_research_trends` | **NO** | Always small (~600 chars) |
| `find_by_*_class` tools | **NO** | Paginated, small per page |
| `explore_research_relationships` | **NO** | Paginated, medium per page |
| `get_relationship_types` | **NO** | Fixed size, call once |

### Subagent Template for Product Summaries

```
Launch subagent (one per paper, run in parallel for 3+ papers):
  "Use openaire_get_research_product_details to get metadata for DOI <doi>.
   Summarize in ~150 words: research question, methodology, key findings,
   significance. Report: title, authors, year, citation count, influence class,
   and your summary."
```

### Subagent Template for Project Output Analysis

```
Launch subagent:
  "Use openaire_get_project_outputs with project_code='<code>' and page_size=100.
   Summarize: total outputs by type, top 5 most-cited publications, any datasets
   or software produced. Report counts and key DOIs."
```

## Parameter Reference

### Critical "At Least One" Constraints

Several tools require at least one parameter from a set:

| Tool | Must provide at least one of |
|------|------------------------------|
| `explore_research_relationships` | `doi`, `target_pid`, `source_type`, `target_type`, `source_publisher`, `target_publisher` |
| `get_research_links` | `source_pid`, `target_pid`, `source_type`, `target_type`, `source_publisher`, `target_publisher` |
| `get_author_profile` | `author_name`, `orcid` |
| `analyze_coauthorship_network` | `author_name`, `orcid` |
| `get_project_outputs` | `project_id`, `project_code` |

### Validation Constraints

| Tool | Parameter | Constraint |
|------|-----------|-----------|
| `find_datasets_by_topic` | `max_publications` | Minimum 5 (will error if lower) |
| `build_subgraph_from_dois` | `dois` | 2-100 DOIs required |
| `discover_by_subject` | `dois` | 1-10 DOIs |
| `discover_by_coauthors` | `dois` | 1-10 DOIs |
| `get_citation_network` | `depth` | 1-3 (depth 2+ can be very slow) |
| `analyze_coauthorship_network` | `max_depth` | 1 or 2 only |

### Bibliometric Class Reference

| Class | Percentile | Typical Use |
|-------|-----------|-------------|
| C1 | Top 0.01% | Landmark/foundational papers (e.g., "Deep learning" Nature 2015: 60K+ citations) |
| C2 | Top 0.1% | Highly notable works |
| C3 | Top 1% | Very good, field-leading work |
| C4 | Top 10% | Above average |
| C5 | Bottom 90% | Average/below average |

### Common Sort Options

For `search_research_products`: `relevance`, `publicationDate`, `dateOfCollection`, `influence`, `popularity`, `citationCount`, `impulse` — each with `ASC` or `DESC`.

For `search_projects`: `relevance`, `startDate`, `endDate`.

Most other search tools: `relevance` only.

### FOS (Fields of Science) Codes

Top-level codes for the `fos` parameter on `search_research_products`:
- `"01 natural sciences"`, `"02 engineering and technology"`, `"03 medical and health sciences"`
- `"04 agricultural and veterinary sciences"`, `"05 social sciences"`, `"06 humanities and the arts"`

Common sub-codes: `"0102 computer and information sciences"`, `"0301 basic medicine"`, `"0502 economics and business"`, `"0106 biological sciences"`

**FOS works for publications. FOS combined with text queries on datasets returns 0.**

## Error Handling

| Error | Cause | Action |
|-------|-------|--------|
| `success: false`, "not found" | Invalid DOI or OpenAIRE ID | Re-search to find valid identifiers. Check ID format — e.g., OpenAIRE project IDs use `corda_______::` (7 underscores), not `corda__h2020::` |
| Empty results on `search_research_products` with FOS + query + `type=["dataset"]` | FOS metadata absent on datasets | Drop the `fos` parameter; use keyword query alone |
| 0 results from `explore_research_relationships(doi=, relationship_type="references")` | Outgoing references sparsely indexed | Switch to `target_pid=<doi>` with `relationship_type="cites"` |
| 0 results from `explore_research_relationships` with `source_subtype`/`target_subtype` | Filter is broken (client-side filter against missing data) | Remove subtype parameters entirely. Filter client-side after retrieval |
| 0 datasets from `find_datasets_by_topic` | ScholeXplorer pub→dataset links are sparse | Expected behavior. Use `search_research_products(type=["dataset"])` or `search_datasets` instead |
| 0 results from `discover_by_subject` | Seed paper has only generic/SDG subject tags | Use `search_research_products` with methodology keywords as workaround |
| 0 results from `get_author_profile(orcid=)` | ORCID coverage is sparse | Retry with `author_name` instead |
| 0 results from `search_persons` | Person index has very sparse coverage | Use `get_author_profile(author_name=)` instead, which searches via publications |
| `find_datasets_by_topic` validation error ">=5" | `max_publications` below minimum | Set `max_publications=5` or higher |
| Nonexistent DOI in `build_subgraph_from_dois` | Graceful degradation | Returns node with `title: "Metadata unavailable"`, 0 edges. Not an error. |
| Nonexistent author in `analyze_coauthorship_network` | Graceful degradation | Returns 1 isolated node, 0 edges, `publications_analyzed=0`. Not an error. |
| Future dates in results (e.g., `9999-01-01`) | Data quality issue in OpenAIRE | Filter client-side. Do not assume 0 results for far-future date queries. |

## Response Structure Differences

Most search tools return `results[]` in the response data. Two exceptions:

| Tool | Array Key | Example |
|------|-----------|---------|
| `search_datasets` | `datasets[]` | `response['data']['datasets']` |
| All other search tools | `results[]` | `response['data']['results']` |

Graph tools (`get_citation_network`, `build_subgraph_from_dois`, `analyze_coauthorship_network`) return `nodes[]` + `edges[]`.

## Anti-Patterns

- **Calling `get_project_outputs` or `analyze_coauthorship_network` in main context without capping limits** — these are context bombs. Always set `page_size=PROJECT_OUTPUT_LIMIT` or delegate to a subagent.
- **Using `doi=` param on `explore_research_relationships` for citation analysis** — outgoing references are sparse. Use `target_pid=` with `cites` instead.
- **Using `source_subtype` or `target_subtype` on `explore_research_relationships`** — broken filter, silently removes all results.
- **Using `orcid` instead of `author_name` on `get_author_profile`** — ORCID coverage is sparse; name search works much better.
- **Using `discover_by_subject` on landmark/highly-cited papers** — these often have noisy SDG subject tags that produce irrelevant results. Works well only on domain-specific papers with precise subject metadata.
- **Combining FOS filter + text query + `type=["dataset"]`** — returns 0. Drop FOS for dataset searches.
- **Using `search_persons` for author discovery** — person index is very sparse. Even famous researchers return 0. Use `get_author_profile(author_name=)` instead.
- **Relying on `find_datasets_by_topic` to find datasets** — the pub→dataset link in ScholeXplorer is too sparse. Use direct dataset search tools.
- **Calling `build_subgraph_from_dois` to check if papers cite each other** — ScholeXplorer citation data is incomplete; returns 0 edges for many obviously related papers. Use `get_citation_network` or `explore_research_relationships` on individual DOIs instead.
- **Setting `detail="full"` on multi-result searches** — marginal extra data vs significant context cost. Use `"minimal"` for routing, `"standard"` for assessment.
- **Using `search_research_products` with `instance_type` filter alongside free-text queries** — these are often incompatible and silently return fewer results than expected. Apply type filtering after retrieval when possible.
