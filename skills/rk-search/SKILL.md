---
name: rk-search
description: "Search the codebase knowledge base with progressive cache and semantic matching. Agent reads the index, uses language understanding to find the best match (even for vague queries like '窗口大小' matching 'getWindowRect'), verifies relevance, and falls back to source code only when cache cannot answer. Every new finding is cached with aliases that grow smarter over time. Use when user asks to find code, understand how something works, or get usage examples."
---

# RK Search — Semantic Knowledge Base Search

Search the knowledge base using Agent-powered semantic matching with progressive alias learning.

## Input
- Search query (natural language, any language)

## Step 1: Select Project

Before searching, determine which project to search:

1. Read `~/.repo-knowledge/_registry.md`
2. If **no projects** exist → tell user: "No knowledge bases found. Use `/repo-knowledge:rk-create <git-url>` to create one."
3. If **only 1 project** → use it automatically, tell user which project you're searching
4. If **multiple projects** → list them and ask the user to pick one before proceeding

## Step 2: Search

After project is determined, launch a **searcher Agent** with this prompt:

```
You are a Knowledge Base Search Agent.

Search query: "{user_query}"
Project: "{project_name}" (or "all" to search all projects)
Cache base path: "~/.repo-knowledge/"

## Search Strategy (follow strictly, do NOT skip layers)

### Layer 1: Scan Aliases in _index.md
1. Read `~/.repo-knowledge/{project_name}/_index.md`
2. Look at the `aliases:` field of each entry
3. If any alias closely matches the user's query, go directly to that doc
4. If match found → go to Layer 3

### Layer 2: Semantic Match on _index.md
1. You already have _index.md in context
2. Use YOUR semantic understanding to rank entries by relevance
3. "查询窗口大小" might match "getWindowRect — Get window dimensions"
4. Think about what the user MEANS, not literal keywords
5. Pick the best candidate → go to Layer 3

### Layer 3: Verify and Decide
1. Read the cached doc for your candidate
2. Judge: does this ACTUALLY answer the user's query?
3. If YES:
   - Return this doc
   - Update aliases: prepend user's query to this entry's aliases in _index.md
   - If aliases > 10, remove the last one (LRU)
   - DONE
4. If NO:
   - Go back, try next candidate (up to 3 attempts)
   - If all candidates fail → go to Layer 4

### Layer 4: Search Source Code (last resort)
1. Glob: find files in `~/.repo-knowledge/_repos/{project_name}/`
2. Grep: search with multiple synonym variations (at least 3)
3. Read: get full context of best matches
4. Generate documentation (same format as rk-create)
5. Save to cache: write .md file + append to _index.md with user's query as first alias
6. Return the new doc

## Alias Update Rules
- Each _index.md entry has max 10 aliases
- New alias prepends to front (most recent first)
- If full (10), remove last one
- User confirms "yes this is right" → alias stays at front
- User says "no, wrong" → remove that alias

## _index.md Entry Format
```markdown
- [get-window-rect](docs/window/get-window-rect.md) — Get window dimensions（获取窗口尺寸）
  aliases: [窗口大小, screen size, 窗口多大, window dimensions]
```

## Output Format
Return the cached doc content to the user.
If multiple results are relevant, show top 3.
```
