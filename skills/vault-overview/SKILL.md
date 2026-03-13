---
name: vault-overview
description: >
  Quick situational awareness of the markdown (.md) vault: index status,
  document clusters, and file tree via mdvdb. Use when starting work in a
  vault or when asked about the state of the knowledge base.
allowed-tools:
  - Bash(mdvdb *)
---

# Vault Overview

Get a quick picture of the vault's current state. Only works with `.md` files indexed by mdvdb.

## Steps

1. Run these three commands in parallel:
   ```
   mdvdb status --json
   mdvdb clusters --json
   mdvdb tree --json
   ```

2. Present a structured overview:

   **Index Status** (from `status`):
   - Number of documents, chunks, and vectors indexed
   - Index file size
   - Whether the index is up to date

   **Topic Clusters** (from `clusters`):
   - Each cluster with its TF-IDF keyword label and document count
   - Highlight the largest and smallest clusters

   **File Tree** (from `tree`):
   - Total file count
   - How many files are synced vs. pending vs. new
   - Mention any unindexed files

3. End with actionable suggestions:
   - If files are unindexed: "Run `mdvdb ingest` to index new files"
   - If the index is empty: "Run `mdvdb ingest` to build the index"
   - If clusters look unbalanced: note which topics dominate
