---
name: write-document
description: >
  Create a new markdown (.md) document optimized for mdvdb indexing. Generates
  proper YAML frontmatter based on the vault's schema, structured headings for
  chunking, and wiki/markdown links to related files for graph connectivity.
  Use when creating new documents in an mdvdb vault.
---

# Write Document

Create a new markdown file optimized for mdvdb search, filtering, and graph connectivity.

**Input:** `$ARGUMENTS` is the topic, title, or description of the document to create.
Optionally include a target path (e.g., `docs/guides/setup.md`).

## Steps

### 1. Understand the vault's schema

Run the schema command to learn what frontmatter fields exist across the vault:
```
mdvdb schema --json
```

This reveals the "relational database" of metadata fields: their types, how many
files use each field, sample values, and any allowed value constraints. Different
folders may have different schemas (e.g., `/blog/` has date/author/tags,
`/people/` has role/team/email).

If a target path was given, also check what files are nearby:
```
mdvdb tree --path <parent-dir> --json
```

### 2. Find related content for linking

Search the vault for content related to the new document's topic:
```
mdvdb search "<topic>" --json --limit 10 --mode hybrid
```

These results will become link candidates — connecting the new document into the
vault's knowledge graph.

### 3. Write the document

Create the `.md` file with these components:

**YAML Frontmatter** (between `---` markers, at the very top):
- Include fields that match the vault's schema for this directory
- Use consistent types: strings, numbers, booleans, arrays, ISO dates
- Date format: `YYYY-MM-DD` or `YYYY-MM-DDTHH:MM:SS`
- Tags as YAML arrays, not comma-separated strings
- Omit fields that don't apply (don't use `null`)
- These fields become search filters: `mdvdb search --filter status=published`

```yaml
---
title: Document Title
date: 2024-01-15
tags:
  - topic-a
  - topic-b
status: draft
category: guides
---
```

**Headings** (structure for chunking):
- Each heading creates a new searchable chunk
- Use proper hierarchy: H1 > H2 > H3 (don't skip levels)
- Keep sections focused: ~300-500 tokens per section is ideal
- Heading text becomes searchable section metadata
- A preamble paragraph before the first heading becomes its own chunk

```markdown
# Main Title

Brief introduction paragraph (this becomes chunk #0).

## First Section

Focused content on one aspect...

### Subsection

Deeper detail...

## Second Section

Another focused topic...
```

**Links** (build the knowledge graph):
- Use wiki links for quick internal references: `[[other-document]]`
- Use markdown links for contextual references: `[setup guide](../guides/setup.md)`
- Link to related documents found in step 2
- Link bidirectionally when it makes sense (A links to B, B links to A)
- Section-specific links work: `[[document#Section Heading]]`
- External URLs (https://) are ignored by the graph — only internal .md links matter
- Every link strengthens the graph and improves search via `--boost-links`

```markdown
Related to [[project-alpha]] and the [[architecture]] overview.
See [detailed implementation](./implementation.md#core-logic) for code.
```

**Content best practices:**
- Standard markdown: bold, italic, lists, code blocks, tables, blockquotes
- Mermaid diagrams supported in fenced code blocks
- Math expressions: `$inline$` and `$$block$$`
- Task lists: `- [x] done` / `- [ ] todo`
- Callouts: `> [!note]`, `> [!important]`, `> [!warning]`

### 4. Verify the document

After writing, suggest the user run:
- `mdvdb ingest --file <path>` to index the new document
- `mdvdb get <path>` to verify frontmatter was parsed correctly
- `mdvdb links <path>` to confirm links were extracted
