---
name: find-related
description: >
  Given a markdown (.md) file, find all related content via typed frontmatter
  relations, semantic edges, outgoing links, backlinks, and multi-hop
  traversal using mdvdb. Use when the user wants to understand what connects
  to a specific document.
---

# Find Related Content

Discover everything related to a specific markdown file using the link graph,
frontmatter relations, and semantic edges.
Only works with `.md` files indexed by mdvdb.

**Input:** `$ARGUMENTS` is the path to a markdown file (relative to project root).

## Steps

1. Run these commands in parallel to gather relationship data:
   ```
   mdvdb links "$ARGUMENTS" --json
   mdvdb links "$ARGUMENTS" --depth 2 --json
   mdvdb backlinks "$ARGUMENTS" --json
   mdvdb edges "$ARGUMENTS" --json
   mdvdb get "$ARGUMENTS" --populate --json
   ```

2. Parse each JSON output:
   - **links** (depth 1): direct links as `links.outgoing[].entry` and
     `links.incoming[]` objects. Each entry has a `field` key — `null` for
     body links, a frontmatter field name (e.g. `"client"`) for typed
     relations (those also have `line_number: 0`)
   - **links --depth 2**: a multi-hop neighborhood tree of
     `{path, state, children}` nodes with reach counts (no per-link `field`
     info at this level)
   - **backlinks**: files that link INTO this file (same `entry.field`
     distinction as depth-1 links)
   - **edges**: semantic relationships with relationship labels and strength
   - **get --populate**: file metadata plus
     `relations` — map `field → [{raw, path, exists, title, frontmatter}]` —
     and `referenced_by` — `[{source, field, title}]`, documents pointing at
     this file through a relation field

3. Present a unified relationship map:
   - **This file**: title, tags, last modified
   - **Typed relations (frontmatter)**: field name → target title/path;
     flag any with `exists: false` as dangling
   - **Referenced by**: source file + the relation field it uses
   - **Outgoing links**: body links this document makes
   - **Incoming links**: files that reference this document in their body
   - **Semantic neighbors**: files connected via semantic edges (with
     relationship type and strength)
   - **2-hop connections**: files reachable through intermediaries

4. Highlight particularly interesting connections (high-strength edges,
   unexpected backlinks, dangling relations that need fixing — see the
   manage-relations skill).
