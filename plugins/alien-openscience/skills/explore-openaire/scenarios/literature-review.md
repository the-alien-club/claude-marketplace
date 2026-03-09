# Literature Review

Conduct a structured literature review on a research topic using OpenAIRE.

## Domain Context

- OpenAIRE covers 600M+ research products across all disciplines.
- Bibliometric classes help identify landmark papers (influence C1-C2) and currently trending work (popularity C1-C2).
- FOS classification enables field-specific filtering (use full code+label, e.g. `0102 computer and information sciences`).
- OpenAIRE's relationship data reveals citation patterns and intellectual lineage.
- Author networks surface research groups active in a field.
- Multiple search approaches (keyword, bibliometric filter, citation traversal) provide complementary coverage.

## Workflow

### Step 1: Find seed papers

```
openaire_search_research_products(
  query="<topic>", type=["publication"],
  sort_by="citationCount DESC", page_size=5, detail="minimal"
)
```

### Step 2: Find landmark papers in the field

```
openaire_find_by_influence_class(
  influence_class="C1", query="<topic>",
  type=["publication"], page_size=5
)
```

Optionally check trending work with `find_by_popularity_class(popularity_class="C1")`.

### Step 3: Trace citation lineage from seeds

```
openaire_explore_research_relationships(
  target_pid="<seed_doi>",
  relationship_type="cites",
  page_size=20
)
```

Always use `target_pid` (not `doi`) for incoming citations. The `doi` parameter finds outgoing references, which are sparsely indexed.

### Step 4: Deep-dive on promising papers

- If 1-2 DOIs: call `get_research_product_details` directly.
- If 3+ DOIs: **MUST** use the Task tool to launch parallel foreground subagents (multiple Task calls in a single message).

### Step 5 (optional): Explore author networks

Search for more work by key authors:

```
openaire_search_research_products(
  author_full_name=["<author>"],
  sort_by="citationCount DESC", page_size=5
)
```

### Step 6 (optional): Publication trend

```
openaire_analyze_research_trends(
  search="<topic>", from_year=2015, to_year=2025
)
```

Always small output (~600 chars). Safe in main context.

## Parallel Search Strategy

When exploring a topic across multiple dimensions, run parallel searches:

```
# All three in parallel — no dependencies
search_research_products(query="<topic>", type=["publication"], sort_by="citationCount DESC", page_size=5)
search_research_products(query="<topic>", type=["dataset"], page_size=5)
search_research_products(query="<topic>", type=["software"], page_size=5)
```
