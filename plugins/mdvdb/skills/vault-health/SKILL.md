---
name: vault-health
description: >
  Run diagnostic health checks on the markdown (.md) vault via mdvdb: doctor
  checks (including relation integrity), orphan detection, and schema
  analysis. Use when the user reports issues or wants to audit their
  knowledge base.
---

# Vault Health Check

Comprehensive diagnostics for the markdown vault.
Only works with `.md` files indexed by mdvdb.

## Steps

1. Run these commands in parallel:
   ```
   mdvdb doctor --json
   mdvdb orphans --json
   mdvdb schema --json
   mdvdb status --json
   mdvdb config --json
   ```
   (`config` shows the fully resolved configuration — provider, model,
   clustering, chunking — after merging shell env, project YAML, `.env`
   secrets, and user YAML; use it to see current values before suggesting
   `config set` fixes.)

2. Present a health report:

   **Doctor Checks** (from `doctor` — 8 checks):
   - Config loads (`.markdownvdb/config.yaml` — YAML; legacy dotenv configs
     auto-migrate on first load)
   - Embedding provider reachable, index integrity, file discovery
   - **Relations check**: dangling relation targets (frontmatter links to
     files that don't exist, shown as `source#field → target`), overlay
     `target:` folders matching no files, and unquoted `[[x]]` frontmatter
     values (the YAML nested-array footgun)
   - List any failing checks with error messages

   **Orphan Files** (from `orphans`):
   - Files with zero incoming or outgoing links (frontmatter relations count
     as links)
   - Orphan count vs. total file count (percentage)

   **Schema Analysis** (from `schema`):
   - Metadata fields found across the vault, with types and coverage
   - `Relation` fields (link-shaped values) and their `relation_target` folder
     if declared in the `.markdownvdb.schema.yml` overlay
   - Inconsistencies (e.g., a field that varies in type across files —
     `Mixed` type)

3. Provide a health summary:
   - **Healthy**: all checks pass, low orphan rate, consistent schema
   - **Warning**: minor issues (some orphans, dangling relations, optional
     checks failing)
   - **Critical**: provider unreachable, index corrupt, many errors

4. For each issue found, suggest a specific fix:
   - Unquoted wiki-link → quote it: `client: "[[clients/acme]]"`
   - Dangling relation → fix the path or create the target file, then
     `mdvdb ingest --file <path>` (see the manage-relations skill)
   - Config problems → adjust via `mdvdb config set <dotted.key> <value>`
