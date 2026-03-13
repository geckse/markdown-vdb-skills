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
   - **Most connected nodes**: top 5 files by total degree (in + out edges)
   - **Isolated nodes**: files with zero connections
   - **Clusters**: if cluster data is present, group nodes by cluster

4. If the graph is small (under 30 nodes), present a text-based adjacency
   summary showing which files connect to which.

5. For larger graphs, focus on statistics and the most important hub nodes.

6. Mention that the raw JSON can be saved for external tools:
   `mdvdb graph --json > graph.json`
