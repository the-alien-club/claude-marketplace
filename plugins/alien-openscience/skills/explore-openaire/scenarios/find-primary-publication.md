# Find Primary Publication

Find the primary/canonical publication for a software tool, method, or dataset.

## Domain Context

- Many software tools, methods, and techniques have a primary publication that should be cited.
- OpenAIRE indexes relationships between software, datasets, and publications.
- The most-cited publication matching an artifact name is often the primary one.
- Type filters (publication, software, dataset) and relationship traversal between entities help trace the connection.

## Workflow

### Step 1: Search for the artifact by name

```
openaire_search_research_products(
  query="<artifact name>",
  type=["publication"],
  sort_by="citationCount DESC",
  page_size=5, detail="minimal"
)
```

The most-cited result is often the primary publication.

### Step 2 (if software/dataset entry exists): Find the software/dataset record

```
openaire_search_research_products(
  query="<artifact name>",
  type=["software"],
  page_size=5
)
```

### Step 3: Trace relationships from the software/dataset to publications

```
openaire_explore_research_relationships(
  doi="<software_or_dataset_doi>",
  target_type="publication",
  page_size=20
)
```

Look for relationship types like `isDocumentedBy`, `isSupplementTo`, or `references`.

### Step 4: Verify the primary publication

```
openaire_get_research_product_details(identifier="<candidate_doi>")
```

Check citation count and bibliometric classes. The primary publication typically has:
- Highest citation count among candidates
- High influence class (C1-C3)
- Title or abstract mentioning the artifact name
