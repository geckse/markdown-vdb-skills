---
name: manage-relations
description: >
  Work with typed frontmatter relations and relation-backed Lookup and Rollup
  computed fields in an mdvdb vault: author readable .md path relations,
  resolve them with --populate, declare relation and computed columns in the
  schema overlay, aggregate related values, and diagnose failures. Use when
  the user asks about relations, lookups, rollups, aggregates, properties,
  foreign keys, reverse referenced_by links, or derived values from related
  documents.
---

# Manage Relations

Frontmatter relations turn links to Markdown documents into typed foreign keys
between indexed `.md` files.

**Input:** `$ARGUMENTS` describes the relation task: a file to inspect, a relation
to author, a field to declare, or a problem to fix.

## What relations are

A frontmatter value whose whole value is a Markdown path or link —
`client: clients/acme.md` (preferred), `client: "[[clients/acme]]"`, or
`author: "[Jane](people/jane.md)"` — is a **relation**: a typed edge from this
file to the target, labeled with the field name. Relations join the link graph
automatically:

- `mdvdb links` / `backlinks` / `graph` entries carry a `field` key — `null` for
  body links, the field name (e.g. `"client"`) for relations. In `links`/
  `backlinks` output, relation entries additionally have `line_number: 0`;
  `graph` edges have no line_number key — use `field != null` there.
- The schema auto-infers `field_type: "Relation"` when all values link to
  Markdown or extensionless targets.
- Relations participate in `--boost-links`, `--expand`, and orphan detection
  like any other link.

## Relation versus File

Do not use a Relation for a physical non-Markdown attachment. In mdvdb ≥ 0.2.0,
a whole-value wiki or Markdown link with an explicit non-Markdown extension
infers as `field_type: "File"`:

```yaml
attachments:
  - "[[assets/mockup.png]]"
  - "[[documents/spec.pdf]]"
```

File fields are collection-local file references, not document foreign keys.
They never create graph edges, backlinks, `referenced_by`, or populated
relations. `--populate` leaves them only in raw frontmatter. Keep them as a
YAML list even for zero or one file. Explicitly pin extensionless or otherwise
ambiguous targets with `field_type: file`; never give a File field `target:`
or `relation_target`.

## Authoring relations

Ordinary Relation values are user-authored. Author them with the Edit/Write
tools, then re-index with `mdvdb ingest --file <path>`. Formula, Lookup, and
Rollup outputs are the exception: their modules safely materialize only the
owned output properties into frontmatter. Never edit those output values by
hand.

**Default to plain `.md` paths.** They are the easiest form for people to read
and write, require no YAML quoting, and work as scalars or list items. Wiki
links and Markdown links remain supported. When using wiki links, quote them:
unquoted `[[x]]` is valid YAML flow-sequence syntax and parses as a nested
array (`[["x"]]`) instead of a string, so mdvdb detects no relation.

