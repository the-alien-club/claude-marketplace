# Author Landscape

Explore a researcher's publication history, collaborators, and research areas.

## Domain Context

- ORCID coverage in OpenAIRE is sparse — name-based search typically has better coverage than ORCID lookup.
- Author profiles reveal publication history, co-author networks, and research areas through subject analysis.
- Co-authorship networks show collaboration patterns.
- The person index (`search_persons`) is very sparse; publication-based author discovery works better.

## Workflow

### Step 1: Get author profile

```
openaire_get_author_profile(
  author_name="<full name>",
  limit=20,
  include_coauthors=true
)
```

**Always use `author_name` over `orcid`.** ORCID-based searches frequently return empty results even for valid ORCIDs.

Output at limit=20 is ~2K chars (safe in main context). If limit>30, delegate to a subagent.

### Step 2 (if deeper collaboration analysis needed): Build co-authorship network

**MUST delegate via Task tool** — this is a context bomb (15K+ chars even at depth=1):

```
Task(
  description="Analyze coauthorship network for <name>",
  subagent_type="general-purpose",
  prompt="Use mcp__openaire-local__openaire_analyze_coauthorship_network with
    author_name='<name>', max_depth=1, limit=30, min_collaborations=2.
    Summarize: total collaborators, top 5 by shared papers, any cross-institutional
    clusters. Report node count, edge count, and key collaborator names."
)
```

Never use `max_depth=2` — it explodes exponentially.

### Step 3 (optional): Search for specific publications

```
openaire_search_research_products(
  author_full_name=["<name>"],
  sort_by="citationCount DESC", page_size=5
)
```

### Step 4 (optional): Trace a specific paper's impact

```
openaire_explore_research_relationships(
  target_pid="<doi>",
  relationship_type="cites",
  page_size=20
)
```
