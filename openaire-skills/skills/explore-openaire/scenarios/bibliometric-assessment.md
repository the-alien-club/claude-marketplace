# Bibliometric Assessment

Find landmark, trending, early-impact, or most-cited papers in a field.

## Domain Context

- OpenAIRE provides four bibliometric dimensions:
  - **Influence:** field-normalized, time-adjusted citation impact (best for foundational papers)
  - **Popularity:** recent citation velocity (best for currently trending)
  - **Impulse:** early citation momentum post-publication (best for immediate impact)
  - **Citation count:** absolute citations (best for most-cited regardless of field/age)
- Classes range from C1 (top 0.01%) to C5 (bottom 90%).
- Combining metrics reveals significance:
  - C1 across all = unambiguous landmark
  - C1 influence + C5 impulse = foundational but not recently trending
  - C5 influence + C1 popularity = currently hot but not yet established

## Choosing the Right Tool

| Focus | Tool | Measures |
|-------|------|----------|
| Foundational/seminal | `find_by_influence_class` | Field-normalized citation impact (time-adjusted) |
| Currently trending | `find_by_popularity_class` | Recent citation velocity |
| Immediate early impact | `find_by_impulse_class` | Early citation momentum post-publication |
| Most-cited (raw) | `find_by_citation_count_class` | Absolute citation count |

## Workflow

### Step 1: Find papers by bibliometric class

```
openaire_find_by_influence_class(
  influence_class="C1",
  query="<topic>",
  type=["publication"],
  page_size=5
)
```

Swap `influence_class` / tool name based on the user's focus. Output is ~500 chars per call.

### Step 2 (optional): Cross-check with a second metric

Run in parallel with Step 1:

```
openaire_find_by_popularity_class(
  popularity_class="C1",
  query="<topic>",
  type=["publication"],
  page_size=5
)
```

### Step 3: Deep-dive on selected papers

```
openaire_get_research_product_details(identifier="<doi>")
```

This returns all four bibliometric classes for a single paper (~1500 chars). Use to triangulate significance.

- If 3+ DOIs: **MUST** use Task tool for parallel subagents.

### Step 4 (optional): Trace citation lineage of a landmark paper

```
openaire_explore_research_relationships(
  target_pid="<landmark_doi>",
  relationship_type="cites",
  page_size=20
)
```