**Good:**
```yaml
---
client: clients/acme.md
author: people/jane.md
project: projects/apollo.md
reviewers:
  - people/jane.md
  - people/joe.md
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
file's own directory. Plain paths must end in `.md` to be detected as
relations. `.md` is optional inside wiki and Markdown links
(`[[clients/acme]]`, `[Jane](people/jane)`), and `[[target|Alias]]` aliases
work.

## Reading relations with --populate

```
mdvdb get <path> --populate --json          # single document, depth 1
mdvdb collection <folder> --populate --json # per row on the returned page
mdvdb search "query" --populate --json      # per result, on results[].file
```

`--populate` resolves each relation value into:

```json
{
  "raw": "clients/acme.md",
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
link syntax before comparing, so all of these match `client: clients/acme.md`:

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
    target: clients        # filename-only values such as acme.md resolve into clients/
    required: true
```

A field with `target:` and no explicit type is treated as a relation. The
declared folder appears as `relation_target` in `mdvdb schema --json` and in
`mdvdb collection --json` columns.

### Restricting relation choices to a folder

`target:` is also the relation picker's candidate scope in Tesseract. When it
is set, the picker lists and searches only markdown documents in that folder
and its subfolders. Without it, the picker falls back to vault-wide
recents/search.

- The target is a vault-root-relative **folder** path, such as `groups` or
  `crm/groups`; do not add a trailing slash.
- Configure it in `.markdownvdb.schema.yml`, not in each document's
  frontmatter. Frontmatter stores only the selected relation value.
- A top-level `fields:` declaration applies vault-wide. Use `scopes:` when the
  same field should target that folder only for a particular source
  folder/table:

  ```yaml
  scopes:
    people:
      fields:
        groups:
          field_type: relation
          target: groups
  ```

- In Tesseract's Database Table View: open the relation column's `⋮` menu,
  choose **Property settings…**, set **Target folder**, and save. The app
  writes the same overlay declaration.
- `target:` scopes filename-only values such as `engineering.md`. A relation
  value that already contains `/`, such as `groups/engineering.md`, remains
  an explicit vault-root-relative path. Extensionless wiki links remain
  supported for compatibility.

## Lookup and Rollup computed fields

Define computed outputs in `.markdownvdb.schema.yml`, not in document YAML.
For example, Contacts can copy the current Client domain while Clients sum the
Formula totals of every Invoice that points back to them:

```yaml
scopes:
  contacts:
    fields:
      client:
        field_type: relation
        target: clients
      client_domain:
        field_type: lookup
        relation_field: client
        target_field: domain

  invoices:
    fields:
      client:
        field_type: relation
        target: clients
      total:
        field_type: formula
        formula: subtotal + tax
        result_type: number

  clients:
    fields:
      invoice_total:
        field_type: rollup
        relation_direction: incoming
        relation_scope: invoices
        relation_field: client
        target_field: total
        formula: values.reduce((sum, value) => sum + value, 0)
        result_type: number
```

- `relation_field` must name an ordinary `Relation` field on the owner for
  outgoing traversal or on source documents for incoming traversal. Put
  `target:` on that Relation definition; never put it on Lookup or Rollup.
- `target_field` is one exact top-level frontmatter key. Dotted/nested paths
  are unsupported.
- A computed output name cannot be `title`, `path`, or its own
  `relation_field` key. Lookup rejects Rollup-only `formula`, `result_type`,
  and `relation_scope`; outgoing Rollup also rejects `relation_scope`.
- Lookup is outgoing-only. A scalar Relation produces a scalar; a Relation
  list produces an ordered list preserving duplicates and nested JSON. A
  missing/null Relation produces `null`; an empty Relation list produces `[]`.
- Rollup defaults to outgoing traversal and evaluates its sandboxed `formula`
  over `values`. Incoming Rollup requires `relation_direction: incoming` and
  `relation_scope`, which is the vault-root-relative source **folder** to scan.
  Incoming sources are deduplicated and path-sorted before evaluation.
- An explicit target `null` is valid. A malformed or unresolved Relation, an
  unreadable target, or a missing target key fails the whole output; never
  silently omit a bad member from an aggregate.
- Lookup/Rollup output properties are read-only and module-owned. Successful
  values are materialized into Markdown frontmatter and are authoritative for
  get/search/collection/filter/sort. `computed_fields` only mirrors successful
  provenance; `computed_field_errors` takes precedence when evaluation fails.
- Formula runs before `lookup_rollup`, so Lookup/Rollup may select Formula or
  acyclic Lookup/Rollup outputs. Formula cannot depend on Lookup/Rollup.
  Dependency cycles fail closed rather than reusing an old value.
- Computed values that look like links never become Relations, graph edges,
  backlinks, or populated values.

The standard presets are ordinary editable formulas:

```text
Sum:     values.reduce((sum, value) => sum + value, 0)
Count:   values.length
Average: values.length ? values.reduce((sum, value) => sum + value, 0) / values.length : null
Minimum: values.length ? values.reduce((minimum, value) => Math.min(minimum, value)) : null
Maximum: values.length ? values.reduce((maximum, value) => Math.max(maximum, value)) : null
```

Sum and Count return `0` for empty `values`; Average, Minimum, and Maximum
return `null`. Rollup accepts `null` as no aggregate, while every non-null
result must match `result_type`.

Validate a Rollup expression, reconcile definitions, and inspect persisted
diagnostics with:

```bash
mdvdb modules validate lookup_rollup \
  --formula 'values.reduce((sum, value) => sum + value, 0)' \
  --result-type number --json
mdvdb modules run lookup_rollup --json
mdvdb modules status lookup_rollup --json
```

There is no public CLI CRUD command for computed definitions. When editing the
overlay directly:

1. Preserve an exact byte-for-byte backup of `.markdownvdb.schema.yml`.
2. Use `modules validate lookup_rollup` only to validate a Rollup expression;
   it does not validate the complete overlay definition.
3. Atomically write the overlay, then run the unscoped
   `mdvdb modules run lookup_rollup --json` reconciliation. Do not limit a
   definition mutation to `--path` or `--shard`.
4. Inspect every `module_reports[].diagnostics` entry, not only the flattened
   requested-module report. Record-level dependency/data diagnostics keep the
   valid definition and remain visible for repair.
5. On a command, `invalid_schema`, or module-level failure, restore the exact
   backup and rerun the previous definitions. Report both the primary and any
   restoration failure.

Rename or remove only the overlay definition; never batch-rename or delete its
materialized values in Markdown. The module cleans only values it can prove it
owns and reports unsafe or concurrent changes instead of overwriting them.

## Health checks

`mdvdb doctor --json` includes a **Relations** check that reports:
- **Dangling relations** — relation values whose target file doesn't exist
  (shown as `source#field → target`)
- **Overlay hygiene** — declared `target:` folders that match no indexed files
- **Unquoted wiki-links** — frontmatter values that parsed as a nested array,
  the `[[x]]`-without-quotes footgun

Fix dangling relations by correcting the path or creating the target file, then
`mdvdb ingest --file <path>`.

Computed failures are reported separately by
`mdvdb modules status lookup_rollup --json` and in each document's
`computed_field_errors`. Fix the Relation, target field, formula, or dependency
cycle, then rerun the module; do not type a replacement output value.

## Related

The query-collection skill renders Relation, Lookup, and Rollup columns;
check-document audits one file's relations and computed diagnostics. If the
user mentions "Tesseract" or "tesseract.md", that's the desktop app companion
for mdvdb — its properties panel, table indicators, relation chips, and
Referenced-by panel use the same Markdown and `.markdownvdb/` index.
