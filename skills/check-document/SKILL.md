---
name: check-document
description: >
  Validate a markdown (.md) file against the vault's schema, check heading
  structure, and assess link connectivity using mdvdb. Reports missing
  frontmatter fields, orphan status, broken links, and structural issues.
  Use when auditing document quality.
allowed-tools:
  - Bash(mdvdb *)
  - Read
---

# Check Document

Audit a markdown file for mdvdb best practices: schema compliance, heading
structure, and link connectivity.

**Input:** `$ARGUMENTS` is the path to the markdown file to check.
If empty, checks the entire vault for common issues.

## Single File Check

### 1. Read and parse the document

Read the file. Note its frontmatter fields, heading structure, and links.

### 2. Schema compliance

```
mdvdb schema --json
mdvdb get "$ARGUMENTS" --json
```

Report:
- **Missing fields**: schema fields not present in this file's frontmatter
- **Type mismatches**: fields with values that don't match the schema type
  (e.g., string where number expected, non-ISO date format)
- **Invalid values**: fields with values outside `allowed_values` if defined
- **Extra fields**: fields in this file not in the vault schema (may be intentional)

### 3. Heading structure

Analyze the heading hierarchy:
- **Skipped levels**: e.g., H1 → H3 with no H2 (breaks chunk hierarchy)
- **No headings**: document has no headings (entire content = one chunk)
- **Oversized sections**: sections likely exceeding 512 tokens (will be sub-split)
- **Missing preamble**: no content before the first heading

### 4. Link connectivity

```
mdvdb links "$ARGUMENTS" --json
mdvdb backlinks "$ARGUMENTS" --json
```

Report:
- **Outgoing links**: count and targets
- **Incoming links**: count and sources
- **Orphan status**: true if zero total links (in + out)
- **Broken links**: links to files that don't exist in the vault

### 5. Summary

Present a checklist:
- [ ] Frontmatter has all required schema fields
- [ ] Field types match schema
- [ ] Headings use proper hierarchy (no skipped levels)
- [ ] Sections are reasonably sized for chunking
- [ ] Document has outgoing links to related content
- [ ] Document has incoming backlinks (not orphaned)

## Vault-Wide Check

If no file is specified, run a broad audit:

```
mdvdb orphans --json
mdvdb schema --json
mdvdb status --json
```

Report:
- Total orphan count and list (files with no links)
- Schema inconsistencies (Mixed-type fields, low-coverage fields)
- Unindexed files that need `mdvdb ingest`
