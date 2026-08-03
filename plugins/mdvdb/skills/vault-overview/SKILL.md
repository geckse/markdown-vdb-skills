---
name: vault-overview
description: >
  Quick situational awareness of an mdvdb Collection or named Shard:
  definitions, index status, scoped statistics, document clusters, Topics,
  and file tree. Use when starting work in a vault or sub-collection, when
  asked about knowledge-base state, or when the user asks about the Collection
  or active Shard behind the Tesseract app.
---

# Vault Overview

Get a quick picture of the vault's current state. Only works with `.md` files indexed by mdvdb.

## Steps

1. Run these commands in parallel:
   ```
   mdvdb status --json
   mdvdb shards list --json
   mdvdb info --json
   mdvdb clusters --json
   mdvdb tree --json
   ```

2. Present a structured overview:

   **Shards** (from `shards list`):
   - Named recursive sub-collections, inferred `parent_id`, folder path, and
     missing definitions (`exists: false`)
   - If the user requested one Shard, rerun `info`, `clusters`, and `tree`
     with `--shard <ID>` and report that local context. Keep `status`
     Collection-wide because the shared index is not partitioned.

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
   - For a requested Shard, use `mdvdb clusters --shard <ID> --json`,
     `--custom`, and `unassigned`. Its automatic clusters and Topics are
     independent from the Collection and every other Shard.

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
   - If a Shard is missing: repair its path with `mdvdb shards update` or
     repair a renamed prefix with `mdvdb shards retarget`

If the user mentions "Tesseract" or "tesseract.md", that's the desktop app
companion for mdvdb — it operates on this same vault and `.markdownvdb/` index,
so everything reported here matches what the app displays. Its active Shard
scopes the visible tree, search, global graph, Information, and schema while
the underlying index status remains Collection-wide.

Use manage-shards for Shard lifecycle and scoped command rules.
