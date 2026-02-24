# Project Impact

Assess the research impact and outputs of a funded project.

## Domain Context

- OpenAIRE has strong coverage of EU-funded projects (Horizon 2020/Europe).
- Project outputs can include publications, datasets, and software.
- Output volume for large projects can be very high (hundreds of items) — project output retrieval may benefit from summarization.
- Project IDs use format `corda_______::` (7 underscores).
- Grant codes are the most reliable way to find specific projects.

## Workflow

### Step 1: Find the project

```
openaire_search_projects(
  search="<project name or topic>",
  funding_short_name=["EC"],     # or "NSF", "NIH" etc.
  page_size=5
)
```

If you have a grant code, you can also search by `code=["<grant_code>"]`.

### Step 2: Get project outputs — MUST delegate

**MUST use Task tool** — this is a context bomb (30K+ chars at default limits):

```
Task(
  description="Get outputs for project <name>",
  subagent_type="general-purpose",
  prompt="Use mcp__openaire-local__openaire_get_project_outputs with
    project_code='<code>' and page_size=100. Summarize: total outputs by type,
    top 5 most-cited publications, any datasets or software produced.
    Report counts and key DOIs."
)
```

### Step 3: Assess key outputs

For the top DOIs returned by the subagent:

- If 1-2 DOIs: call `get_research_product_details` directly.
- If 3+ DOIs: launch parallel subagents via Task tool.

### Step 4 (optional): Check publication trend for the project topic

```
openaire_analyze_research_trends(
  search="<project topic>",
  from_year=<project_start_year>,
  to_year=<project_end_year + 3>
)
```
