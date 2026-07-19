---
name: vault-overview
description: >
  Quick situational awareness of the markdown (.md) vault: index status,
  vault statistics, document clusters, topics, and file tree via mdvdb. Use
  when starting work in a vault, when asked about the state of the knowledge
  base, or when the user asks about the vault behind the Tesseract app.
---

# Vault Overview

Get a quick picture of the vault's current state. Only works with `.md` files indexed by mdvdb.

## Steps

1. Run these commands in parallel:
   ```
   mdvdb status --json
   mdvdb info --json
   mdvdb clusters --json
   mdvdb tree --json
   ```

2. Present a structured overview:

   **Index Status** (from `status`):
   - Number of documents, chunks, and vectors indexed
   - Index file size

   **Vault Stats** (from `info`):
   - Files on disk vs indexed (`file_count` / `indexed_file_count`)
   - Sync breakdown: `sync.new`, `sync.changed`, `sync.unchanged`, `sync.deleted`
   - Whether the index is up to date (`new`, `changed`, and `deleted` all 0)
   - Embedding provider, model, and dimensions
   - Full-reindex cost estimate (`reindex_estimated_tokens`,
     `reindex_estimated_api_calls`)

   **Clusters & Topics** (from `clusters`):
   - Auto clusters come from Leiden community detection (stable ids); each has
     a TF-IDF keyword label and document count — highlight largest and smallest
   - If topics are defined, also run `mdvdb clusters --custom --json` and report
     per-topic document counts plus the Unassigned bucket
     (`mdvdb clusters unassigned --json`)

   **File Tree** (from `tree`):
   - Total file count
   - How many files are indexed vs. modified vs. new vs. deleted
     (`indexed_count`, `modified_count`, `new_count`, `deleted_count`)
   - Mention any unindexed files

3. End with actionable suggestions:
   - If `sync.new` or `sync.changed` > 0: "Run `mdvdb ingest` to index them"
   - If the index is empty: "Run `mdvdb ingest` to build the index"
   - If many documents are Unassigned: suggest defining or tuning topics
     (see the manage-topics skill)
   - If auto clusters look unbalanced: note which clusters dominate

If the user mentions "Tesseract" or "tesseract.md", that's the desktop app
companion for mdvdb — it operates on this same vault and `.markdownvdb/` index,
so everything reported here matches what the app displays.
