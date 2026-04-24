---
name: rk-pull
description: "Pull knowledge base from remote and merge into local. Fetches remote state, Agent compares diffs and merges (alias LRU merge for _index.md, union dedup for _registry.md, newer-cached-date wins for docs). Does NOT push — local only. Use when user says 'pull knowledge', 'sync from remote', 'rk-pull', or on a new machine to get existing knowledge without re-indexing."
---

# RK Pull — Pull Remote Knowledge Base to Local

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

If count = 0: tell user "Already up to date." Stop.

## Step 5: Identify changed files

```bash
git -C {HOME}/.repo-knowledge diff HEAD origin/main --name-only
```

## Step 6: Agent merge for each changed file

Process each file from the diff. Use `git -C {HOME}/.repo-knowledge show origin/main:<relative-path>` to read the remote version.

### `_registry.md`
1. Read local `{HOME}/.repo-knowledge/_registry.md`
2. Read remote: `git -C {HOME}/.repo-knowledge show origin/main:_registry.md`
3. Union of all data rows, deduplicate by project name, keep the row with the most recent `Last Update` date for duplicates
4. Write merged result with Write tool

### `<project>/_index.md`
1. Read local version
2. Read remote: `git -C {HOME}/.repo-knowledge show origin/main:<project>/_index.md`
3. Parse entries by doc path
4. Merge:
   - **Same doc path in both**: union aliases, deduplicate, keep first 10 (remote aliases go first here — remote is the newer authoritative source in a pull)
   - **Only in remote**: append to local (new knowledge from other machines)
   - **Only in local**: keep as-is
5. Write merged `_index.md`

### `<project>/docs/*.md`
1. Read local file frontmatter (`cached:` field)
2. Read remote: `git -C {HOME}/.repo-knowledge show origin/main:<path>`
3. Keep whichever has the newer `cached:` date
4. If remote is newer: overwrite local with Write tool

### `<project>/_meta.md`
1. Read both versions
2. Merge: `last_update` = newer date, `total_docs` = higher number, `git_url` = either, `last_commit` = from side with newer `last_update`
3. Write merged result

### New project (exists on remote, not local)
If remote has an entire project directory that does not exist locally:
1. Bring over all files from that project: `_meta.md`, `_index.md`, all `docs/`
2. Note: `_repos/<project>` will NOT exist — this is expected. `rk-update` will auto-clone source when needed.
3. Add project to local `_registry.md`

## Step 7: Commit merged state

```bash
git -C {HOME}/.repo-knowledge add -A
git -C {HOME}/.repo-knowledge diff --cached --quiet || \
  git -C {HOME}/.repo-knowledge commit -m "pull: merged from remote $(python3 -c \"import datetime; print(datetime.datetime.now().strftime('%Y-%m-%d %H:%M'))\")"
```

> Do NOT push — this is a local-only operation.

## Step 8: Report

Tell user:
- New projects pulled (if any)
- Per project: new entries added, aliases merged
- Reminder: `_repos/<project>` for pulled projects will be auto-cloned next time `rk-update` runs
