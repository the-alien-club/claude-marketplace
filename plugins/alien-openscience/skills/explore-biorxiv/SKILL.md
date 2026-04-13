---
name: explore-biorxiv
description: bioRxiv Preprint Data Cluster Exploration Skill. Use when searching for preprint full-text content in biology, neuroscience, genomics, bioinformatics, and related life sciences via the bioRxiv MCP server.
---

# bioRxiv Preprint Data Cluster

Full-text access to biology preprints. Its unique value over OpenAIRE is **quotable text** — you can find and cite what papers actually say about mechanisms, methods, and results, not just their titles and citation counts.

## When bioRxiv Adds Value

Use bioRxiv when you need **what's written in papers**, not just metadata:
- Mechanistic explanations (how a process works at molecular/cellular level)
- Methods details (experimental protocols, assay conditions)
- Literature review passages that synthesize a subfield
- Evidence statements with inline citations you can trace

**Do not use bioRxiv for:** clinical outcomes (use medRxiv), CS/ML topics (use WebSearch), or finding specific known papers by title (use OpenAIRE).

## Primary Tool: Vector Search

`vector_search_chunks` is your main tool here. It returns **text passages** ranked by semantic similarity — exactly what you need for quotable content.

```
vector_search_chunks(
  query="<natural language description of what you need>",
  limit=10,
  score_threshold=0.55
)
```

**Use score_threshold=0.55, not 0.7.** At 0.7 you typically get 0-2 results. At 0.55 you get 5-10 useful passages with acceptable noise. Scores above 0.65 are high-quality hits.

### Writing good vector search queries

Write queries as you'd describe the topic to a colleague, not as keyword strings:
- Good: `"how antibody drug conjugate linker stability affects payload release and off-target toxicity in vivo"`
- Bad: `"ADC linker stability toxicity"`

Longer, descriptive queries produce better semantic matches.

## Secondary Tool: Keyword Search

`keyword_search` is better for finding **specific papers** by exact terms:

```
keyword_search(
  query="CRISPR-Cas9 off-target editing",
  limit=10,
  context_mode="smart",
  response_format="markdown"
)
```

Best for: author names, gene names, drug names, technique names, specific terminology.

## Available Datasets (MUST call `list_datasets` before searching)

~50 subject areas (up to 1000 entries each): Neuroscience, Genomics, Bioinformatics, Cell Biology, Molecular Biology, Immunology, Microbiology, Cancer Biology, Pharmacology, Biochemistry, and more.

You MUST call `list_datasets(response_format="markdown")` before your first search. This is a small call (<500 chars) that tells you what subject areas exist and how many entries each has. Use the results to:
- Confirm bioRxiv covers your topic (if not, use WebSearch instead)
- Select `dataset_ids` for targeted searches (reduces noise)
- Avoid searching a biology preprint server for non-biology topics

## Combining with OpenAIRE

The strongest research pattern is:
1. **OpenAIRE** finds landmark papers (citation counts, influence class)
2. **bioRxiv** provides quotable full-text from those or related papers

This gives your answers both **authority** (high-citation papers) and **depth** (mechanistic detail from full text).

## Context Budget

| Operation | Size | Action |
|-----------|------|--------|
| `vector_search_chunks` (10 results) | ~2000 chars | Safe |
| `keyword_search` (10 results, markdown) | ~2000 chars | Safe |
| `list_datasets` (markdown, no schema) | Small | Safe. Never `include_schema=true` |
| `get_entry_content` (full paper) | 10K-100K | Paginate: use `char_offset` + `char_limit` |
