---
name: explore-medrxiv
description: medRxiv Preprint Data Cluster Exploration Skill. Use when searching for preprint full-text content in clinical medicine, epidemiology, public health, and health sciences via the medRxiv MCP server.
---

# medRxiv Preprint Data Cluster Exploration Skill

You are searching and reading preprint manuscripts through the medRxiv data cluster. This server provides full-text access to medical and health sciences preprints organized by clinical speciality, with both keyword search (typo-tolerant, ranked) and semantic vector search (meaning-based similarity).

## When to Use medRxiv vs OpenAIRE

- **medRxiv**: Full-text search over clinical/health preprint manuscripts. Use when you need actual paper content — clinical trial details, epidemiological methods, patient cohort descriptions, statistical analyses, or health policy evidence buried in the body text.
- **OpenAIRE**: Structured metadata search (titles, abstracts, citations, authors, projects, bibliometrics). Use for discovery, citation analysis, impact assessment, or finding published (peer-reviewed) work.
- **Combine both**: Use OpenAIRE to identify influential studies by citation/impact, then medRxiv to read their full preprint content for clinical detail.

## Available Datasets

medRxiv datasets are organized by clinical speciality (~50 categories). Largest collections:
- Infectious Diseases (11,744), Epidemiology (10,367), Public and Global Health (6,627)
- Neurology (4,512), Genetic and Genomic Medicine (4,393), Psychiatry (3,725)
- Cardiovascular Medicine (3,129), Health Informatics (2,779), Oncology (2,243)

Smaller but well-covered: Pediatrics, Ophthalmology, Surgery, Respiratory Medicine, Endocrinology, Gastroenterology, many more.

Use `datacluster_list_datasets` to see all available specialities with entry counts.

## Tool Routing

| Goal | Tool | Notes |
|------|------|-------|
| See available specialities | `datacluster_list_datasets` | Returns categories with entry counts |
| Understand a dataset's structure | `datacluster_get_dataset` | Schema, fields, entry count |
| Find papers by keywords | `datacluster_keyword_search` | Typo-tolerant, ranked, <50ms |
| Find papers by meaning | `datacluster_vector_search_chunks` | Semantic similarity, snippet-level, <100ms |
| Read a paper's full text | `datacluster_get_entry_content` | Paginate if >50K chars |
| List files for an entry | `datacluster_get_entry_documents` | Original PDFs, processed text |

## Context Budget Awareness

### Safe in main context
- `list_datasets` — always use `response_format="markdown"`. Never use `include_schema=true` (120K+ chars in JSON).
- `keyword_search` (limit=10) — moderate output with snippets. Use `response_format="markdown"`.
- `vector_search_chunks` (limit=10) — moderate output with chunks
- `get_dataset` — use `response_format="markdown"`

### Paginate or delegate
- `get_entry_content` on long papers — use `char_offset` and `char_limit` to paginate. Never request full content of documents >50K chars in one call.

## Search Strategy

### Keyword search
Best for: clinical terms, drug names, trial identifiers, ICD codes, specific conditions.

```
datacluster_keyword_search(
  query="COVID-19 vaccine effectiveness booster dose",
  limit=10, context_mode="smart"
)
```

Filter by speciality:
```
datacluster_keyword_search(
  query="GLP-1 receptor agonist cardiovascular outcomes",
  dataset_ids=[<cardiovascular_medicine_id>],
  limit=10
)
```

### Vector/semantic search
Best for: clinical questions phrased naturally, finding studies about a condition without knowing exact MeSH terms.

```
datacluster_vector_search_chunks(
  query="long-term neurological effects after mild COVID infection in children",
  limit=10, score_threshold=0.7
)
```

### Combined strategy
1. Start with vector search for broad clinical question discovery
2. Refine with keyword search using specific clinical terms from initial results
3. Read full content of the most relevant entries for methodology and outcomes

## Parallel Search

datacluster tools are stateless — run searches in parallel across different specialities or angles:

```
# All three in parallel
datacluster_keyword_search(query="long COVID neurological", dataset_ids=[<neurology_id>], limit=5)
datacluster_keyword_search(query="post-COVID syndrome cognitive", dataset_ids=[<psychiatry_id>], limit=5)
datacluster_vector_search_chunks(query="persistent symptoms after SARS-CoV-2 infection affecting brain function", limit=5)
```

## Output Presentation

- Search results: numbered list with entry IDs, titles, relevance scores, and key snippets
- Full-text content: summarize key sections (background, methods, results, conclusions) rather than reproducing verbatim
- Clinical findings: highlight study design, sample size, key outcomes, and confidence intervals when available
- When results include DOIs: format as clickable links
