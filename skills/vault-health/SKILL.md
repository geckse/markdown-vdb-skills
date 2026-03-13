---
name: vault-health
description: >
  Run diagnostic health checks on the markdown (.md) vault via mdvdb: doctor
  checks, orphan detection, and schema analysis. Use when the user reports
  issues or wants to audit their knowledge base.
allowed-tools:
  - Bash(mdvdb *)
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
   ```

2. Present a health report:

   **Doctor Checks** (from `doctor`):
   - Config validity, embedding provider connectivity, index integrity
   - List any failing checks with error messages

   **Orphan Files** (from `orphans`):
   - Files with zero incoming or outgoing links
   - Orphan count vs. total file count (percentage)

   **Schema Analysis** (from `schema`):
   - Metadata fields found across the vault
   - Field types and coverage
   - Inconsistencies (e.g., a field that varies in type across files)

3. Provide a health summary:
   - **Healthy**: all checks pass, low orphan rate, consistent schema
   - **Warning**: minor issues (some orphans, optional checks failing)
   - **Critical**: provider unreachable, index corrupt, many errors

4. For each issue found, suggest a specific fix.
