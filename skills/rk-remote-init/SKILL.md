---
name: rk-remote-init
description: "Initialize remote sync for the knowledge base. Sets up ~/.repo-knowledge/ as a git repository, configures a user-provided remote URL, links to the main branch, and either pushes local data (first machine) or pulls existing data (subsequent machines). Use when user says 'set up remote sync', 'connect remote repo', 'init remote', 'rk-remote-init', or provides a git URL for syncing the knowledge base."
---

# RK Remote Init — Initialize Remote Sync

Link the local knowledge base to a user-provided remote git repository.

> **Security warning**: The remote repository will contain documentation that includes code snippets from your indexed projects. Always use a **private** repository.

## Input
- Git remote URL (SSH or HTTPS, e.g. `git@github.com:user/my-knowledge.git`)

## Step 1: Resolve home directory (cross-platform)

```bash
python3 -c "import os; print(os.path.expanduser('~'))"
```

Store result as `{HOME}`. Use `{HOME}/.repo-knowledge` as the base path for all operations.

## Step 2: Ensure base directory exists

Use the Write tool to create `{HOME}/.repo-knowledge/_registry.md` if it does not already exist (this also creates the directory). Content if creating new:
```markdown
# RepoKnowledge Registry

| Project | Git URL | Last Update | Docs |
|---------|---------|-------------|------|
```

## Step 3: Check if already a git repo

```bash
git -C {HOME}/.repo-knowledge rev-parse --git-dir 2>/dev/null && echo "yes" || echo "no"
```

If not a git repo, initialize:
```bash
git -C {HOME}/.repo-knowledge init
git -C {HOME}/.repo-knowledge checkout -b main 2>/dev/null || true
```

## Step 4: Create .gitignore

Use Write tool to create `{HOME}/.repo-knowledge/.gitignore`:
```
_repos/
*.tmp
```

## Step 5: Configure remote

```bash
git -C {HOME}/.repo-knowledge remote add origin <git-url> 2>/dev/null || \
git -C {HOME}/.repo-knowledge remote set-url origin <git-url>
```

## Step 6: Detect whether remote already has data

```bash
git -C {HOME}/.repo-knowledge fetch origin main 2>/dev/null && echo "has_data" || echo "empty"
```

### If remote already has data (subsequent machine setup):

Run the full merge flow from `rk-pull` (Steps 3–7 of rk-pull) to bring remote knowledge into local before pushing.

Then:
```bash
git -C {HOME}/.repo-knowledge push -u origin main
```

### If remote is empty (first machine):

```bash
git -C {HOME}/.repo-knowledge add -A
git -C {HOME}/.repo-knowledge commit -m "init: initial knowledge base" --allow-empty
git -C {HOME}/.repo-knowledge push -u origin main
```

## Step 7: Save remote config

Use Write tool to write `{HOME}/.repo-knowledge/_remote.md`:
```
remote: <git-url>
initialized: {YYYY-MM-DD}
```

## Step 8: Confirm

Tell the user:
- Remote URL configured
- Whether this was a first-time init (pushed local) or a new machine (pulled remote)
- Next step: `rk-update` will now auto-sync after each run
