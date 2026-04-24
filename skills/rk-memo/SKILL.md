---
name: rk-memo
description: "Save a piece of personal knowledge to the user's personal knowledge base (_personal project). No git repo needed — stores anything the user knows by heart: commands, tips, best practices, personal workflows. Searchable via rk-search, synced via rk-push/pull. Use when user says 'remember this', 'save this note', 'rk-memo', or wants to store a frequently-used snippet or trick."
---

# RK Memo — Save Personal Knowledge

Store a piece of personal knowledge that isn't tied to any codebase.

## Input
Free-form: `<title>: <content>` or just natural text describing what to save.

## Step 1: Resolve home directory (cross-platform)

```bash
python3 -c "import os; print(os.path.expanduser('~'))"
```

Store as `{HOME}`. Base path: `{HOME}/.repo-knowledge`.

## Step 2: Initialize _personal project if it doesn't exist

Check if `{HOME}/.repo-knowledge/_personal/` exists.

If not, create it:

Use Write tool to create `{HOME}/.repo-knowledge/_personal/_meta.md`:
```
type: personal
git_url: none
created: {YYYY-MM-DD}
last_update: {YYYY-MM-DD}
total_docs: 0
```

Use Write tool to create `{HOME}/.repo-knowledge/_personal/_index.md`:
```markdown
# _personal Knowledge Index

```

Add entry to `{HOME}/.repo-knowledge/_registry.md`:
```
| _personal | (personal) | {YYYY-MM-DD} | 0 |
```

Also create the docs directory by writing a placeholder if needed.

## Step 3: Derive doc filename

Convert the title to kebab-case:
- Lowercase
- Replace spaces and special characters with `-`
- Remove consecutive dashes

Example: `kubectl 查看 pod 状态` → `kubectl-pod-status`

Check if `{HOME}/.repo-knowledge/_personal/docs/<kebab-name>.md` already exists.
If it exists: this is an update — overwrite the existing doc.
If not: this is a new entry.

## Step 4: Write the doc

Use Write tool to save `{HOME}/.repo-knowledge/_personal/docs/<kebab-name>.md`:

```markdown
---
cached: {YYYY-MM-DD}
source: personal
---

## {title}

{content}
```

Keep content exactly as the user provided. If the user gave a code snippet, preserve it as a fenced code block with appropriate language tag.

## Step 5: Update _index.md

For **new entries**: append to `{HOME}/.repo-knowledge/_personal/_index.md`:
```markdown
- [{title}](docs/<kebab-name>.md) — {one-line summary}
  aliases: [{title}, {key terms extracted from content}]
```

For **existing entries** (update): find the line with this doc path and update the description. Keep existing aliases, prepend the new title as alias if it changed.

## Step 6: Update _meta.md

Read current `total_docs`, increment by 1 (only for new entries).
Update `last_update` to today.
Write updated `_meta.md`.

## Step 7: Update _registry.md

Find the `_personal` row and update `Last Update` and `Docs` count.

## Step 8: Auto-push if remote configured

Check if `{HOME}/.repo-knowledge/_remote.md` exists.
If yes: run rk-push flow (Steps 3–10 of rk-push).

## Step 9: Confirm

Tell the user:
- Title saved
- How to retrieve it: `/repo-knowledge:rk-search <title or related terms>`
