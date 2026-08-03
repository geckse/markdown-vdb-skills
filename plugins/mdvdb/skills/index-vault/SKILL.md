---
name: index-vault
description: >
  Ingest or re-index markdown (.md) files into the mdvdb vector database.
  Checks cost with mdvdb info, previews index/embedding work, then ingests and
  reports Formula/Lookup/Rollup materialization diagnostics. Must be explicitly
  invoked because it modifies the index and may safely update module-owned
  computed frontmatter.
disable-model-invocation: true
---

# Index Vault

Safely ingest markdown files with a cost-check and preview-first approach.
Only works with `.md` files.

**Input:** `$ARGUMENTS` can be:
- Empty: incremental ingest of changed files
- `--reindex`: force full re-embedding of all files
- `--file path/to/file.md`: ingest a single specific file
- `preview`: only show preview, do not ingest

## Steps

1. If the vault has no `.markdownvdb/` directory yet, bootstrap it first:
   `mdvdb init`, then configure the embedding provider via
   `mdvdb config set embedding.provider openai` (or `ollama`) and put the API
   key in the vault's `.env` (`OPENAI_API_KEY=...`) — never in YAML.

2. Check the vault state and cost (free, offline, read-only):
   ```
   mdvdb info --json
   ```
   Key fields: `sync` (`{new, changed, unchanged, deleted}` — what an
   incremental ingest would touch), `reindex_chunks` /
   `reindex_estimated_tokens` / `reindex_estimated_api_calls` (full-reindex
   cost), `embedding` (provider, model, dimensions), `index_file_size`,
   `last_updated`. Scope to a folder with `mdvdb info <folder>`.
   If `sync.new`, `sync.changed`, and `sync.deleted` are all 0 and no
   `--reindex` was requested, report that the index is already up to date and
   stop (`deleted > 0` means an incremental ingest will remove stale entries —
   no embedding cost).

3. Preview discovery, chunking, and embedding work:
   ```
   mdvdb ingest --preview --json
   ```
   If the user specified `--reindex`, add that flag.
   If the user specified `--file PATH`, add that flag.

   Preview does not execute Formula or Lookup/Rollup and therefore does not
   enumerate their frontmatter patches. Treat it as an index/API-cost preview,
   not a computed-write preview. Actual module writes remain per-document,
   atomic, ownership-checked, and conflict-safe.

4. Display the preview:
   - Files that will be indexed (new or changed)
   - Files that will be skipped (unchanged)
   - Total embedding API calls that will be made

5. If `$ARGUMENTS` is just "preview", stop here.

6. Otherwise, proceed with the actual ingest:
   ```
   mdvdb ingest --json
   ```
   (Add `--reindex` or `--file PATH` if specified.)

7. Report the ingest results:
   - Files indexed, skipped, removed
   - Chunks created
   - Duration
   - Any errors encountered
   - Ordered `module_reports` (`formula` before `lookup_rollup`), including
     files evaluated, fields updated, and every path/field diagnostic

8. If there were ingest errors, suggest `mdvdb doctor`. For computed failures,
   inspect `mdvdb modules status formula --json` and
   `mdvdb modules status lookup_rollup --json`. Fix the input, Relation,
   target field, formula, or dependency and rerun the relevant module; never
   type, batch-copy, or delete its materialized output.

Formula and `lookup_rollup` are always-on. Their successful values are
materialized into Markdown frontmatter without re-embedding an unchanged body;
frontmatter is authoritative for queries, while `computed_fields` is only a
provenance mirror and `computed_field_errors` suppresses failed stale values.

For extended editing sessions, `mdvdb watch` re-indexes changed files
automatically (run it as a background process) instead of repeated per-file
ingest calls.

Note: if the user runs the Tesseract desktop app (tesseract.md) on this vault,
it shares the same `.markdownvdb/` index and can watch and index files itself —
step 1's `sync` counts reveal whether files are already indexed, and any ingest
done here is immediately visible in the app.
