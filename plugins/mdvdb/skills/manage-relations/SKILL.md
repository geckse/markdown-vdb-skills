---
name: manage-relations
description: >
  Work with typed frontmatter relations between markdown (.md) files in an
  mdvdb vault: author relation fields as quoted wiki-links, resolve them with
  --populate (including reverse referenced_by lookups), declare relation
  columns in the schema overlay, filter by relation, and fix dangling
  relations. Use when the user wants to connect documents via frontmatter
  fields (client, author, project, ...), build relational structure, or asks
  about relations, properties, foreign keys, or "referenced by".
---

# Manage Relations

Frontmatter relations turn link-shaped frontmatter values into typed foreign keys
between markdown files. Only works with `.md` files indexed by mdvdb.

**Input:** `$ARGUMENTS` describes the relation task: a file to inspect, a relation
to author, a field to declare, or a problem to fix.

## What relations are

A frontmatter value whose whole value is a link — `client: "[[clients/acme]]"`,
`author: "[Jane](people/jane.md)"`, or a bare path `project: projects/apollo.md` —
is a **relation**: a typed edge from this file to the target, labeled with the
field name. Relations join the link graph automatically:

- `mdvdb links` / `backlinks` / `graph` entries carry a `field` key — `null` for
  body links, the field name (e.g. `"client"`) for relations. In `links`/
  `backlinks` output, relation entries additionally have `line_number: 0`;
  `graph` edges have no line_number key — use `field != null` there.
- The schema auto-infers `field_type: "Relation"` when all of a field's values
  are link-shaped.
- Relations participate in `--boost-links`, `--expand`, and orphan detection
  like any other link.

## Authoring relations

mdvdb never writes frontmatter — it is read-only over your files. Author
relations with the Edit/Write tools, then re-index with `mdvdb ingest --file <path>`.

**Critical rule: quote wiki-links.** Unquoted `[[x]]` is valid YAML
flow-sequence syntax — it parses as a nested array (`[["x"]]`) instead of a
string, so mdvdb silently detects no relation.

**Good:**
```yaml
---
client: "[[clients/acme]]"                      # quoted wiki-link
author: "[Jane Doe](people/jane.md)"            # markdown link
project: projects/apollo.md                     # bare path
reviewers: ["[[people/jane]]", "[[people/joe]]"] # list = multi-value relation
---
```

**Bad:**
```yaml
---
client: [[clients/acme]]    # unquoted — YAML parses this as [["clients/acme"]], no relation
---
```

**Target resolution** (for frontmatter values): if the value contains `/` it
resolves from the vault root (falling back to the source file's directory);
otherwise a schema-overlay `target:` folder is tried; otherwise the source
file's own directory. `.md` is optional inside wiki and markdown links
(`[[clients/acme]]`, `[Jane](people/jane)`); bare-path values must end in
`.md` to be detected as relations. `[[target|Alias]]` aliases work.

## Reading relations with --populate

```
mdvdb get <path> --populate --json          # single document, depth 1
mdvdb collection <folder> --populate --json # per row on the returned page
mdvdb search "query" --populate --json      # per result, on results[].file
```

`--populate` resolves each relation value into:

```json
{
  "raw": "[[clients/acme|Acme]]",
  "path": "clients/acme.md",
  "exists": true,
  "title": "Acme Corp",
  "frontmatter": { "industry": "manufacturing" }
}
```

- `relations` is a map `field → [RelationValue, ...]` (always arrays, even for
  single values). `path` is `null` if unresolvable; `title`/`frontmatter` are
  `null` when the target doesn't exist. Never nested — depth 1 only.
- `mdvdb get --populate` additionally returns `referenced_by`:
  `[{"source": "invoices/inv-1.md", "field": "client", "title": "Invoice 1"}, ...]`
  — every document pointing at this file through a relation field.

## Filtering by relation

`--filter` on `search` and `collection` is relation-aware: both sides normalize
link syntax before comparing, so all of these match `client: "[[clients/acme|Acme]]"`:

```
mdvdb search "invoice" --filter client=clients/acme --json
mdvdb collection invoices --filter client=clients/acme.md --json
mdvdb collection invoices --filter "client=[[clients/acme]]" --json
```

Matching is purely syntactic (no path resolution): a bare `[[acme]]` value
scoped by an overlay `target: clients` matches `--filter client=acme`, not
`client=clients/acme`.

## Declaring relations in the schema overlay

Create `.markdownvdb.schema.yml` in the vault root to declare relation fields
explicitly (useful before data exists, for `required`, and to scope bare links):

```yaml
fields:
  client:
    field_type: relation   # `type:` is accepted as an alias
    target: clients        # bare [[x]] values resolve into clients/
    required: true
```

A field with `target:` and no explicit type is treated as a relation. The
declared folder appears as `relation_target` in `mdvdb schema --json` and in
`mdvdb collection --json` columns.

## Health checks

`mdvdb doctor --json` includes a **Relations** check that reports:
- **Dangling relations** — relation values whose target file doesn't exist
  (shown as `source#field → target`)
- **Overlay hygiene** — declared `target:` folders that match no indexed files
- **Unquoted wiki-links** — frontmatter values that parsed as a nested array,
  the `[[x]]`-without-quotes footgun

Fix dangling relations by correcting the path or creating the target file, then
`mdvdb ingest --file <path>`.

## Related

The query-collection skill renders relation columns as tables; check-document
audits a single file's relations. If the user mentions "Tesseract" or
"tesseract.md", that's the desktop app companion for mdvdb — its properties
panel, relation chips, and Referenced-by panel display exactly this data from
the same shared `.markdownvdb/` index.
