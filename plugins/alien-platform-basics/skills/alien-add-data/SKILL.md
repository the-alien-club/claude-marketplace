---
name: alien-add-data
description: Get documents into the Alien platform — provision a data cluster, create a dataset, wire an ingestion pipeline, upload documents, and poll until they're searchable. Use whenever the user wants to ingest, upload, index, or add documents/files/a corpus to Alien.
---

# Alien: Add Data (Flow 1)
Documents live on a cluster's data-plane (its own FastAPI service), reached
through the backend's proxy. Provisioning a cluster stands up real
infrastructure (PostgreSQL, object storage, the vector store, and the workflow
engine), so this flow has two asynchronous waits: cluster provisioning, then
per-document ingestion. Run `alien-getting-started` first if you have not
already confirmed identity and write access.

## Call sequence
1. `alien_create_cluster` — `name` and `embedding_provider` required;
   optional `is_public`, `data_plane_id`, `source_name`, `source_url`,
   `source_description`, `metadata`. Returns immediately with a cluster
   record that is still provisioning. Skip this step and reuse an existing
   cluster id (from `alien_list_clusters`) when one already exists.
2. Poll `alien_get_cluster_provisioning_status` with the returned
   `cluster_id` until the cluster reports ready. This can take several
   minutes — do not proceed to dataset creation before it is ready.
3. `alien_create_dataset` — `cluster_id`, `name`, `slug` required; optional
   `description`, `dataset_type` (`text` default, or `audio` / `voice` /
   `images`), `schema_definition` (a minimal text schema is generated for you
   when omitted).
4. `alien_apply_pipeline_preset` — `cluster_id`, `dataset_id` required;
   optional `preset_name` (defaults to `general_purpose`) and `overrides`.
   Wires OCR → chunk → embed with on-upload triggering. Apply this BEFORE
   adding documents, or uploads will not auto-process.
5. `alien_add_document` — `cluster_id`, `dataset_id`, `name` required;
   optional `slug`, `metadata`, `url`. Two behaviors depending on `url`:
   - If a public `url` is given, the tool fetches the bytes itself, uploads
     them, and this auto-fires ingestion.
   - If `url` is omitted, the MCP cannot stream local file bytes through a
     tool call — it creates the entry and returns a ready-to-run `curl`
     recipe for you to upload the file yourself. This is the documented
     fallback, not an error; run the curl command, then continue to step 6.
6. Poll `alien_get_ingestion_status` — `cluster_id`, `entry_id` (from the
   entry returned by `alien_add_document`) required. Repeat until status is
   `processed`, which means the document is searchable.

## The ingestion loop
Steps 5–6 repeat per document: add, then poll status to `processed` before
treating a document as available for retrieval. For a batch of documents, add
them all first, then poll each entry id — don't serialize add→poll→add→poll
if the corpus is large; the ingestion pipeline runs independently per entry.

## Gotchas
- Cluster and dataset creation are genuinely asynchronous. Don't skip the
  provisioning-status poll — creating a dataset against a not-yet-ready
  cluster will fail.
- `alien_apply_pipeline_preset` must run before `alien_add_document`, not
  after — an entry uploaded before the preset is applied will not be
  auto-processed by that preset.
- When `alien_add_document` returns the curl fallback, the token placeholder
  in the recipe (`<YOUR_ALIEN_ACCESS_TOKEN>`) must be replaced with a real
  access token before running it — never fabricate or reuse a token from
  another session.
- `dataset_type` changes the ingestion/search modality; picking the wrong one
  (e.g. `text` for an audio corpus) will not just format oddly — it changes
  which pipeline steps apply.

## Next
- Once data is ingested, wrap it into a shareable MCP config: `alien-publish-mcp`.
- To also expose an external API alongside this data: `alien-external-api-to-mcp`.
- To control who can see this dataset or price it: `alien-rbac-and-pricing`.
- Tearing a cluster or dataset back down: `alien-cleanup`.
