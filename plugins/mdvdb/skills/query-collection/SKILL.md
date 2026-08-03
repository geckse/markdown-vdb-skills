---
name: query-collection
description: >
  Query a folder or named Shard/sub-collection of markdown (.md) files as a
  database table via mdvdb: rows are files, columns are frontmatter fields.
  Supports filtering, sorting, pagination, resolving relations with
  --populate, reading raw File attachment columns, and current materialized
  Formula, Lookup, and Rollup values. Use when the user wants to list, filter,
  or tabulate documents in a folder or Shard — "show all drafts", "list
  clients", "which posts are tagged rust" — or run a Notion/Dataview-style
  table query.
---

# Query Collection

Treat a folder as a database table: files are rows, frontmatter fields are columns.
Only works with `.md` files indexed by mdvdb.

**Input:** `$ARGUMENTS` is a folder path or Shard ID plus optional constraints
in natural language (filters, sort order, limit).

## Steps

1. Build the query from `$ARGUMENTS`:
   ```
   mdvdb collection <path> --json
   ```
   For a configured recursive Shard, use its immutable ID instead of a path:
   ```
   mdvdb collection --shard research --recursive --json
   ```
   Resolve IDs with `mdvdb shards list --json`. Do not combine `--shard` with
   the positional path selector.

   Add flags as the request requires:
   - `-r` / `--recursive` — include nested subfolders (default: direct children only)
   - `--sort <field>` — sort by a frontmatter field (default: by path)
   - `--order asc|desc` — sort direction (default `asc`)
   - `-f KEY=VALUE` / `--filter KEY=VALUE` — repeatable, combined with AND.
     Equality only. Relation-aware: `-f client=clients/acme` matches
     `client: "[[clients/acme|Acme]]"`. Array containment: `-f tags=rust`
     matches `tags: [rust, go]`.
   - `--limit N` / `--offset N` — pagination
   - `--populate` — resolve relation fields on the returned rows

   Notes: `list` is an alias for `collection`; path defaults to `.` (whole
   vault). A Shard still returns direct children by default, so retain
   `--recursive` when the user means its complete sub-collection.

2. Parse the JSON output:
   ```json
   {
     "scope": "blog/",
     "recursive": false,
     "columns": [
       {"name": "status", "field_type": "String", "occurrence_count": 12,
        "sample_values": ["draft"], "required": false, "in_schema": true,
        "relation_target": null}
     ],
     "rows": [
       {"path": "blog/post.md", "title": "Post", "frontmatter": {},
        "modified_at": 1770000000, "state": "indexed"}
     ],
     "total_rows": 12,
     "offset": 0
   }
   ```
   - `columns[]` is the folder's table definition. Columns with
     `field_type: "Relation"` are relation columns; `relation_target` names
     the overlay-declared target folder when configured (null otherwise).
     Columns with `field_type: "File"` contain collection-root-relative
     non-Markdown references; their `relation_target` is always null.
     `Lookup` columns expose `relation_field`, `target_field`, and effective
     `relation_direction: "Outgoing"`. `Rollup` also exposes `formula`,
     `result_type`, its effective direction, and `relation_scope` only for an
     incoming Rollup; that scope names its source folder.
     `required` comes from the schema overlay; `in_schema` means the column
     is part of the folder's schema rather than discovered only from a row.
   - `rows[].frontmatter` stays unpopulated; with `--populate`,
     `rows[].relations` holds resolved values
     (`{field: [{raw, path, exists, title, frontmatter}]}`).
     File values stay in `rows[].frontmatter` as string arrays and never enter
     `rows[].relations`.
   - Successful Formula/Lookup/Rollup values are materialized in
     `rows[].frontmatter`; that is the authoritative query/display value.
     `rows[].computed_fields` is only a successful-value/provenance mirror.
     If `rows[].computed_field_errors` contains a field, show that diagnostic
     and never fall back to an older value read directly from disk.
   - `total_rows` counts matches after filtering but before `--limit`/`--offset` —
     use it for "showing X of Y".

3. Render a markdown table. Pick columns the user asked for, or the highest
   `occurrence_count` ones. For relation cells show the target's `title` when
   `exists` is true; otherwise show the `raw` value and flag it as dangling.
   For File cells, parse each quoted `[[path]]` from the raw array and show
   the root-relative path or basename. mdvdb does not populate or thumbnail
   physical files; check the collection filesystem when existence matters.
   Render Lookup using its actual scalar/list/object JSON shape and Rollup
   using its declared `result_type`. Both use the ordinary filter and sort
   flags because their current results are materialized frontmatter. A
   link-shaped computed result is still not a Relation and never appears in
   `rows[].relations`.

4. To drill into a row, use `mdvdb get <row.path> --populate --json`. Row
   `state` is one of `indexed`, `modified`, `new`, or `deleted`; rows with
   `new` or `modified` are not fully indexed yet — suggest `mdvdb ingest`
   if search over them matters.

## Tesseract value colors

Tesseract automatically assigns theme/accent-aware palette colors to Select
(`allowed_values`) and Tags chips. Explicit choices are synced in the schema
overlay beside the field:

```yaml
scopes:
  invoices:
    fields:
      status:
        allowed_values: [draft, paid]
        value_colors:
          draft: 2
          paid: "neutral:7"
```

Numeric values are accent-palette slots `0`–`23`. Muted, accent-tinted
neutral brightness steps use `"neutral:N"` with slots `0`–`11`. These are
palette references, not fixed hex colors, so both palettes adapt to the active
theme/accent. Preserve `value_colors` when editing
`.markdownvdb.schema.yml`; current mdvdb CLI versions safely ignore this
frontend annotation.

## Related

Use manage-shards for named scope lifecycle, search-docs for content search,
and manage-relations for authoring relation columns. If the user mentions
"Tesseract" or "tesseract.md", that's the desktop app companion for mdvdb —
**Open Shard as Table** renders this same recursive Shard from the shared
`.markdownvdb/` index; selecting a Shard alone does not open a table.
