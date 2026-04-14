---
name: rk-update
description: "Incrementally update a codebase knowledge base by checking git commit history. Finds changed files since last update, regenerates only affected documentation, and updates the index. Use when user says 'update knowledge base', 'refresh codebase', 'sync repo knowledge', or 'rk-update'."
---

# RK Update — Incremental Knowledge Base Update

Update an existing knowledge base based on git commit history.

## Input
- Project name (must already exist in registry)

## Process

### Step 1: Read Project Metadata
```
Read ~/.repo-knowledge/<project-name>/_meta.md
Extract: last_commit hash
```

### Step 2: Pull Latest Code
```bash
cd ~/.repo-knowledge/_repos/<project-name>/
git pull
```

### Step 3: Find Changed Files
```bash
git diff --name-only <last_commit>..HEAD
```

This gives you the list of files that changed.

### Step 4: Process Each Changed File
Launch an Agent for incremental update:

```
You are a Knowledge Base Update Agent.

Codebase path: "~/.repo-knowledge/_repos/{project-name}/"
Cache path: "~/.repo-knowledge/{project-name}/"
Changed files:
{list of changed files from git diff}

## For Each Changed File:

### If file was MODIFIED:
1. Read the new version of the file
2. Find all functions/classes in it
3. Check if docs already exist for them in the cache
4. Regenerate the docs with updated code
5. Update the cached .md files

### If file was ADDED:
1. Read the file
2. Generate docs for each function/class (same format as rk-create)
3. Save to cache
4. Append to _index.md

### If file was DELETED:
1. Find cached docs that reference this file (check `source:` in frontmatter)
2. Delete those cached .md files
3. Remove entries from _index.md

## After processing all files:
1. Update _meta.md with new last_commit and last_update date
2. Update total_docs count
3. Update _registry.md
```

### Step 5: Report Summary
Output what was updated:
- Files modified: X
- New docs created: Y
- Docs removed: Z
- Total docs now: N
