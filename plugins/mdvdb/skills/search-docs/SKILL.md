---
name: search-docs
description: >
  Search indexed markdown (.md) files using semantic, lexical, or hybrid search
  via mdvdb. Supports frontmatter and relation filters, score thresholds, time
  decay, link-hop boosting, and resolving relations with --populate. Use when
  you need to find information across a markdown vault.
---

# Search Docs

Search the markdown vault using `mdvdb search`. Only works with `.md` files indexed by mdvdb.

**Input:** `$ARGUMENTS` is the search query. Optionally include flags like
`--mode semantic`, `--limit 10`, `--filter tag=design`, `--path docs/`.

## Steps

1. Run the search command with JSON output:
   ```
   mdvdb search "$ARGUMENTS" --json --limit 10
   ```
   Add flags matching the user's request:
   - `--mode hybrid|semantic|lexical` (shorthands: `--semantic`, `--lexical`);
     `--edge-search` searches link-context edge embeddings instead — its
     matches arrive in a top-level `edge_results` array (`score`, `edge_id`,
     `source_path`, `target_path`, `link_text`, `context`) while `results`
     stays empty
   - `--min-score 0.4` — drop weak matches (0.0–1.0)
   - `--filter KEY=VALUE` — repeatable, combined with AND. Relation-aware:
     `--filter client=clients/acme` matches `client: "[[clients/acme|Acme]]"`.
     Array containment: `--filter tags=rust` matches `tags: [rust, go]`
   - `--path PREFIX` — restrict to files under a path
   - `--boost-links` — link-graph ranking boost; add `--hops 2` (1–3) to extend
     the boost across multi-hop neighbors
   - `--expand 2` (0–3) — include chunks from linked files as `graph_context`
   - `--decay` / `--no-decay`, `--decay-half-life <days>`,
     `--decay-include`/`--decay-exclude <comma-separated-prefixes>` — time decay
   - `--populate` — resolve frontmatter relations on each result's file

2. Parse the JSON output. The structure is:
   ```json
   {
     "results": [
       {
         "score": 0.87,
         "chunk": {
           "chunk_id": "path/to/file.md#0",
           "heading_hierarchy": ["Document Title", "Section"],
           "content": "...",
           "start_line": 1,
           "end_line": 20
         },
         "file": {
           "path": "path/to/file.md",
           "frontmatter": {},
           "file_size": 1234,
           "modified_at": 1770000000
         }
       }
     ],
     "query": "...",
     "total_results": 5,
     "mode": "hybrid"
   }
   ```
   (Example abridged — `file` also carries `path_components`.) With
   `--expand`, a `graph_context` array is added (chunks from linked files);
   with `--populate`, each `file` gains a `relations` map
   (`{field: [{raw, path, exists, title, frontmatter}]}`).

3. Present results clearly: file path (`file.path`), section (join
   `chunk.heading_hierarchy` with " > "), score, and a brief excerpt from
   `chunk.content`. If zero results, suggest checking the index with
   `mdvdb status` or `mdvdb info` (counts of new/changed files; `mdvdb tree`
   identifies which files they are).

4. If the user wants to read a specific result, use `file.path` to read the
   markdown file.

For tabular folder listings use the query-collection skill; for relation
details use the manage-relations skill.
