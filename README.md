# mdvdb — Claude Code Plugin

Claude Code plugin for [markdown-vdb](https://github.com/geckse/markdown-vdb) — a filesystem-native vector database for Markdown files.

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

Requires `mdvdb` to be installed and available on your `PATH`.

## Skills

Once installed, Claude automatically picks the right skill based on what you ask.

| Skill | Description |
|---|---|
| `search-docs` | Search indexed markdown files using semantic, lexical, or hybrid search |
| `search-and-summarize` | Search for a topic, read top matches in full, and produce a cited synthesis |
| `explore-topic` | Deep research combining semantic search with graph expansion and linked context |
| `find-related` | Find all related content via semantic edges, links, backlinks, and multi-hop traversal |
| `index-vault` | Ingest or re-index markdown files into the vector database |
| `vault-overview` | Quick situational awareness: index status, clusters, and file tree |
| `vault-health` | Diagnostic checks: doctor, orphan detection, and schema analysis |
| `check-document` | Validate a file against the vault schema, check structure and link connectivity |
| `enhance-document` | Improve a file for better indexing: add frontmatter, restructure headings, add links |
| `write-document` | Create a new markdown file optimized for indexing with proper frontmatter and links |
| `graph-visualize` | Export and summarize the vault's knowledge graph for visualization or analysis |

## Structure

```
.claude-plugin/
  marketplace.json                  # Marketplace catalog
plugins/
  mdvdb/
    .claude-plugin/plugin.json      # Plugin manifest
    skills/
      search-docs/SKILL.md
      explore-topic/SKILL.md
      ...
```

## License

MIT
