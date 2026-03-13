---
name: explore-topic
description: >
  Deep research on a topic across markdown (.md) files by combining semantic
  search with graph expansion and linked context via mdvdb. Use when the user
  needs a thorough understanding of a topic, not just a list of matches.
allowed-tools:
  - Bash(mdvdb *)
  - Read
---

# Explore Topic

Perform deep research by searching with graph expansion to pull in linked context.
Only works with `.md` files indexed by mdvdb.

**Input:** `$ARGUMENTS` is the research topic or question.

## Steps

1. Run an expanded search to get results plus their linked context:
   ```
   mdvdb search "$ARGUMENTS" --json --limit 5 --expand 2 --boost-links --mode hybrid
   ```

2. Parse the JSON output. In addition to `results`, look for:
   - `graph_context`: chunks from files linked to the top results
   - Each result has `file`, `section`, `score`, `snippet`, `metadata`

3. Identify the most relevant files from both `results` and `graph_context`.
   Read the top 3-5 files using the Read tool to get full content.

4. Synthesize findings into a structured summary:
   - **Key findings**: Main points discovered across the vault
   - **Source files**: Which files contributed what information
   - **Connections**: How the files relate to each other (from graph_context)
   - **Gaps**: Aspects of the topic not covered in the vault

5. If the initial search returns few results, try a second search with
   `--mode lexical` to catch keyword matches that semantic search missed.
