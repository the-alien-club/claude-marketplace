# Co-Citation Analysis

Analyze co-citation patterns to reveal the intellectual structure of a field.

## Domain Context

- Co-citation analysis identifies papers frequently cited together, revealing intellectual structure of a field.
- OpenAIRE's ScholeXplorer indexes citation relationships but coverage is partial.
- Incoming citations (via `target_pid`) are much better indexed than outgoing references (via `doi`).
- Citation count classes (C1-C5) provide field-normalized impact assessment.
- C1 = top 0.01% (landmark), C2 = top 0.1%, C3 = top 1%, C4 = top 10%, C5 = bottom 90%.

## Workflow

### Step 1: Identify seed papers

If the user provides DOIs, use them directly. Otherwise, find seeds:

```
openaire_search_research_products(
  query="<field topic>", type=["publication"],
  sort_by="citationCount DESC", page_size=5, detail="minimal"
)
```

### Step 2: Get citation networks for each seed

For each seed DOI, trace who cites it:

```
openaire_explore_research_relationships(
  target_pid="<seed_doi>",
  relationship_type="cites",
  page_size=20
)
```

Run these in parallel for multiple seeds.

### Step 3: Identify co-cited papers

Look for DOIs that appear in the citation lists of multiple seeds. These are the co-cited papers — the intellectual foundations of the field.

### Step 4: Assess significance of co-cited papers

For the most frequently co-cited DOIs:

- If 1-2 DOIs: call `get_research_product_details` directly.
- If 3+ DOIs: launch parallel subagents via Task tool.

Check bibliometric classes to assess significance:
- C1 across all metrics = unambiguous landmark
- C1 influence + C5 impulse = foundational but not recently trending
- C5 influence + C1 popularity = currently hot but not yet established

### Step 5 (optional): Build citation subgraph

```
openaire_get_citation_network(
  identifier="<most_cited_seed>",
  direction="both", depth=1,
  max_nodes=50
)
```

Note: `build_subgraph_from_dois` often returns 0 edges due to incomplete ScholeXplorer data. Prefer `get_citation_network` on individual DOIs.
