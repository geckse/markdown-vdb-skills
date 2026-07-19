---
name: query-collection
description: >
  Query a folder of markdown (.md) files as a database table via mdvdb: rows
  are files, columns are frontmatter fields. Supports filtering, sorting,
  pagination, and resolving relation fields with --populate. Use when the
  user wants to list, filter, or tabulate documents in a folder — "show all
  drafts", "list clients", "which posts are tagged rust" — or any
  Notion/Dataview-style table query over the vault.
---

# Query Collection

Treat a folder as a database table: files are rows, frontmatter fields are columns.
Only works with `.md` files indexed by mdvdb.

**Input:** `$ARGUMENTS` is a folder path plus optional constraints in natural
language (filters, sort order, limit).

## Steps

1. Build the query from `$ARGUMENTS`:
   ```
   mdvdb collection <path> --json
   ```
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

   Notes: `list` is an alias for `collection`; path defaults to `.` (whole vault).

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
     `required` comes from the schema overlay; `in_schema` means the column
     is part of the folder's schema rather than discovered only from a row.
   - `rows[].frontmatter` stays raw; with `--populate`, `rows[].relations`
     holds resolved values (`{field: [{raw, path, exists, title, frontmatter}]}`).
   - `total_rows` counts matches after filtering but before `--limit`/`--offset` —
     use it for "showing X of Y".

3. Render a markdown table. Pick columns the user asked for, or the highest
   `occurrence_count` ones. For relation cells show the target's `title` when
   `exists` is true; otherwise show the `raw` value and flag it as dangling.

4. To drill into a row, use `mdvdb get <row.path> --populate --json`. Row
   `state` is one of `indexed`, `modified`, `new`, or `deleted`; rows with
   `new` or `modified` are not fully indexed yet — suggest `mdvdb ingest`
   if search over them matters.

## Related

Use search-docs for content search, manage-relations for authoring relation
columns. If the user mentions "Tesseract" or "tesseract.md", that's the desktop
app companion for mdvdb — its database table view renders this same folder from
the same shared `.markdownvdb/` index.
