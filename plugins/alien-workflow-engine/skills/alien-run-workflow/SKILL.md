---
name: alien-run-workflow
description: Run an Alien workflow or agent and collect its result — trigger asynchronously and poll, smoke-test synchronously, get connection info for a real client, and read run-cost analytics. Use whenever the task is to execute, test, or wire up an already-built Alien agent or workflow, or to find where a run's output lands.
---

# Alien: Run a Workflow
There are three ways to run an Alien workflow/agent, and they are not
interchangeable. Pick by intent: reliable execution, quick liveness check, or
handing a real application the wiring to call the agent itself.

## When to use
"Run the workflow", "test the agent", "did the run finish?", "how do I call
this agent from my app?", "how much did that run cost?".

## 1. Reliable execution — async trigger + poll
This is the default. It never hits a gateway timeout because the run happens in
the background.
1. `alien_run_workflow` — `workflow_id` (required), optional `input` (an object
   passed to the workflow's `http_request` node; for an agent this carries the
   prompt/messages). Returns a compact job view including the job `id` and
   `status`.
2. `alien_get_workflow_run` — `job_id` (required), optional `full` (bool,
   default false). Poll until `status` is terminal (completed/failed). Keep
   `full` false for normal polling; set it true only to debug the graph (it
   embeds the entire workflow and is large).
   - The final agent text lands at
     `result.results.<http_response node id>[0].results.content`. For an agent
     built with `alien_create_agent`, that is the response pass-through
     (`content` + `metadata`).

## 2. Quick liveness check — synchronous
`alien_run_agent` — `workflow_id` (required), `input` (prompt string, required),
optional `model` (default `gpt-4o` — this is the Responses-API model id, not the
agent's own configured LLM). It drives the OpenAI-compatible
`/agent/{id}/responses` endpoint non-streaming inside one request, with a ~30s
gateway timeout. Use it ONLY to confirm a just-created agent responds. A
multi-tool-call agent will time out here even when the run would succeed —
the tool detects the timeout and points you back to path 1. Never use this as
the real execution path for a working agent.

## 3. Wire a real client
`alien_get_agent_connection_info` — `workflow_id`. Returns the agent's
Responses-API base URL and `/responses` endpoint, plus ready-to-use `curl` and
Python (OpenAI SDK) snippets with the auth header. This is how an external app
should actually talk to a deployed agent: the platform impersonates an
OpenAI-compatible `/responses` endpoint per workflow, so any OpenAI client works
by pointing `base_url` at the agent and streaming `input`.

## Run-cost analytics
`alien_get_workflow_consumption` — optional `period` (`live|daily|weekly|
monthly`, default monthly), optional `start`/`end` (ISO8601). Org-scoped
compute spend for workflow runs. This is compute cost, distinct from dataset
royalties (see the `alien-platform-basics` plugin's `alien-analytics`).

## Gotchas
- Abilities: `alien_run_workflow` needs `workflow:write`, `alien_run_agent`
  needs `workflow:run`, poll/connection/consumption need `workflow:read`.
- A failed job carries the reason in its `error` field and per-node errors in
  the full record — pull `full=true` on `alien_get_workflow_run` to see the
  node that failed and its traceback.
- If a fresh agent fails immediately with `worker_disconnected`, the graph was
  hand-built or otherwise missing session wiring — rebuild via
  `alien_create_agent` (see `alien-build-agent`).

## Next
- Build or fix the agent you're running: `alien-build-agent`.
- Understand what the graph is doing: `alien-workflow-engine`.
- Delete a finished/broken workflow: the `alien-platform-basics` plugin's
  `alien-cleanup` (`alien_delete_workflow`).
