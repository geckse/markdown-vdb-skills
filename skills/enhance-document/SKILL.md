---
name: enhance-document
description: >
  Improve an existing markdown (.md) file for better mdvdb indexing: add missing
  frontmatter fields based on the vault schema, restructure headings for optimal
  chunking, and add links to related documents for graph connectivity. Use when
  a document needs better metadata, structure, or connections.
allowed-tools:
  - Bash(mdvdb *)
  - Read
  - Edit
---

# Enhance Document

Improve an existing markdown file's frontmatter, heading structure, and links
for better search, filtering, and graph connectivity in mdvdb.

**Input:** `$ARGUMENTS` is the path to the markdown file to enhance.

## Steps

### 1. Read the current document

Read the file to understand its current state: frontmatter, headings, content, links.

### 2. Check the vault schema

```
mdvdb schema --json
```

Compare the document's frontmatter against the vault schema:
- What fields exist in the schema but are missing from this document?
- Are field values consistent with the schema's types and allowed values?
- Are dates in ISO format (`YYYY-MM-DD`)?
- Are tags proper YAML arrays, not comma-separated strings?

Also check what sibling files look like for context:
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
- Is it an orphan (no links in or out)?

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

**Frontmatter improvements:**
- Add missing schema fields with appropriate values
- Fix type inconsistencies (e.g., string date → ISO date)
- Convert comma-separated tags to YAML arrays
- Remove `null` values (omit the field instead)

**Heading improvements:**
- Ensure proper hierarchy (H1 > H2 > H3, no skipped levels)
- Break oversized sections into subsections (aim for ~300-500 tokens per section)
- Add a preamble paragraph before the first heading if missing
- Make heading text descriptive (they become searchable chunk labels)

**Link improvements:**
- Add links to related documents found via search
- Use `[[wiki links]]` for quick references within the text
- Use `[text](path.md)` for contextual references
- Suggest links to/from orphan documents where relevant
- Add section-specific links where useful: `[[doc#Section]]`

### 6. Report changes

Summarize what was improved:
- Frontmatter fields added or fixed
- Headings restructured
- Links added (and to which documents)
- Suggest re-indexing: `mdvdb ingest --file <path>`
