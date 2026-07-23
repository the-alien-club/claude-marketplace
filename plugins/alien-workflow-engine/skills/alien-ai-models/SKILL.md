---
name: alien-ai-models
description: List the AI models available on the Alien platform and get the exact model slug to use when building an agent or subagent. Use before alien_create_agent, alien_update_agent, or alien_add_subagent, or whenever you need to know which LLM, embedding, reranker, or TTS models the platform offers.
---

# Alien: AI Models
Every agent and subagent runs on a model identified by a `slug`. This skill
gets you the valid slugs; guessing a model name will fail agent creation.

## When to use
Right before any tool that takes a `model` argument —
`alien_create_agent`, `alien_update_agent`, `alien_add_subagent` — or when
asked what models/providers the platform supports.

## Call
`alien_list_ai_models` — optional `model_type`
(`llm|embedding|reranker|tts|image-generation|video-generation|all`, default
`llm`), optional `select` (`public|private`, default `public`). Returns
`{count, models: [{slug, name, model_type, provider, is_verified}]}`. The
`slug` is the value you pass to `model=`.

## Notes
- The default (`model_type=llm`, `select=public`) returns the verified public
  LLM catalog — the safe set for building agents. That default is deliberate:
  querying private models with no active org usually returns an empty list,
  which looks like "no models" when there are plenty.
- To offer a specific capability (a reranker for `rerank_search_results`, a TTS
  voice model for `voice_generation`), filter by the matching `model_type`.
- `is_verified: true` marks models the platform has validated end-to-end —
  prefer these unless you have a reason not to.
- The `model` argument to `alien_run_agent` is separate: that is the
  Responses-API model id for a synchronous smoke test, not the agent's own
  configured LLM. The agent's real model is the slug you set at build time.

## Next
- Use the slug to build: `alien-build-agent`.
- Understand where the model sits in the graph: `alien-workflow-engine`.
