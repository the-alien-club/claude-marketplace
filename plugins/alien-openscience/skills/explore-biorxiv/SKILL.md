---
name: explore-biorxiv
description: bioRxiv Preprint Data Cluster Exploration Skill. Use when searching for preprint full-text content in biology, neuroscience, genomics, bioinformatics, and related life sciences via the bioRxiv MCP server.
---

# bioRxiv Preprint Data Cluster Exploration Skill

You are searching and reading preprint manuscripts through the bioRxiv data cluster. This server provides full-text access to biology preprints organized by subject area, with both keyword search (typo-tolerant, ranked) and semantic vector search (meaning-based similarity).

## When to Use bioRxiv vs OpenAIRE

- **bioRxiv**: Full-text search over preprint manuscripts. Use when you need the actual content of papers, specific methods sections, experimental details, or when the answer is buried in the body text — not just metadata.
- **OpenAIRE**: Structured metadata search (titles, abstracts, citations, authors, projects, bibliometrics). Use for discovery, citation analysis, impact assessment, or finding published (peer-reviewed) work.
- **Combine both**: Use OpenAIRE to identify key papers by citation/impact, then bioRxiv to read their full preprint content for methodological detail.

## Available Datasets

bioRxiv datasets are organized by subject area (~50 categories, up to 1000 entries each). Examples:
- Neuroscience, Genomics, Bioinformatics, Cell Biology, Molecular Biology
- Immunology, Microbiology, Ecology, Evolutionary Biology
- Cancer Biology, Developmental Biology, Pharmacology

Use `datacluster_list_datasets` to see all available categories.

## Tool Routing

| Goal | Tool | Notes |
|------|------|-------|
| See available subject areas | `datacluster_list_datasets` | Returns categories with entry counts |
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
Best for: exact terms, known titles, author names, specific methods or gene names.

```
datacluster_keyword_search(
  query="CRISPR-Cas9 gene editing efficiency",
  limit=10, context_mode="smart"
)
```

Filter by dataset to scope to a subject area:
```
datacluster_keyword_search(
  query="single-cell RNA sequencing",
  dataset_ids=[<genomics_dataset_id>],
  limit=10
)
```

### Vector/semantic search
Best for: conceptual queries, finding papers about a technique without knowing exact terminology.

```
datacluster_vector_search_chunks(
  query="methods for measuring protein-protein interactions in living cells",
  limit=10, score_threshold=0.7
)
```

### Combined strategy
1. Start with vector search for broad discovery
2. Refine with keyword search using specific terms from initial results
3. Read full content of the most relevant entries

## Parallel Search

datacluster tools are stateless — run searches in parallel across different angles:

```
# All three in parallel
datacluster_keyword_search(query="optogenetics cortical circuits", limit=5)
datacluster_vector_search_chunks(query="using light to control neural activity in cortex", limit=5)
datacluster_keyword_search(query="channelrhodopsin expression mouse", dataset_ids=[<neuroscience_id>], limit=5)
```

## Output Presentation

- Search results: numbered list with entry IDs, titles, relevance scores, and key snippets
- Full-text content: summarize key sections (abstract, methods, results, conclusions) rather than reproducing verbatim
- When results include DOIs: format as clickable links
