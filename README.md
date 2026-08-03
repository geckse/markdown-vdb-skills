# mdvdb — Claude Code Plugin

Claude Code plugin for [markdown-vdb](https://github.com/geckse/markdown-vdb) — a filesystem-native vector database for Markdown files with named Shards (recursive sub-collections) over one shared index.

## Installation

Add the marketplace and install:

```
/plugin marketplace add geckse/markdown-vdb-skills
/plugin install mdvdb@mdvdb-skills
```

Or test locally:

```bash
claude --plugin-dir ./plugins/mdvdb
```

Requires `mdvdb` **≥ 0.2.0** installed and available on your `PATH` (check with
`mdvdb --version`).

All commands accept `--root <vault>` when the working directory is not the vault — every skill's command works from anywhere by adding it.

## Skills

Once installed, Claude automatically picks the right skill based on what you ask.

| Skill | Description |
|---|---|
| `search-docs` | Search indexed markdown files using semantic, lexical, or hybrid search — with frontmatter/relation filters and `--populate` |
| `search-and-summarize` | Search for a topic, read top matches in full, and produce a cited synthesis |
| `explore-topic` | Deep research combining semantic search with graph expansion and linked context |
| `find-related` | Find all related content via typed relations, semantic edges, links, backlinks, and multi-hop traversal |
| `manage-shards` | Create and use named recursive sub-collections with scoped search, tables, graph analysis, automatic clusters, and independent Topics |
| `query-collection` | Query a folder as a database table: rows are files, columns are frontmatter fields, with filters, sorting, relations, and raw File attachment columns |
| `manage-relations` | Author, resolve, filter, and repair typed frontmatter relations between Markdown documents; distinguish them from non-Markdown File fields |
| `manage-topics` | Create, tune, and inspect topics (custom clusters): thresholds, seeds, and the Unassigned bucket |
| `index-vault` | Ingest or re-index markdown files into the vector database, with cost estimates from `mdvdb info` |
| `vault-overview` | Quick situational awareness: index status, vault stats, clusters, topics, and file tree |
| `vault-health` | Diagnostic checks: doctor (incl. relation integrity), orphan detection, and schema analysis |
| `check-document` | Validate a file against the vault schema, check structure, relations, and link connectivity |
| `enhance-document` | Improve a file for better indexing: add frontmatter, relations, restructure headings, add links |
| `write-document` | Create a new markdown file optimized for indexing with proper frontmatter, relations, and links |
| `graph-visualize` | Export and summarize the vault's knowledge graph for visualization or analysis |

## Tesseract

[Tesseract](https://tesseract.md) is the local-first desktop app companion for mdvdb: a markdown editor with named Shard sub-collections, a 3D knowledge graph, scoped topics, relations UI, File attachment tiles, and a folder-as-table database view. Get it at [https://tesseract.md](https://tesseract.md) — it is developed in its own separate repository. It runs the same `mdvdb` CLI against the same vault — shared `.markdownvdb/` index and `config.yaml` — so everything these skills do via CLI is immediately visible in the app, and vice versa. Tesseract also imports Obsidian tags and graph color groups as Collection topics. When a user mentions "Tesseract" or "tesseract.md", they mean this app operating on the same vault.

## Structure

```
.claude-plugin/
  marketplace.json                  # Marketplace catalog
plugins/
  mdvdb/
    .claude-plugin/plugin.json      # Plugin manifest
    skills/
      search-docs/SKILL.md
      manage-shards/SKILL.md
      explore-topic/SKILL.md
      ...
```

## License

MIT
