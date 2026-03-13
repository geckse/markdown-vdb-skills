---
name: find-related
description: >
  Given a markdown (.md) file, find all related content via semantic edges,
  outgoing links, backlinks, and multi-hop traversal using mdvdb. Use when
  the user wants to understand what connects to a specific document.
---

# Find Related Content

Discover everything related to a specific markdown file using the link graph and semantic edges.
Only works with `.md` files indexed by mdvdb.

**Input:** `$ARGUMENTS` is the path to a markdown file (relative to project root).

## Steps

1. Run these commands in parallel to gather relationship data:
   ```
   mdvdb links "$ARGUMENTS" --depth 2 --json
   mdvdb backlinks "$ARGUMENTS" --json
   mdvdb edges "$ARGUMENTS" --json
   mdvdb get "$ARGUMENTS" --json
   ```

2. Parse each JSON output:
   - **links**: outgoing links and multi-hop neighborhood at depth 2
   - **backlinks**: files that link INTO this file
   - **edges**: semantic relationships with relationship labels and strength
   - **get**: file metadata including frontmatter, hash, chunk count

3. Present a unified relationship map:
   - **This file**: title, tags, last modified
   - **Outgoing links**: files this document references
   - **Incoming links**: files that reference this document
   - **Semantic neighbors**: files connected via semantic edges (with relationship type and strength)
   - **2-hop connections**: files reachable through intermediaries

4. Highlight particularly interesting connections (high-strength edges,
   unexpected backlinks).
