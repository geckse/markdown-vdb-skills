---
name: manage-shards
description: >
  Create, inspect, update, retarget, remove, and work inside mdvdb Shards:
  named recursive sub-collection scopes over one shared Collection index.
  Covers nested Shards, Shard-scoped search/tree/info/schema/table/graph
  commands, local automatic clusters, and independent Shard Topics. Use when
  the user mentions a Shard, sub-collection, reusable folder scope,
  Shard-local graph communities or Topics, or wants agents to operate within
  one part of a vault without creating another index.
---

# Manage Shards

Use a Shard as a stable name for a recursive folder scope inside one
Collection.

**Input:** `$ARGUMENTS` is the requested Shard operation, Shard name/ID, path,
or scoped task. Add `--root <vault>` to every command when outside the
Collection root.

## Mental model

- Treat a Shard as a named **sub-collection lens**, not a nested database.
  Keep one `.markdownvdb/` directory, index, watcher, link graph, and set of
  collection-root-relative document identities.
- Infer nesting from folder containment. A document can belong to an ancestor
  and descendant Shard at the same time.
- Keep links, backlinks, relations, `get`, and explicit document navigation
  Collection-wide. A Shard is not an access-control boundary.
- Store definitions only in the project's raw
  `.markdownvdb/config.yaml`. Use the CLI for mutations so unrelated YAML is
  preserved and concurrent writers share the config lock.

## Inspect

Run:

```bash
mdvdb shards list --json
mdvdb shards get <id> --json
```

Read `parent_id` as the inferred nearest ancestor Shard and `exists` as the
current folder state. A missing definition remains valid and repairable.
Shard management works before ingest and never changes the shared index.
For a missing Shard, `list`, `get`, update, and local Topic definition CRUD
remain available. Path-resolved commands fail, while graph/clusters return
empty or disabled local analysis.

## Create

Choose an immutable, unique kebab-case ID and a collection-relative folder:

```bash
mdvdb shards add research \
  --name "Research" \
  --path work/research \
  --json
```

Use `--create-dir` only when the user explicitly wants a missing folder
created:

```bash
mdvdb shards create research-drafts \
  --name "Research Drafts" \
  --path work/research/drafts \
  --create-dir \
  --json
```

Reject root/empty paths, absolute paths, `..`, `.markdownvdb`, duplicate
normalized paths, duplicate names ignoring case, and non-directory targets.
Normalize separators to `/`. Treat symlinked paths as ordinary directories,
not a containment or security guarantee.

## Rename, move, and repair

Edit the display name or one definition's path:

```bash
mdvdb shards update research --name "Research Library" --json
mdvdb shards update research --path archive/research --json
```

An updated path must already be a directory unless `--create-dir` is supplied.
`update <ID> --create-dir` without a new path can recreate the currently
configured missing folder.

After a folder prefix was renamed or moved, retarget the Shard at that prefix
and every nested Shard atomically:

```bash
mdvdb shards retarget work/research archive/research --json
```

`retarget` updates definitions; it does not move the folder. Coordinate the
filesystem operation separately. Tesseract coordinates both when a folder is
renamed in the app and rolls the filesystem rename back if retargeting fails.
For an external rename, keep the missing definition visible until it is
repaired with `update` or `retarget`. The new retarget base must already
exist; descendant targets may remain missing.

Path updates and retargeting invalidate disposable local analysis. They do
not re-ingest moved documents or rewrite root-relative identities, so let the
watcher process the move or run `mdvdb ingest --json` afterwards. Automatic
clusters then rebuild lazily; local Topic centroids may need ingest.

## Remove

```bash
mdvdb shards remove research --json
```

Confirm the target first. Removing a Shard deletes only its definition,
independent local Topic definitions, and best-effort disposable analysis
cache. It never deletes the folder, Markdown files, embeddings, links, or
shared index. `delete` is an alias.

## Run scoped commands

Use the immutable ID rather than repeating a path:

```bash
mdvdb search "vector databases" --shard research --json
mdvdb tree --shard research --json
mdvdb info --shard research --json
mdvdb schema --shard research --json
mdvdb collection --shard research --recursive --json
mdvdb graph --shard research --json
mdvdb modules run formula --shard research --json
mdvdb modules status formula --shard research --json
mdvdb modules run lookup_rollup --shard research --json
mdvdb modules status lookup_rollup --shard research --json
```

For computed modules, a Shard limits the requested output owners; it is not a
dependency boundary. A Lookup/Rollup run may refresh Formula inputs outside
the Shard, and a Formula run may update downstream Lookup/Rollup owners beyond
it. Manual reports list the dependency-aware module order. Do not assume that
every safely affected file is physically inside the selected Shard.

Except for graph projection, do not combine `--shard` with a path selector.
`collection --shard` still returns direct children by default; add
`--recursive` for the complete Shard.

Do not add `--shard` to `ingest`, `watch`, `status`, `doctor`, `get`, `links`,
`backlinks`, `orphans`, or `edges`. Ingest, watching, diagnostics, index
status, orphans, and vault-wide semantic edges remain Collection-wide;
document commands keep explicit root-relative paths.

## Use local graph analysis and Topics

Automatic clusters are computed lazily from only the Shard's existing stored
document vectors:

```bash
mdvdb clusters --shard research --json
mdvdb graph --shard research --json
```

They reuse Collection clustering settings, while Leiden automatically lowers
the effective neighbor count for small Shards so a local graph is not forced
fully connected. They do not re-embed documents or change index bytes.
Read-only analysis can atomically create or refresh the disposable sidecar
`.markdownvdb/cache/shards/<ID>.json`; it never stores embeddings or graph
topology and does not change `mdvdb status`.

Define independent local Topics with the same Topic commands:

```bash
mdvdb clusters --shard research add Methods \
  --description "Research methods and experiments" \
  --seeds methodology,experiment \
  --threshold 0.35 \
  --json
mdvdb ingest --json
mdvdb graph --shard research --json
mdvdb clusters --shard research --custom --json
mdvdb clusters --shard research unassigned --json
```

Collection, ancestor, sibling, and descendant Topic definitions do not
inherit. New or changed definitions need the next Collection-wide ingest to
compute centroids; until then graph analysis can report
`topics: "needs_ingest"` while topology and automatic clusters remain usable.
Verify `graph.analysis.topics` becomes `ready`: Shard Topic refresh failure is
non-fatal to the overall ingest, and empty `--custom`/`unassigned` arrays alone
do not prove readiness.

For graph only, combine the Shard with a descendant path to project visible
topology without changing the Shard-wide cluster or Topic identities:

```bash
mdvdb graph --shard research --path work/research/drafts --json
```

The graph is a strict induced subgraph. Search expansion may still return
linked supplementary context outside the Shard; do not present that context
as a direct in-Shard result.

An existing Shard folder can still contain zero indexed documents because
ignore and source rules remain Collection-owned.

## Report

State the immutable Shard ID, display name, normalized path, inferred parent,
folder existence, and action taken. For analysis, state that clusters and
Topics are Shard-local and call out `too_small`, `none`, `needs_ingest`, or
error states rather than treating them as an empty Collection.

## Related

Use search-docs for scoped retrieval, query-collection for a recursive Shard
table, graph-visualize for local topology, and manage-topics for detailed
Topic tuning.
