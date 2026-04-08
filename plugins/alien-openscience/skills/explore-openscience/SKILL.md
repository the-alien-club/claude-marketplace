---
name: explore-openscience
description: Open Science Research Toolkit — strategy guide for searching across OpenAIRE, bioRxiv, medRxiv, and web search. Load this first for any research question.
---

# Open Science Research Toolkit

## Metacognitive Frame

This skill gives you access to structured research tools (OpenAIRE, bioRxiv, medRxiv) alongside WebSearch. Before doing anything, adopt this frame — it will shape every tool choice you make in this session.

**Notice your defaults.** You have a strong prior toward WebSearch. It's the general-purpose tool, it always returns something, and reaching for it feels like the safe move. This prior is well-calibrated for most tasks — but for research questions it leads you to use specialized MCP tools as redundant search engines (same keywords, same role) rather than for what they uniquely provide. The result: you get blog posts and vendor pages from WebSearch, while the structured evidence sitting in MCP goes unused.

**Each source provides something the others structurally cannot:**

| Source | What only it provides |
|--------|----------------------|
| **OpenAIRE** | Bibliometric authority — citation counts, influence classes (C1-C5), field-normalized ranking. This lets you distinguish a landmark paper (899 citations, influence C3) from a blog post summarizing it. No other source quantifies a paper's importance. |
| **bioRxiv** | Quotable full-text from biology preprints — what the paper actually says about mechanisms, methods, results. Not metadata, not summaries — the text itself. |
| **medRxiv** | Same, for clinical medicine — trial results, patient outcomes, epidemiological evidence, written in the paper's own words. |
| **WebSearch** | Breadth — everything MCP doesn't index. CS/ML papers, clinical guidelines, very recent work, institutional pages, specific known papers. |

**The test for whether you're using MCP well:** after answering, ask yourself — did OpenAIRE give me something I couldn't have gotten from WebSearch? If you only called `search_research_products` with the same keywords you'd put into WebSearch, the answer is no. If you called `find_by_influence_class` and got back the field's landmark papers ranked by normalized impact — that's unique value no web search can replicate.

**Adopt this as a running practice:** after each research answer in a session, briefly note which sources earned their keep and which were overhead. This calibrates your intuition across questions and is more durable than any routing table.

## The Workflow

### Step 1: MCP first (parallel with each other, not with WebSearch)

You MUST include `find_by_influence_class` and one `list_datasets` call in your first tool batch. Do not skip these in favor of `search_research_products` alone.

```
# 1. OpenAIRE: landmark papers via bibliometric filtering (REQUIRED)
find_by_influence_class(influence_class="C3", query="<2-3 key terms>", page_size=10)

# 2. OpenAIRE: broader keyword search sorted by citations
search_research_products(query="<2-4 terms>", sort_by="citationCount DESC", page_size=10, detail="standard")

# 3. Datacluster: discover available datasets, then search (REQUIRED for biomedical topics)
list_datasets(response_format="markdown")  # Know what subject areas exist before searching
```

Which datacluster for #3:
- **Mechanisms** (molecular, cellular, biochemical) → bioRxiv
- **Clinical outcomes** (patients, trials, epidemiology) → medRxiv
- **Both or unclear** → run both in parallel

Keep the dataset listing in your context — use it to select `dataset_ids` for targeted searches in subsequent steps.

**Hard rule: If the topic is CS, ML, NLP, computer vision, physics, or social science — do NOT call bioRxiv or medRxiv. These are life science preprint servers with full-text access to biology and clinical medicine papers only. Use WebSearch for non-biomedical topics.**

### Step 2: Evaluate results before reaching for WebSearch (REQUIRED)

You MUST state in 1-2 sentences what you got from each source before proceeding. This is not optional — write it out explicitly:

- **OpenAIRE:** "Got N landmark papers on [topic], highest cited: [X] with [N] citations" OR "0 results — topic too niche for keyword search"
- **Datacluster:** "Got N passages, best score [X] on [topic] — usable/not usable" OR "Results off-topic — [source] does not cover this domain"

Name your gaps explicitly:
- Did OpenAIRE return landmark papers? (citations > 100 = useful anchors)
- Did the datacluster return quotable text? (score > 0.65 = directly usable)
- What specific topics or papers are still missing?

If MCP covered the core, WebSearch targets **periphery and recency** — not the same ground.

### Step 3: Targeted WebSearch for named gaps

Specific gap-filling queries only:
- A known paper MCP missed (e.g., "Hamblett 2004 drug antibody ratio")
- Clinical trial data, FDA approvals, guidelines
- CS/ML/engineering topics (weak OpenAIRE coverage)
- Very recent work (last 6 months)
- Broad reviews that synthesize across many studies

### Step 4: Cross-source enrichment

- Papers from WebSearch → look up in OpenAIRE for citation counts before citing
- Papers from OpenAIRE → check dataclusters for quotable full-text if you need mechanistic depth
- Prefer citing with bibliometric data (citation count + influence class) when available

## After You Answer (REQUIRED self-check)

