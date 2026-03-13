---
name: search-and-summarize
description: >
  Search the markdown (.md) vault for a topic via mdvdb, read the top matching
  files in full, and produce a comprehensive cited synthesis.
allowed-tools:
  - Bash(mdvdb *)
  - Read
context: fork
disable-model-invocation: true
---

# Search and Summarize

Search the vault, read the best matches, and synthesize a comprehensive answer.
Only works with `.md` files indexed by mdvdb.

**Input:** `$ARGUMENTS` is the research question or topic.

## Steps

1. Search the vault with link boosting:
   ```
   mdvdb search "$ARGUMENTS" --json --limit 8 --mode hybrid --boost-links
   ```

2. Identify the top-scoring files (score > 0.5 or top 5, whichever is more).

3. Read each identified file in full using the Read tool. Use the `file` field
   from each search result as the path.

4. For each file, note:
   - Main points relevant to the query
   - Key data, definitions, or arguments
   - How it relates to the other files

5. Produce a structured synthesis:

   **Answer**: A direct, comprehensive answer to the research question,
   drawing on all sources read.

   **Key Points**: Bulleted list of the most important findings.

   **Sources**: For each claim, cite the source file and section.
   Format: `[filename.md#Section Heading]`

   **Contradictions or Gaps**: Note any conflicting information between
   sources, or aspects of the question not covered by the vault.

6. Only include information actually found in the vault files.
   Do not add external knowledge unless explicitly asked.
