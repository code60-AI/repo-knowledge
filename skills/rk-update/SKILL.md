---
name: rk-update
description: "Incrementally update a codebase knowledge base by checking git commit history. Finds changed files since last update, regenerates only affected documentation, and updates the index. Auto-pulls from remote before updating and auto-pushes after if remote is configured. Use when user says 'update knowledge base', 'refresh codebase', 'sync repo knowledge', or 'rk-update'."
---

# RK Update — Incremental Knowledge Base Update

## Input
- Project name (must already exist in registry)

## Step 1: Resolve home directory (cross-platform)

```bash
python3 -c "import os; print(os.path.expanduser('~'))"
```

Store as `{HOME}`. Base path: `{HOME}/.repo-knowledge`.

## Step 2: Auto-pull from remote (if configured)

Check if `{HOME}/.repo-knowledge/_remote.md` exists.

If it exists: run the full merge flow from `rk-pull` (Steps 3–7) before doing anything else.
This ensures local knowledge is up to date with all other machines before we add new changes.

## Step 3: Ensure source repo exists

Check if `{HOME}/.repo-knowledge/_repos/<project-name>/` exists.

If it does NOT exist:
1. Read `{HOME}/.repo-knowledge/<project-name>/_meta.md` to get `git_url`
2. Clone the source repo:
   ```bash
   git clone <git_url> {HOME}/.repo-knowledge/_repos/<project-name>
   ```
3. Continue — the repo is now available locally.

## Step 4: Read project metadata

Read `{HOME}/.repo-knowledge/<project-name>/_meta.md`.
Extract: `last_commit` hash.

## Step 5: Pull latest source code

```bash
git -C {HOME}/.repo-knowledge/_repos/<project-name> pull
```

## Step 6: Find changed files since last sync

```bash
git -C {HOME}/.repo-knowledge/_repos/<project-name> diff --name-only <last_commit>..HEAD
```

If no files changed: tell user "Already up to date." Skip to Step 8 (auto-push still runs if remote is configured).

## Step 7: Process each changed file

Launch an Agent with this prompt:

```
You are a Knowledge Base Update Agent.

Codebase path: "{HOME}/.repo-knowledge/_repos/{project-name}/"
Cache path: "{HOME}/.repo-knowledge/{project-name}/"
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
3. Update _registry.md row for this project
```

## Step 8: Auto-push to remote (if configured)

Check if `{HOME}/.repo-knowledge/_remote.md` exists.

If it exists: run the full push flow from `rk-push` (Steps 3–10) to sync the updated knowledge to remote.

## Step 9: Report summary

Output:
- Files modified: X
- New docs created: Y
- Docs removed: Z
- Total docs now: N
- Remote sync: pushed / skipped (no remote configured)
