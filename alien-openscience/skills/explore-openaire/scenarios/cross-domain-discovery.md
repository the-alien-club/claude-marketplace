# Cross-Domain Discovery

Find methods, techniques, or data from outside a researcher's home field.

## Domain Context

- Cross-domain work often surfaces through citation networks that bridge fields. Direct keyword searches in unfamiliar domains may miss terminology differences.
- OpenAIRE indexes 600M+ research products across all disciplines with FOS (Fields of Science) classification.
- Citation-based discovery naturally bridges terminology gaps between fields.
- Dropping domain-specific terms from search queries helps find methodology in its native field.

## Workflow

### Step 1: Find seed in user's domain

```
openaire_search_research_products(
  query="<domain-specific terms>",
  type=["publication"], sort_by="citationCount DESC", page_size=5
)
```

### Step 2: Build citation network from seed

```
openaire_get_citation_network(
  identifier="<seed_doi>",
  direction="both", depth=1,
  max_nodes=50
)
```

Output at max_nodes=50 is ~1500 chars (safe in main context). If max_nodes>100, delegate to a subagent.

### Step 3: Search the cross-domain methodology

Deliberately drop domain-specific terms to find the methodology in its native field:

```
openaire_search_research_products(
  query="<methodology terms WITHOUT domain terms>",
  type=["publication"], sort_by="citationCount DESC", page_size=5
)
```

### Step 4 (optional): Trace adoption pattern

See who else is citing the cross-domain paper:

```
openaire_explore_research_relationships(
  target_pid="<cross_domain_doi>",
  relationship_type="cites",
  page_size=20
)
```

### Step 5 (optional): Use FOS to explore a specific field

```
openaire_search_research_products(
  query="<broad methodology term>",
  fos=["0102 computer and information sciences"],
  type=["publication"], sort_by="citationCount DESC"
)
```

FOS works for publications but not datasets.
