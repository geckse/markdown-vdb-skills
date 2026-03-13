---
name: index-vault
description: >
  Ingest or re-index markdown (.md) files into the mdvdb vector database.
  Runs a preview first for safety, then ingests. Must be explicitly invoked
  since it modifies the index.
disable-model-invocation: true
---

# Index Vault

Safely ingest markdown files with a preview-first approach.
Only works with `.md` files.

**Input:** `$ARGUMENTS` can be:
- Empty: incremental ingest of changed files
- `--reindex`: force full re-embedding of all files
- `--file path/to/file.md`: ingest a single specific file
- `preview`: only show preview, do not ingest

## Steps

1. Always start with a preview to show what will happen:
   ```
   mdvdb ingest --preview --json
   ```
   If the user specified `--reindex`, add that flag.
   If the user specified `--file PATH`, add that flag.

2. Display the preview:
   - Files that will be indexed (new or changed)
   - Files that will be skipped (unchanged)
   - Total embedding API calls that will be made

3. If `$ARGUMENTS` is just "preview", stop here.

4. Otherwise, proceed with the actual ingest:
   ```
   mdvdb ingest --json
   ```
   (Add `--reindex` or `--file PATH` if specified.)

5. Report the ingest results:
   - Files indexed, skipped, removed
   - Chunks created
   - Duration
   - Any errors encountered

6. If there were errors, suggest running `mdvdb doctor` to diagnose.
