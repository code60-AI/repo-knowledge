---
name: rk-search
description: "Search the codebase knowledge base with breadth-first scan and progressive alias learning. Scans ALL project indexes in parallel (including _personal), scores candidates by alias/semantic match, then deep-verifies the best match. Use when user asks to find code, understand how something works, or get usage examples."
---

# RK Search — Two-Phase Semantic Knowledge Base Search

Search the knowledge base using breadth-first scanning across all projects, then deep verification of the best candidate.

## Input
- Search query (natural language, any language)
- Optional: project name (if user explicitly specifies which project to search)

## Step 1: Check for User-Specified Project

If the user explicitly names a project (e.g., "search cef-bridge for Execute"):
- Skip Phase 1 (Breadth Scan) entirely
- Go directly to Step 3 with that single project
- This avoids unnecessary scanning when the user knows where to look

Otherwise, proceed to Phase 1.

## Step 2: Phase 1 — Breadth Scan (All Projects)

Shallow-scan every knowledge base in parallel to find the best candidates across all projects.

### 2.1: Discover Projects

1. Read `~/.repo-knowledge/_registry.md` to get registered projects
2. Check if `~/.repo-knowledge/_personal/_index.md` exists
3. Build the scan list:
   - If `_personal` exists AND is not already in the registry: prepend `_personal` to the project list (highest priority)
   - Add all registered projects in registry order
4. If scan list is empty → tell user: "No knowledge bases found. Use `/repo-knowledge:rk-create <git-url>` to create one, or `/repo-knowledge:rk-memo` to save personal knowledge."
5. If scan list has exactly 1 entry → skip scoring, go directly to Step 3 with that single project

### 2.2: Read All Indexes in Parallel

Read `~/.repo-knowledge/{project}/_index.md` for every project in the scan list simultaneously.

### 2.3: Score Candidates

For each entry in each project's index, compute a score:

| Match Type | Score | Condition |
|------------|-------|-----------|
| Alias exact match | 3 (HIGH) | An alias exactly equals the query (case-insensitive) |
| Alias partial match | 2 (MEDIUM) | An alias contains the query or query contains the alias |
| Semantic match | 1 (LOW-MEDIUM) | Entry description semantically relates to the query |

Scoring rules:
- A single entry gets the **highest** matching score only (no double-counting)
- Add a **project priority bonus**: `_personal` entries get +0.5; registry projects get +0.3 / position (1st = +0.3, 2nd = +0.15, 3rd = +0.1, etc.) to break ties
- Only entries with score > 0 are candidates

### 2.4: Rank and Select

1. Sort all candidates by score descending (higher = better)
2. Take the **top 3** candidates
3. If **no candidates** found → tell user: "No matches found for '{query}' across {N} knowledge bases. Try rephrasing your query or use `/repo-knowledge:rk-create` to index a new codebase."
4. Otherwise → proceed to Step 3 with the ranked candidate list

## Step 3: Phase 2 — Deep Verification (Searcher Agent)

Launch a **searcher Agent** to deep-verify candidates through Layer 3 and Layer 4.

If breadth scan produced candidates (from Step 2):
```
You are a Knowledge Base Search Agent.

Search query: "{user_query}"
Mode: breadth-scan verification
Ranked candidates (try in order, max 3):

{for each candidate, format:}
- Project: "{project_name}", Doc: "{doc_path}", Score: {score}, Reason: {one of: 'alias exact match', 'alias partial match', 'semantic match'}

Cache base path: "~/.repo-knowledge/"

## Verification Strategy

### Layer 3: Verify and Decide
1. Read the cached doc for the top-ranked candidate
2. Judge: does this ACTUALLY answer the user's query?
3. If YES:
   - Return this doc
   - Update aliases: prepend user's query to this entry's aliases in _index.md
   - If aliases > 10, remove the last one (LRU)
   - DONE
4. If NO:
   - Try the next candidate in the ranked list (up to 3 total attempts)
   - If all candidates fail → go to Layer 4

### Layer 4: Search Source Code (last resort)
1. Pick the project from the top-ranked candidate that has a `_repos` directory
   - If the top candidate's project has no `_repos` (e.g., `_personal`), try the next candidate's project
   - If no candidate has a source repo → inform user and suggest rephrasing the query
2. Glob: find files in `~/.repo-knowledge/_repos/{project_name}/`
3. Grep: search with multiple synonym variations (at least 3)
4. Read: get full context of best matches
5. Generate documentation (same format as rk-create)
6. Save to cache: write .md file + append to _index.md with user's query as first alias
7. Return the new doc

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

If user specified a single project (from Step 1):
```
You are a Knowledge Base Search Agent.

Search query: "{user_query}"
Project: "{project_name}"
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

## _personal Handling Notes
- `_personal` is auto-detected by checking `~/.repo-knowledge/_personal/_index.md`
- If `rk-memo` already added `_personal` to `_registry.md`, it will be found via registry read — no duplicate
- If `_personal` is NOT in registry, the filesystem check adds it with highest priority
- When absent from both registry and filesystem, it is simply not included in the scan (no error)
