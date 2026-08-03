---
name: check-document
description: >
  Validate a markdown (.md) file against the vault's schema, check heading
  structure, verify frontmatter relations, File attachment fields, and
  Formula/Lookup/Rollup diagnostics, and assess link connectivity using mdvdb.
  Reports missing authored fields, dangling relations, computed dependency
  failures, broken attachments, orphan status, broken links, and structural
  issues. Use when auditing document quality.
---

# Check Document

Audit a markdown file for mdvdb best practices: schema compliance, heading
structure, relation integrity, and link connectivity.

**Input:** `$ARGUMENTS` is the path to the markdown file to check.
If empty, checks the entire vault for common issues.

## Single File Check

### 1. Read and parse the document

Read the file. Note its frontmatter fields, heading structure, and links.

### 2. Schema compliance and relations

```
mdvdb schema --path <parent-dir> --json
mdvdb get "$ARGUMENTS" --populate --json
mdvdb modules status formula --path <parent-dir> --json
mdvdb modules status lookup_rollup --path <parent-dir> --json
```

(Scoping the schema to the file's folder avoids false "missing field" findings
from other folders' schemas; drop `--path` for a whole-vault view.)

Report:
- **Missing authored fields**: ordinary schema fields not present in this
  file's frontmatter (ordinary fields marked `required` are hard failures).
  Do not report an absent Formula/Lookup/Rollup output as something to type in.
- **Type mismatches**: fields with values that don't match the schema type
  (e.g., string where number expected, non-ISO date format; `Relation` fields
  should link to Markdown documents; `File` fields should be arrays of quoted,
  collection-root-relative links to non-Markdown files)
- **Invalid values**: fields with values outside `allowed_values` if defined
- **Extra fields**: fields in this file not in the vault schema (may be intentional)
- **Dangling relations**: entries in the populated `relations` map with
  `exists: false` — the target file doesn't exist
- **Broken File references**: resolve each raw File value against the vault
  root and report missing files. Reject absolute paths, URLs, `..`, Markdown
  targets, and paths outside the vault. mdvdb does not include File values in
  `--populate`; inspect the raw frontmatter and filesystem directly.
- **Non-canonical File values**: legacy scalar values remain readable, but
  edited File fields must use a YAML list even for zero or one file
- **Unquoted wiki-links**: a frontmatter value that parsed as a nested array
  (e.g. `[["clients/acme"]]`) signals an unquoted `[[x]]` — valid YAML but
  not a string, so no relation is detected. Replace it with a plain `.md` path
  (preferred) or quote the wiki link
- **Computed failures**: inspect `computed_field_errors` from `get` and both
  module-status responses. A missing relation target/key, type error,
  dependency failure/cycle, or refused concurrent write means the output is
  unavailable; never accept or restore a stale materialized value manually.
  Successful values in `frontmatter` are authoritative; `computed_fields` is
  only a provenance mirror.

### 3. Heading structure

Analyze the heading hierarchy:
- **Skipped levels**: e.g., H1 → H3 with no H2 (the chunk's heading hierarchy
  is missing an intermediate level)
- **No headings**: document has no headings (one section, sub-split only by
  the token size guard at arbitrary boundaries, with no heading context)
- **Oversized sections**: sections likely exceeding `chunking.max_tokens`
  (default 512) will be sub-split
- **Missing preamble**: no content before the first heading

### 4. Link connectivity

```
mdvdb links "$ARGUMENTS" --json
mdvdb backlinks "$ARGUMENTS" --json
```

Report:
- **Outgoing links**: count and targets; entries with a non-null `field` are
  frontmatter relations, `field: null` are body links
- **Incoming links**: count and sources; typed incoming references also appear
  as `referenced_by` in step 2's populated output
- **Orphan status**: true if zero total links (in + out)
- **Broken links**: links to files that don't exist in the vault

### 5. Summary

Present a checklist:
- [ ] Frontmatter has all required schema fields
- [ ] Field types match schema
- [ ] Relation fields resolve to existing files (no dangling relations)
- [ ] File fields are canonical lists and every referenced physical file exists
- [ ] Formula/Lookup/Rollup outputs are current or have an actionable
      diagnostic, and no computed output was manually edited
- [ ] Relation values use plain `.md` paths by default; wiki links that remain
      are quoted
- [ ] Headings use proper hierarchy (no skipped levels)
- [ ] Sections are reasonably sized for chunking
- [ ] Document has outgoing links to related content
- [ ] Document has incoming backlinks (not orphaned)

## Vault-Wide Check

If no file is specified, run a broad audit:

```
mdvdb doctor --json
mdvdb orphans --json
mdvdb schema --json
mdvdb info --json
mdvdb modules status formula --json
mdvdb modules status lookup_rollup --json
```

Report:
- Doctor findings, including the Relations check (dangling targets, overlay
  hygiene, unquoted wiki-links)
- Total orphan count and list (files with no links)
- Schema inconsistencies (Mixed-type fields, low-coverage fields)
- Formula/Lookup/Rollup diagnostics, grouped by path and output field; fix the
  definition or dependency and rerun the module instead of authoring outputs
- Sync state from `info` (`sync.new` / `sync.changed` files need `mdvdb ingest`)
