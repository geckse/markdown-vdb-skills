---
name: explore-topic
description: >
  Deep research on a topic across an mdvdb Collection or named
  Shard/sub-collection by combining semantic search with graph expansion and
  linked context. Use when the user needs a thorough understanding of a
  topic, not just a list of matches, optionally inside one reusable scope.
---

# Explore Topic

Perform deep research by searching with graph expansion to pull in linked context.
Only works with `.md` files indexed by mdvdb.

**Input:** `$ARGUMENTS` is the research topic or question.

## Steps

1. Run an expanded search to get results plus their linked context:
   ```
   mdvdb search "$ARGUMENTS" --json --limit 5 --expand 2 --boost-links --hops 2 --mode hybrid --populate
   ```
   If the user names a Shard, resolve its ID with
   `mdvdb shards list --json` and add `--shard <ID>`. Direct results stay
   inside it; `graph_context` may cross the boundary and must be labeled as
   supplementary **Outside Shard** context.

2. Parse the JSON output. In addition to `results`, look for:
   - `graph_context`: chunks from files linked to the top results; each item
     has `chunk`, `file`, `linked_from`, and `hop_distance`
   - Each result has `score`, `chunk` (with `heading_hierarchy` and `content`),
     and `file` (with `path`, `frontmatter`, and — from `--populate` —
     `relations` resolving typed frontmatter links)

3. If the topic matches a configured Topic name, also check
   `mdvdb clusters --custom --json`, or add `--shard <ID>` for the same Shard
   owner. Summaries contain per-Topic `document_count` and `mean_score`. To
   enumerate member documents with per-document similarity scores, use
   `mdvdb graph --json` with that same optional Shard scope — each node
   carries `custom_cluster_ids` and `custom_cluster_scores` (see
   manage-topics).

4. Identify the most relevant files from both `results` and `graph_context`.
   Read the top 3-5 files using the Read tool to get full content.

5. Synthesize findings into a structured summary:
   - **Key findings**: Main points discovered across the vault
   - **Source files**: Which files contributed what information
   - **Connections**: How the files relate to each other (from graph_context)
   - **Related entities**: relation targets surfaced by `--populate`
     (clients, people, projects referenced in frontmatter)
   - **Gaps**: Aspects of the topic not covered in the vault

6. If the initial search returns few results, try a second search with
   `--mode lexical` to catch keyword matches that semantic search missed.
