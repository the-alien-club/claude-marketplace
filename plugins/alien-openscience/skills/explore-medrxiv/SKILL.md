---
name: explore-medrxiv
description: medRxiv Preprint Data Cluster Exploration Skill. Use when searching for preprint full-text content in clinical medicine, epidemiology, public health, and health sciences via the medRxiv MCP server.
---

# medRxiv Preprint Data Cluster

Full-text access to medical and health sciences preprints. Its unique value is **clinical evidence text** — trial results, patient outcomes, epidemiological findings, and health policy analysis written in the papers themselves.

## When medRxiv Adds Value

Use medRxiv when you need **clinical/health content from papers**, not just metadata:
- Clinical trial results and study designs
- Epidemiological methods and cohort descriptions
- Patient outcome data and statistical analyses
- Public health policy evidence
- Drug safety and efficacy data from preprints

**Do not use medRxiv for:** basic science mechanisms (use bioRxiv), CS/ML topics (use WebSearch), or finding specific known papers by title (use OpenAIRE).

## When to Choose medRxiv Over bioRxiv

- Question about **what happens to patients** → medRxiv
- Question about **how something works at molecular level** → bioRxiv
- Question about **drug clinical outcomes** → medRxiv
- Question about **drug mechanism of action** → bioRxiv
- Question spans both → run both in parallel

## Primary Tool: Vector Search

```
vector_search_chunks(
  query="<clinical question in natural language>",
  limit=10,
  score_threshold=0.55
)
```

**Use score_threshold=0.55, not 0.7.** Same rationale as bioRxiv — 0.7 is too restrictive for most queries.

Write queries as clinical questions:
- Good: `"long-term cardiovascular outcomes in patients treated with GLP-1 receptor agonists compared to standard care"`
- Bad: `"GLP-1 cardiovascular"`

## Secondary Tool: Keyword Search

Better for specific clinical terms, drug names, trial identifiers:

```
keyword_search(
  query="remdesivir COVID-19 hospitalization outcomes",
  limit=10,
  context_mode="smart",
  response_format="markdown"
)
```

## Available Datasets (MUST call `list_datasets` before searching)

~50 clinical specialities. Largest:
- Infectious Diseases (11,744), Epidemiology (10,367), Public and Global Health (6,627)
- Neurology (4,512), Genetic and Genomic Medicine (4,393), Psychiatry (3,725)
- Cardiovascular Medicine (3,129), Health Informatics (2,779), Oncology (2,243)

You MUST call `list_datasets(response_format="markdown")` before your first search. This is a small call (<500 chars) that tells you what clinical specialities exist and how many entries each has. Use the results to:
- Confirm medRxiv covers your clinical area (if not, use WebSearch instead)
- Select `dataset_ids` for targeted searches (reduces noise)
- Avoid searching a medical preprint server for non-clinical topics

## Combining with OpenAIRE

1. **OpenAIRE** finds high-citation published studies (metadata + impact)
2. **medRxiv** provides full-text clinical detail from preprint versions

This combination gives answers both **citation authority** and **clinical depth** (methods, outcomes, statistics from the text itself).

## Context Budget

| Operation | Size | Action |
|-----------|------|--------|
| `vector_search_chunks` (10 results) | ~2000 chars | Safe |
| `keyword_search` (10 results, markdown) | ~2000 chars | Safe |
| `list_datasets` (markdown, no schema) | Small | Safe. Never `include_schema=true` |
| `get_entry_content` (full paper) | 10K-100K | Paginate: use `char_offset` + `char_limit` |
