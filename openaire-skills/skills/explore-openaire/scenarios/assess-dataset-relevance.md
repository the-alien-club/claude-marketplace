# Assess Dataset Relevance

Evaluate whether a dataset is suitable for a specific research purpose.

## Domain Context

- Dataset quality signals in OpenAIRE include: citation count, open access status, publisher/repository, publication date, and linking publications.
- Tracing which publications use a dataset reveals its adoption in peer-reviewed research.
- ScholeXplorer relationship types like `isSupplementTo` and `isReferencedBy` connect datasets to publications.
- Incoming citations (via `target_pid`) are better indexed than outgoing references.

## Workflow

### Step 1: Get dataset metadata

If you have a DOI:

```
openaire_get_research_product_details(identifier="<dataset_doi>")
```

If you have a name:

```
openaire_search_research_products(
  query="<dataset name>", type=["dataset"],
  page_size=5
)
```

Or use the dedicated endpoint:

```
openaire_search_datasets(search="<dataset name>", page_size=5)
```

Note: `search_datasets` returns `datasets[]` not `results[]`.

### Step 2: Find publications that use this dataset

```
openaire_explore_research_relationships(
  target_pid="<dataset_doi>",
  relationship_type="cites",
  page_size=20
)
```

If no results, try broader relationship types:

```
openaire_explore_research_relationships(
  target_pid="<dataset_doi>",
  page_size=20
)
```

Look for `isReferencedBy`, `isSupplementedBy`, `isCitedBy` relationships.

### Step 3: Assess citing publications

For the top publications that cite this dataset:

- Check if they are peer-reviewed and in relevant fields.
- If 3+ DOIs: use Task tool for parallel subagents.
- If 1-2 DOIs: call `get_research_product_details` directly.

### Step 4: Quality signals to report

Summarize:
- **Adoption:** How many publications cite/use this dataset?
- **Recency:** When was it last updated or cited?
- **Access:** Is it open access?
- **Repository:** Is it in a trusted repository (Zenodo, Figshare, domain-specific)?
- **Field match:** Are the citing publications in the user's research area?
