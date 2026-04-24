---
name: rk-delete
description: "Delete a codebase knowledge base and all its cached documentation. Removes the project cache, cloned repo, and registry entry. Use when user says 'delete knowledge base', 'remove repo', 'rk-delete', or wants to clean up a project."
---

# RK Delete — Remove Knowledge Base

Delete a project's knowledge base completely.

## Input
- Project name

## Process

### Step 1: Confirm with User
Ask: "Are you sure you want to delete the knowledge base for `{project-name}`? This will remove all cached documentation."

### Step 2: Delete Cache
```bash
rm -rf ~/.repo-knowledge/<project-name>/
```

### Step 3: Delete Cloned Repo (skip for personal knowledge bases)

Check `~/.repo-knowledge/<project-name>/_meta.md` before deleting (read it first from Step 2's backup or skip if already deleted).

If `type: personal` or `git_url: none`: skip this step — there is no cloned repo.

Otherwise:
```bash
rm -rf ~/.repo-knowledge/_repos/<project-name>/
```

### Step 4: Update Registry
Remove the project's row from `~/.repo-knowledge/_registry.md`.

### Step 5: Confirm
Tell the user: "Knowledge base for `{project-name}` has been deleted."
