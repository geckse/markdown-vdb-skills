---
name: search-docs
description: >
  Search indexed markdown (.md) files using semantic, lexical, or hybrid search
  via mdvdb. Use when you need to find information across a markdown vault.
allowed-tools:
  - Bash(mdvdb *)
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
   If the user specified a mode, limit, filter, or path scope in their request,
   add the corresponding flags: `--mode`, `--limit`, `--filter KEY=VALUE`, `--path PREFIX`.

2. Parse the JSON output. The structure is:
   ```json
   {
     "results": [
       {
         "file": "path/to/file.md",
         "section": "## Heading",
         "score": 0.87,
         "snippet": "...",
         "metadata": {}
       }
     ],
     "query": "...",
     "total_results": N,
     "mode": "hybrid"
   }
   ```

3. Present results clearly: file path, section heading, score, and a brief snippet.
   If zero results, suggest checking the index with `mdvdb status`.

4. If the user wants to read a specific result, use the file path to read the markdown file.
