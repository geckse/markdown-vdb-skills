# mdvdb Skills for Claude Code

Claude Code skills for [markdown-vdb](https://github.com/geckse/markdown-vdb) — a filesystem-native vector database for Markdown files.

## Installation

Clone this repo into your project's skills directory:

```bash
git clone https://github.com/geckse/markdown-vdb-skills.git skills
```

Or add as a git submodule:

```bash
git submodule add https://github.com/geckse/markdown-vdb-skills.git skills
```

Requires `mdvdb` to be installed and available on your `PATH`.

## Skills

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

## Usage

Once installed, skills are available as slash commands in Claude Code:

```
/search-docs query about your topic
/vault-overview
/index-vault
```

## License

MIT
