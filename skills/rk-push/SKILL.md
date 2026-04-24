---
name: rk-push
description: "Push local knowledge base to the remote repository. Always fetches remote first, Agent compares diffs and merges (alias LRU merge for _index.md, union dedup for _registry.md, newer-cached-date wins for docs), then commits and pushes. Retries once on push rejection. Use when user says 'push knowledge', 'sync to remote', 'rk-push', or after making local updates."
---

# RK Push — Sync Local Knowledge Base to Remote

## Step 1: Resolve home directory (cross-platform)

```bash
python3 -c "import os; print(os.path.expanduser('~'))"
```

Store as `{HOME}`. Base path: `{HOME}/.repo-knowledge`.

## Step 2: Check remote is configured

Read `{HOME}/.repo-knowledge/_remote.md`.
If not found: tell user to run `/repo-knowledge:rk-remote-init <git-url>` first. Stop.

## Step 3: Fetch remote

```bash
git -C {HOME}/.repo-knowledge fetch origin main
```

## Step 4: Check if remote has new commits

```bash
git -C {HOME}/.repo-knowledge rev-list HEAD..origin/main --count
```

If count > 0, remote is ahead — proceed to Step 5 (merge).
If count = 0, no remote changes — skip to Step 8.

## Step 5: Identify changed files

```bash
git -C {HOME}/.repo-knowledge diff HEAD origin/main --name-only
```

## Step 6: Agent merge for each changed file

Process each file from the diff. Use `git -C {HOME}/.repo-knowledge show origin/main:<relative-path>` to read the remote version of any file.

### `_registry.md`
1. Read local `{HOME}/.repo-knowledge/_registry.md`
2. Read remote: `git -C {HOME}/.repo-knowledge show origin/main:_registry.md`
3. Merge: union of all data rows, deduplicate by project name (first column), keep the row with the most recent `Last Update` date for duplicates
4. Write merged result with Write tool

### `<project>/_index.md`
1. Read local version
2. Read remote: `git -C {HOME}/.repo-knowledge show origin/main:<project>/_index.md`
3. Parse entries — each entry is identified by its doc path (the `(path/to/file.md)` part)
4. Merge:
   - **Same doc path in both**: union of alias lists, deduplicate, keep first 10 (local aliases go first as they are more recent)
   - **Only in remote**: append to local
   - **Only in local**: keep as-is
5. Write merged `_index.md`

### `<project>/docs/*.md` (any doc file)
1. Read local file frontmatter (`cached:` field)
2. Read remote version frontmatter: `git -C {HOME}/.repo-knowledge show origin/main:<path>`
3. Keep whichever has the newer `cached:` date
4. If remote is newer: overwrite local with Write tool

### `<project>/_meta.md`
1. Read both versions
2. Merge fields:
   - `last_update`: keep the more recent date
   - `total_docs`: keep the higher number
   - `git_url`: keep either (should be identical)
   - `last_commit`: keep from whichever side has the newer `last_update`
3. Write merged result

## Step 7: Stage merged files

```bash
git -C {HOME}/.repo-knowledge add -A
```

## Step 8: Commit

```bash
git -C {HOME}/.repo-knowledge diff --cached --quiet || \
  git -C {HOME}/.repo-knowledge commit -m "sync: $(python3 -c \"import datetime; print(datetime.datetime.now().strftime('%Y-%m-%d %H:%M'))\")"
```

If nothing staged, skip commit.

## Step 9: Push (with retry)

```bash
git -C {HOME}/.repo-knowledge push origin main
```

If push is rejected (remote is ahead — race condition):
- Re-run from Step 3 (fetch → merge → commit → push)
- Maximum 2 retries
- If still failing after 2 retries: tell user "Push failed after retries — please run `/repo-knowledge:rk-push` again."

## Step 10: Report

Tell user:
- How many files were synced
- Per project: new entries added, alias lists merged