You MUST write a brief self-check after each research answer. If you find yourself skipping this, stop and do it now — it is not optional. Write one line per question:

1. **MCP unique value?** Name one thing MCP gave you that WebSearch couldn't (influence class, quotable passage, trend data). If nothing — note what you should have tried.
2. **Did you use OpenAIRE as a research graph or as a keyword search engine?** OpenAIRE's value is structured bibliometric data — influence classes, citation networks, trend analysis, field-normalized ranking. If you only called `search_research_products` with the same keywords you'd put into WebSearch, you missed what makes it unique.
3. **Datacluster evaluation?** Did you state whether datacluster results were usable? If you received results and moved on without commenting, go back and evaluate them now.
4. **Did you skip a relevant datacluster?** Clinical question without medRxiv? Mechanisms question without bioRxiv?

If you notice you skipped Steps 1 or 2 above — acknowledge it. This self-awareness is more valuable than retroactively fixing the search strategy.

## Search Tips

**OpenAIRE uses AND logic.** More terms = fewer results (opposite of Google). Keep queries to 2-4 key terms. If 0 results, drop a term.

**Datacluster score threshold: 0.55, not 0.7.** At 0.7 you get 0-2 results and conclude "nothing here." At 0.55 you get 5-10 useful passages. Scores above 0.65 are high-quality hits.

**`find_by_influence_class` is your most valuable first call.** C3 (top 1%) for most fields. C2 (top 0.1%) for very broad fields. This surfaces the papers that matter — `search_research_products` alone cannot do this.

**Don't abandon MCP after one failed query.** Drop terms on OpenAIRE. Lower threshold on dataclusters. Try keyword instead of vector, or vice versa. Conclude "MCP doesn't have this" only after reformulation.

**Adapt queries to the source.** Short keywords for OpenAIRE. Natural language descriptions for vector search. Specific gap descriptions for WebSearch. Sending the same string everywhere wastes each source's strengths.

## Source Selection by Question Type

| Question type | Start with | Then add | WebSearch for |
|---|---|---|---|
| **Literature synthesis** | OpenAIRE `find_by_influence_class` + `search_research_products` | bioRxiv/medRxiv vector search | Recency, specific missing papers |
| **Citation identification** | OpenAIRE `main_title` or keyword search | — | WebFetch to read reference lists |
| **Clinical/medical** | medRxiv keyword + vector search | OpenAIRE for citation authority | Guidelines, trial registries |
| **Biology/methods** | bioRxiv vector search | OpenAIRE for metadata | Protocols, vendor docs |
| **CS/ML/engineering** | WebSearch (weak MCP coverage) | OpenAIRE as secondary | The one case where WebSearch leads |

## bioRxiv vs medRxiv

- **bioRxiv**: Biology, neuroscience, genomics, bioinformatics, cell biology, molecular biology, immunology, pharmacology, cancer biology, biochemistry
- **medRxiv**: Clinical medicine, epidemiology, public health, infectious diseases, clinical trials, surgery, psychiatry, cardiology, oncology outcomes

**Mechanisms** → bioRxiv. **Patients** → medRxiv.

## Context Budget

| Operation | Size | Action |
|-----------|------|--------|
| `find_by_influence_class` (10 results) | ~500 chars | Safe — most valuable first call |
| `search_research_products` (10, standard) | ~2500 chars | Safe |
| `vector_search_chunks` (10 results) | ~2000 chars | Safe |
| `keyword_search` (10, markdown) | ~2000 chars | Safe |
| `list_datasets` (markdown, no schema) | Small | Safe. Never `include_schema=true` |
| `get_project_outputs` | 30K+ | Always delegate to subagent |
| `analyze_coauthorship_network` | 15K+ | Always delegate to subagent |
| `get_entry_content` (full paper) | 10K-100K | Paginate with `char_offset`/`char_limit` |

## Delegating to Subagents

When you spawn Task subagents to parallelize research, treat them as first-class citizens — their results deserve the same evaluation and integration as your own tool calls. Be metacognitive about their capabilities.

**Subagent fallback behavior:** If a subagent cannot access MCP tools or gets errors from MCP calls, it MUST fall back to WebSearch — never to Bash, Grep, curl, or any terminal-based workaround. Writing scripts to call APIs directly is never acceptable. If WebSearch is also not producing useful results, the subagent should return early with what it has and state clearly: "MCP tools unavailable, WebSearch insufficient for this subtopic."

**Parent response to subagent failure:** If a subagent returns early due to tool access issues, do NOT spawn more subagents for the same task type. Instead, handle the remaining research directly in your main session where MCP tools are available, or accept the gap and synthesize from what you have.

**Instruct subagents to be metacognitive.** When writing the subagent's prompt, explicitly tell it to monitor its own behavior: if it finds itself making many Bash calls, writing Python scripts, or repeatedly searching without finding useful results, it should stop, acknowledge the issue, and return control to you with whatever it has gathered so far. A subagent that recognizes it is stuck and returns early is far more valuable than one that spirals through hundreds of tool calls producing nothing useful.
