---
name: enhance-document
description: >
  Improve an existing markdown (.md) file for better mdvdb indexing. Applies
  markdown best practices for frontmatter, heading structure, chunking, and
  link connectivity. Use when a document needs better metadata, structure,
  or connections — or when the user asks how to write good markdown for mdvdb.
---

# Enhance Document

Improve a markdown file so mdvdb can index, search, chunk, and link it optimally.
This skill doubles as a reference for writing markdown that works well with mdvdb.

**Input:** `$ARGUMENTS` is the path to the markdown file to enhance.

## Markdown Guidelines for mdvdb

These are the rules that matter for how mdvdb parses, chunks, embeds, and searches
your files. Follow them when enhancing a document.

### Frontmatter

mdvdb reads YAML frontmatter between `---` fences. It never writes to your files —
frontmatter is read-only metadata used for filtering and schema inference.

**Rules:**
- Use proper YAML types: arrays for tags (`[design, api]`), not comma-separated strings
- Dates in ISO format: `2024-03-15`, not `March 15`
- Omit fields rather than setting them to `null`
- Keep field names consistent across the vault — mdvdb infers a schema from all files
- Common useful fields: `title`, `tags`, `status`, `type`, `created`, `updated`
- Every field you add becomes a filter dimension: `mdvdb search "x" --filter status=draft`

**Good:**
```yaml
---
title: Authentication Flow
tags: [auth, security, api]
status: published
created: 2024-03-15
---
```

**Bad:**
```yaml
---
title: Authentication Flow
tags: auth, security, api    # string, not array — can't filter by tag
date: March 15               # not ISO — inconsistent type
category: null               # don't set null, just omit
---
```

### Headings and Chunking

mdvdb splits documents by headings. Each heading starts a new chunk. Chunks that
exceed `MDVDB_CHUNK_MAX_TOKENS` (default 512) are sub-split with overlap windows.
Heading text becomes the chunk's searchable label in results.

**Rules:**
- Start with one `# H1` title (matches the document's purpose)
- Use proper hierarchy: H1 → H2 → H3. Never skip levels (H1 → H3)
- Each section should be roughly 100–500 tokens (a few paragraphs)
- Break large sections into subsections — don't dump 2000 words under one heading
- Make headings descriptive and specific: `## OAuth2 Token Refresh` not `## Details`
- Add a brief preamble paragraph before the first H2 (it becomes its own chunk)
- Avoid empty sections (heading with no content underneath)

**Good structure:**
```markdown
# Authentication Guide

Brief overview of the auth system and what this document covers.

## OAuth2 Flow

How the OAuth2 flow works in our system...

### Token Refresh

When tokens expire, the refresh mechanism...

### Error Handling

Common auth errors and how to handle them...

## API Key Authentication

Alternative auth method for service-to-service calls...
```

**Bad structure:**
```markdown
# Auth

## Details

(2000 words of content under one heading — will be sub-split with overlap,
losing heading context)

#### Error Codes

(skipped H2 and H3 — broken hierarchy)
```

### Links and Graph Connectivity

mdvdb extracts markdown links and wikilinks to build a link graph. Links affect
search ranking (`--boost-links`), enable graph traversal (`mdvdb links`, `backlinks`,
`neighborhood`), and identify orphans. More links = better search results.

**Rules:**
- Use `[[wikilinks]]` for quick inline references to other vault files
- Use `[descriptive text](path/to/file.md)` for contextual references
- Link to related topics, not just sequential pages
- Section-specific links work: `[[auth#Token Refresh]]` or `[see refresh](auth.md#token-refresh)`
- Aim for no orphans — every file should link to or be linked from at least one other file
- Links use relative paths from the project root

**Good:**
```markdown
This uses the same token format as [[oauth2-spec]].
For error handling, see [Authentication Errors](errors/auth-errors.md).
Related: [[api-keys]], [[session-management#Expiry]]
```

### Content Quality for Search

mdvdb embeds chunk content as vectors and indexes it for BM25 lexical search.
Both benefit from clear, specific writing.

**Rules:**
- Front-load key terms in paragraphs — first sentence matters most for BM25
- Use the same terminology consistently (don't alternate between "auth" and "authentication" for the same concept)
- Include specific terms and names — semantic search matches meaning, lexical search matches words
- Avoid walls of code without prose explanation — embeddings work better on natural language
- Tables and lists are fine — they're included in chunk text

## Steps

### 1. Read the current document

Read the file to understand its current state: frontmatter, headings, content, links.

### 2. Check the vault schema

```
mdvdb schema --json
```

Compare the document's frontmatter against the vault schema:
- What fields exist in the schema but are missing from this document?
- Are field values consistent with the schema's types?
- Are dates in ISO format? Are tags proper YAML arrays?

Also check sibling files for context:
```
mdvdb tree --path <parent-dir> --json
```

### 3. Check link health

```
mdvdb links "$ARGUMENTS" --json
mdvdb backlinks "$ARGUMENTS" --json
```

Assess connectivity:
- Does this file link to related content?
- Do other files link back to it?
- Is it an orphan?

### 4. Find link candidates

Search for content related to this document's topic:
```
mdvdb search "<key terms from document>" --json --limit 10 --mode hybrid
```

Also check for orphans that might benefit from a link:
```
mdvdb orphans --json
```

### 5. Enhance the document

Apply improvements using the Edit tool. Never change the document's meaning —
only improve its structure and metadata.

**Frontmatter:** Add missing schema fields, fix types, convert tag strings to arrays.

**Headings:** Fix hierarchy, break oversized sections, make heading text descriptive.

**Links:** Add links to related documents found via search, link orphans where relevant.

**Content:** Front-load key terms, add brief prose before code blocks, ensure consistent terminology.

### 6. Report changes

Summarize what was improved:
- Frontmatter fields added or fixed
- Headings restructured
- Links added (and to which documents)
- Suggest re-indexing: `mdvdb ingest --file <path>`
