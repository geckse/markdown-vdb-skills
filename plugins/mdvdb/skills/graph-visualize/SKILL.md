---
name: graph-visualize
description: >
  Export and summarize the knowledge graph of the markdown (.md) vault via
  mdvdb for visualization or analysis. Use when the user wants to see or
  analyze the vault's link structure and document relationships.
---

# Graph Visualize

Export and summarize the vault's knowledge graph.
Only works with `.md` files indexed by mdvdb.

**Input:** `$ARGUMENTS` can be:
- Empty: full document-level graph
- A path prefix (e.g., `docs/`): restrict to that subtree
- `chunk`: use chunk-level granularity instead of document-level

## Steps

1. Determine the command flags from `$ARGUMENTS`:
   - If a path is specified: add `--path PREFIX`
   - If "chunk" is mentioned: add `--level chunk`
   - Default: `--level document`

2. Run the graph export:
   ```
   mdvdb graph --json --level document
   ```
   (Add `--path PREFIX` or `--level chunk` as needed.)

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

4. For semantic relationship analysis, `mdvdb edges --json` (no file argument)
   lists all vault-wide semantic edges, and `--relationship <label>` filters
   them by relationship-label substring — useful for "show every pair
   connected by X"-style questions.

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
