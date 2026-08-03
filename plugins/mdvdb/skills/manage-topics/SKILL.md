---
name: manage-topics
description: >
  Create, update, tune, and inspect Collection-wide or Shard-local Topics
  (custom clusters) in an mdvdb vault: clusters
  add/update/remove/list/unassigned, per-Topic thresholds, independent Shard
  scopes, and clustering config via mdvdb config set. Use when the user wants
  to organize documents into Topics, color Tesseract's graph, tune a Topic,
  review Unassigned documents, or give a Shard/sub-collection its own Topics.
---

# Manage Topics

Define and tune Topics — user-defined semantic groupings of documents.
Only works with `.md` files indexed by mdvdb.

**Input:** `$ARGUMENTS` is the requested topic operation (create, update,
remove, inspect, tune).

## Concepts

- **Auto clusters** (`mdvdb clusters --json`) are computed by Leiden community
  detection (seeded, stable ids; K-means fallback). They emerge from the data
  and can't be edited directly. `--shard <ID>` recomputes them lazily from
  only that Shard's stored document vectors.
- **Topics** (custom clusters) are user-defined in `.markdownvdb/config.yaml`
  and managed via the CLI. Each Topic has a name, optional description,
  optional seed keywords, and optional similarity threshold.
- Collection Topics and every Shard's Topics are independent owners. Names
  need only be unique within one owner; definitions and assignments never
  inherit between the Collection, parent Shards, children, or siblings.
- Assignment is **multi-label**: a document joins every topic whose cosine
  similarity meets `max(topic threshold, clustering.topics.min_similarity)` —
  the global floor defaults to 0.30. Documents matching no topic land in the
  explicit **Unassigned** bucket.
- Topic definition changes are detected automatically on the next ingest
  (fingerprint check) — centroids and assignments recompute as needed.

## Steps

1. Choose the owner. Omit `--shard` for Collection Topics or add the immutable
   Shard ID for local Topics. Confirm available IDs with
   `mdvdb shards list --json`.

2. Inspect the current state:
   ```
   mdvdb clusters list --json        # topic definitions from config.yaml
   mdvdb clusters --custom --json    # computed topics: document_count + mean_score per topic
   mdvdb clusters unassigned --json  # documents matching no topic

   mdvdb clusters --shard research list --json
   mdvdb clusters --shard research --custom --json
   mdvdb clusters --shard research unassigned --json
   ```
   To list a topic's member documents with per-document similarity scores,
   use `mdvdb graph [--shard ID] --json` — each node carries
   `custom_cluster_ids` and `custom_cluster_scores`. For a Shard response,
   confirm `analysis.context: "shard"` and the expected `shard_id`.

3. Create a Topic (needs seeds and/or a description — a description improves
   matching). Keep the same optional `--shard <ID>` on every operation:
   ```
   mdvdb clusters add "Topic Name" --seeds "keyword1,keyword2" --description "What this topic covers" --threshold 0.4
   mdvdb clusters --shard research add "Topic Name" --seeds "keyword1,keyword2" --description "Local meaning" --threshold 0.4
   ```

4. Update a Topic (only pass what changes):
   ```
   mdvdb clusters update "Topic Name" --seeds "..." --description "..." --threshold 0.35 --rename "New Name"
   mdvdb clusters --shard research update "Topic Name" --threshold 0.35
   ```
   `--description ""` clears the description; `--threshold=-1` (equals form
   required) clears the per-topic threshold so the global floor applies.

5. Remove a Topic:
   ```
   mdvdb clusters remove "Topic Name"
   mdvdb clusters --shard research remove "Topic Name"
   ```
   Removing a Shard itself also removes that Shard's local Topic definitions,
   but never its files, folder, embeddings, or shared index.

6. Tune the Collection-wide assignment floor:
   ```
   mdvdb config set clustering.topics.min_similarity 0.4
   ```
   `config set` writes any dotted key into `.markdownvdb/config.yaml`.

7. Verify and iterate: definition changes are picked up at the next
   Collection-wide ingest (fingerprint check), so run `mdvdb ingest` first,
   then re-run the computed and Unassigned commands for the same owner.
   Report per-Topic document counts and how the Unassigned bucket changed.
   Until a new Shard Topic centroid is ingested, its graph remains usable but
   reports `topics: "needs_ingest"`; distinguish that from
   `topics: "none"`. After ingest, verify the same Shard's graph reports
   `analysis.topics: "ready"`; ingest success alone is not proof because one
   Shard's Topic refresh failure is non-fatal, and empty computed arrays are
   ambiguous.
   - Topic too broad (grabs everything) → raise its `--threshold`
   - Many relevant docs Unassigned → lower thresholds, improve descriptions/
     seeds, or add missing topics

## Related

manage-shards creates and scopes named sub-collections; vault-overview reports
Topics alongside index status. If the user mentions
"Tesseract" or "tesseract.md", that's the desktop app companion for mdvdb —
Topics color its 3D knowledge graph and power its scoped Topics UI, reading
the same `.markdownvdb/config.yaml` these commands write. Obsidian tags and
graph color groups remain Collection-owned when Tesseract imports them; they
do not automatically become Shard Topics.
