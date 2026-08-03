---
name: enhance-document
description: >
  Improve an existing markdown (.md) file for better mdvdb indexing. Applies
  markdown best practices for frontmatter, typed relations, File attachment
  fields, and read-only Formula/Lookup/Rollup outputs, plus heading structure,
  chunking, and link connectivity. Use when a document needs better metadata,
  structure, attachments, or connections — or when the user asks how to write
  good markdown for mdvdb.
---

# Enhance Document

Improve a markdown file so mdvdb can index, search, chunk, and link it optimally.
This skill doubles as a reference for writing markdown that works well with mdvdb.

**Input:** `$ARGUMENTS` is the path to the markdown file to enhance.

## Markdown Guidelines for mdvdb

These are the rules that matter for how mdvdb parses, chunks, embeds, and searches
your files. Follow them when enhancing a document.

### Frontmatter

mdvdb reads YAML frontmatter between `---` fences. Ordinary properties are
user-authored metadata used for filtering and schema inference. Formula,
Lookup, and Rollup properties are the exception: their modules safely
materialize owned results into frontmatter, and users must treat those outputs
as read-only.

**Rules:**
- Use proper YAML types: arrays for tags (`[design, api]`), not comma-separated strings
- Dates in ISO format: `2024-03-15`, not `March 15`
- Omit fields rather than setting them to `null`
- Keep field names consistent across the vault — mdvdb infers a schema from all files
- Common useful fields: `title`, `tags`, `status`, `type`, `created`, `updated`
- Every field you add becomes a filter dimension: `mdvdb search "x" --filter status=draft`
- Links to Markdown documents are **relations**; links with explicit
  non-Markdown extensions are **File** values — see below
- Never manually add, change, or remove a schema field whose type is `Formula`,
  `Lookup`, or `Rollup`; fix its inputs, dependency, or overlay definition and
  let the owning module reconcile it

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

### Relations (typed frontmatter links)

A frontmatter value that is entirely a link to a Markdown document becomes a
**relation** — a typed edge labeled with the field name. Relations are
filterable (`--filter client=clients/acme`), resolvable (`--populate`), and
appear in the link graph. Prefer plain `.md` paths in frontmatter because they
are easier to read and write. Wiki links and Markdown links remain supported
for existing content. Wiki-link values that remain must be quoted: unquoted
`[[x]]` is valid YAML but parses as a nested array instead of a string, so
mdvdb detects no relation.

**Good:**
```yaml
client: clients/acme.md
authors:
  - people/jane.md
  - people/joe.md
project: projects/apollo.md
```

**Bad:**
```yaml
client: [[clients/acme]]     # unquoted — parses as a nested array, no relation detected
```

See the manage-relations skill for resolution rules, the schema overlay
(`target:` folders), and health checks.

### File attachment fields

Use a `File` field for collection-local, non-Markdown resources that belong to
the document without becoming knowledge-graph nodes: images, PDFs, media,
office files, archives, and other physical files. Canonical YAML is always a
list:

```yaml
attachments:
  - "[[assets/mockup.png]]"
  - "[[documents/spec.pdf]]"
```

Quote every wiki link and keep paths relative to the vault root. Do not use
absolute paths, URLs, `..`, or Markdown targets. An explicit non-Markdown
extension infers `File` in mdvdb ≥ 0.2.0; add `field_type: file` to
`.markdownvdb.schema.yml` for empty lists and extensionless/ambiguous
filenames. Do not add `target:`. File values are raw frontmatter only:
`--populate`, backlinks, orphan checks, and the Markdown graph intentionally
ignore them.

### Headings and Chunking

mdvdb splits documents by headings. Each heading starts a new chunk. Chunks that
exceed `chunking.max_tokens` (default 512; set via
`mdvdb config set chunking.max_tokens 512`, or the `MDVDB_CHUNK_MAX_TOKENS` env
override) are sub-split with overlap windows.
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
every sub-chunk sharing the same generic heading label instead of a
specific per-topic one)

#### Error Codes

(skipped H3 — broken hierarchy)
```

### Links and Graph Connectivity

mdvdb extracts markdown links and wikilinks to build a link graph. Links affect
search ranking (`--boost-links`), enable graph traversal (`mdvdb links --depth 1-3`,
`mdvdb backlinks`), and identify orphans. More links = better search results.
Frontmatter relations (above) join the same graph as typed edges.

**Rules:**
- Use `[[wikilinks]]` for quick inline references to other vault files
- Use `[descriptive text](path/to/file.md)` for contextual references
- Link to related topics, not just sequential pages
- Section-specific links work: `[[auth#Token Refresh]]` or `[see refresh](auth.md#token-refresh)`
- Aim for no orphans — every file should link to or be linked from at least one other file
- Body links (wikilinks and markdown links) resolve relative to the containing
  file's directory — use `../` to reach files in other folders. Only
  frontmatter relation values containing `/` resolve from the vault root first

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
- Which fields are `Formula`, `Lookup`, or `Rollup` outputs? Exclude them from
  missing-field suggestions. Check `computed_field_errors` and module status
  instead of manufacturing a value.

Also check sibling files for context:
```
mdvdb tree --path <parent-dir> --json
```

### 3. Check link health

```
mdvdb links "$ARGUMENTS" --json
mdvdb backlinks "$ARGUMENTS" --json
mdvdb get "$ARGUMENTS" --populate --json
```

Assess connectivity:
- Does this file link to related content?
- Do other files link back to it?
- Is it an orphan?
- Do its relations resolve? (`relations` entries with `exists: false` are
  dangling); who references it via relation fields (`referenced_by`)?

### 4. Find link candidates

Search for content related to this document's topic:
```
mdvdb search "<key terms from document>" --json --limit 10 --mode hybrid
```

Also check for orphans that might benefit from a link:
```
mdvdb orphans --json
```

And check the folder's relation columns — fields with `relation_target` set
that this file lacks are candidates for new relation fields:
```
mdvdb collection <parent-dir> --json
```

### 5. Enhance the document

Apply improvements using the Edit tool. Never change the document's meaning —
only improve its structure and metadata.

**Frontmatter:** Add missing ordinary schema fields, fix types, and convert tag
strings to arrays. Preserve every module-owned Formula/Lookup/Rollup output
byte-for-byte; do not add a missing output or delete a stale-looking one by
hand.

**Relations:** Convert prose references into typed frontmatter relations where
the folder has a field convention (from step 4's columns). Write plain `.md`
paths by default; preserve supported wiki/Markdown link syntax unless changing
that relation, and always quote wiki-link values that remain.

**Files:** Keep attachment values as quoted, root-relative wiki links in a
list; normalize any legacy scalar File value when editing it. Never delete a
physical file merely because its frontmatter reference is removed.

**Headings:** Fix hierarchy, break oversized sections, make heading text descriptive.

**Links:** Add links to related documents found via search, link orphans where relevant.

**Content:** Front-load key terms, add brief prose before code blocks, ensure consistent terminology.

### 6. Report changes

Summarize what was improved:
- Frontmatter fields added or fixed
- Relations added or repaired
- File attachment fields normalized or repaired
- Computed outputs left module-owned; report any diagnostics separately
- Headings restructured
- Links added (and to which documents)
- Suggest re-indexing: `mdvdb ingest --file <path>`, then verify with
  `mdvdb get <path> --populate --json` (all relations should show `exists: true`
  and `computed_field_errors` should be empty). If not, inspect
  `mdvdb modules status formula --json` and
  `mdvdb modules status lookup_rollup --json`, fix the dependency or
  definition, and rerun the owning module rather than editing its output.
