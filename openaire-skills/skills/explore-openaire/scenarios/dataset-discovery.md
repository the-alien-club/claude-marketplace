# Dataset Discovery

Find research datasets, trace datasets from papers, or find dataset users.

## Domain Context

- Dataset coverage in OpenAIRE is sparse. Subject metadata on datasets is often empty.
- FOS filters combined with text queries on datasets return 0 results.
- Publication-to-dataset links in ScholeXplorer are very sparse.
- Direct dataset search or searching for `type=["dataset"]` has better coverage than relationship traversal.
- Data papers (`instance_type=["Data Paper"]`) sometimes surface datasets that direct search misses.
- Multiple search approaches may be needed: direct type filter, dedicated dataset endpoint, and data paper search.

## Workflow: Find Datasets

Use a cascading strategy — dataset discovery is the weakest area of OpenAIRE.

### Step 1: Direct type filter (best coverage)

```
openaire_search_research_products(
  query="<topic>", type=["dataset"],
  sort_by="citationCount DESC", page_size=5
)
```

### Step 2: Dedicated dataset endpoint (different index)

```
openaire_search_datasets(search="<topic>", page_size=5)
```

Note: `search_datasets` returns `datasets[]` not `results[]`. The `subjects` parameter has no practical effect.

### Step 3 (fallback): Data papers

```
openaire_search_research_products(
  query="<topic> dataset",
  type=["publication"],
  instance_type=["Data Paper"],
  sort_by="citationCount DESC", page_size=5
)
```

### Step 4 (last resort): Topic-based linking

```
openaire_find_datasets_by_topic(
  query="<topic>",
  max_publications=10,
  sort_publications_by="citationCount"
)
```

**Warning:** Almost always returns 0 datasets. ScholeXplorer pub-to-dataset links are sparse. Use Steps 1-3 first.

## Workflow: Find Users of a Dataset

```
openaire_explore_research_relationships(
  target_pid="<dataset_doi>",
  relationship_type="cites",
  page_size=20
)
```

Use `target_pid` for incoming citations. If no results, try `relationship_type="isReferencedBy"` or `"isSupplementedBy"`.
