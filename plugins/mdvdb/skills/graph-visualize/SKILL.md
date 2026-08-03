---
name: graph-visualize
description: >
  Export and summarize a Collection-wide or Shard-local mdvdb knowledge graph
  for visualization or analysis, including links, relations, automatic
  clusters, Topics, and descendant folder projections. Use when the user
  wants to inspect graph structure, analyze one named Shard/sub-collection,
  or verify that local communities and Topics differ from the Collection.
---

# Graph Visualize

Export and summarize the vault's knowledge graph.
Only works with `.md` files indexed by mdvdb.

**Input:** `$ARGUMENTS` can be:
- Empty: full document-level graph
- A path prefix (e.g., `docs/`): restrict to that subtree
- A Shard ID: use that Shard as the independent graph-analysis context
- `chunk`: use chunk-level granularity instead of document-level

## Steps

1. Determine the command flags from `$ARGUMENTS`:
   - If a path is specified: add `--path PREFIX`
   - If a Shard is specified: resolve its immutable ID with
     `mdvdb shards list --json`, then add `--shard ID`
   - If "chunk" is mentioned: add `--level chunk`
   - Default: `--level document`

2. Run the graph export:
   ```
   mdvdb graph --json --level document
   ```
   Add `--shard ID`, `--path PREFIX`, or `--level chunk` as needed.

   Without a Shard, `--path` only filters visible Collection topology and
   retains Collection analysis identities. With a Shard, the complete Shard
   is the independent analysis corpus and an optional descendant `--path`
   only projects visible topology:
   ```
   mdvdb graph --shard research --json
   mdvdb graph --shard research --path work/research/drafts --json
   ```
   An ancestor path clamps to the Shard; a disjoint path is empty.

3. Present a graph summary:
   - **Node count**: total documents/chunks in the graph
   - **Edge count**: total connections
   - **Typed vs body edges** (document-level graphs): edges carry a `field`
     key — `null` for body links, a frontmatter field name for typed
     relations. Summarize which relation types (fields) are present.
     (Chunk-level edges are embedding-similarity edges with a cosine `weight`,
     not links)
   - **Most connected nodes**: top 5 files by total degree (in + out edges)
   - **Isolated nodes**: files with zero connections
   - **Clusters**: if cluster data is present, group nodes by cluster
   - **Analysis provenance**: read `analysis.context`, `shard_id`,
     `shard_path`, and cluster/Topic status. Never label a Collection response
     as Shard data. Shard automatic clusters are computed lazily from only its
     existing stored document vectors; local Topics do not inherit from other
     scopes.
   - **Analysis state**: distinguish `too_small`, `disabled`, `none`,
     `needs_ingest`, and `error`. A Shard graph can keep topology and automatic
     clusters while new Topic definitions need a Collection-wide ingest.

   Document and chunk graphs are strict induced subgraphs: include only nodes
   in the visible boundary and edges whose endpoints are both present.

4. For semantic relationship analysis, `mdvdb edges --json` (no file argument)
   lists all vault-wide semantic edges, and `--relationship <label>` filters
   them by relationship-label substring — useful for "show every pair
   connected by X"-style questions. `edges` is Collection-wide and has no
   `--shard`; do not present that output as the closed Shard graph.

5. If the graph is small (under 30 nodes), present a text-based adjacency
   summary showing which files connect to which.

6. For larger graphs, focus on statistics and the most important hub nodes.

7. Mention that the raw JSON can be saved for external tools:
   `mdvdb graph --json > graph.json`. For tooling, `--compact` (alias
   `--intern-contexts`, requires `--json`) emits the versioned wire format
   used by the Tesseract app.

8. For interactive viewing: if the user has the Tesseract desktop app
   (tesseract.md), it renders this same graph in 3D with topic coloring and
   relation edges — no export needed, it reads the same `.markdownvdb/` index.
   An active Shard becomes the app graph's non-removable outer boundary and
   uses its own automatic clusters and independent Topics.

## Related

Use manage-shards to create or repair the named scope and manage-topics to
tune Collection or Shard Topic definitions.
