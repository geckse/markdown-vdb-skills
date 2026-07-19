---
name: manage-topics
description: >
  Create, update, tune, and inspect topics (custom clusters) in an mdvdb
  vault: clusters add/update/remove/list/unassigned, per-topic thresholds,
  and clustering config via mdvdb config set. Topics are multi-label semantic
  groupings that also color the Tesseract app's knowledge graph. Use when the
  user wants to organize documents into topics, rename or tune a topic,
  review unassigned documents, or adjust clustering settings.
---

# Manage Topics

Define and tune topics — user-defined semantic groupings of documents.
Only works with `.md` files indexed by mdvdb.

**Input:** `$ARGUMENTS` is the requested topic operation (create, update,
remove, inspect, tune).

## Concepts

- **Auto clusters** (`mdvdb clusters --json`) are computed by Leiden community
  detection (seeded, stable ids; K-means fallback). They emerge from the data
  and can't be edited directly.
- **Topics** (custom clusters) are user-defined in `.markdownvdb/config.yaml`
  and managed via the CLI. Each topic has a name, optional description,
  optional seed keywords, and optional similarity threshold.
- Assignment is **multi-label**: a document joins every topic whose cosine
  similarity meets `max(topic threshold, clustering.topics.min_similarity)` —
  the global floor defaults to 0.30. Documents matching no topic land in the
  explicit **Unassigned** bucket.
- Topic definition changes are detected automatically on the next ingest
  (fingerprint check) — centroids and assignments recompute as needed.

## Steps

1. Inspect the current state:
   ```
   mdvdb clusters list --json        # topic definitions from config.yaml
   mdvdb clusters --custom --json    # computed topics: document_count + mean_score per topic
   mdvdb clusters unassigned --json  # documents matching no topic
   ```
   To list a topic's member documents with per-document similarity scores,
   use `mdvdb graph --json` — each node carries `custom_cluster_ids` and
   `custom_cluster_scores`.

2. Create a topic (needs seeds and/or a description — a description improves
   matching):
   ```
   mdvdb clusters add "Topic Name" --seeds "keyword1,keyword2" --description "What this topic covers" --threshold 0.4
   ```

3. Update a topic (only pass what changes):
   ```
   mdvdb clusters update "Topic Name" --seeds "..." --description "..." --threshold 0.35 --rename "New Name"
   ```
   `--description ""` clears the description; `--threshold=-1` (equals form
   required) clears the per-topic threshold so the global floor applies.

4. Remove a topic:
   ```
   mdvdb clusters remove "Topic Name"
   ```

5. Tune the global assignment floor:
   ```
   mdvdb config set clustering.topics.min_similarity 0.4
   ```
   `config set` writes any dotted key into `.markdownvdb/config.yaml`.

6. Verify and iterate: definition changes are picked up at the next ingest
   (fingerprint check), so run `mdvdb ingest` first, then re-run
   `mdvdb clusters --custom --json` and `mdvdb clusters unassigned --json`,
   and report per-topic document counts and how the Unassigned bucket changed.
   - Topic too broad (grabs everything) → raise its `--threshold`
   - Many relevant docs Unassigned → lower thresholds, improve descriptions/
     seeds, or add missing topics

## Related

vault-overview reports topics alongside index status. If the user mentions
"Tesseract" or "tesseract.md", that's the desktop app companion for mdvdb —
topics color its 3D knowledge graph and power its topics UI, reading the same
`.markdownvdb/config.yaml` these commands write. Tesseract also auto-imports
Obsidian tags and graph color groups as topics via these same CLI commands, so
some topic definitions may be app-managed.
