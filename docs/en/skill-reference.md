# Skill Reference

All Skills are triggered via Claude Code's `/` command syntax.

---

## rk-create

**Trigger:** `/repo-knowledge:rk-create <git-url>`

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `git-url` | string (required) | HTTPS or SSH Git URL |

**Behavior:**

1. Infers project name from the URL (last segment, stripping `.git`)
2. Clones the repository to `~/.repo-knowledge/_repos/<project>/`
3. Dispatches Indexer Agent to scan all code files and generate documentation
4. Saves `_meta.md`, `_index.md`, and registers the project in `_registry.md`

**Example:**

```
/repo-knowledge:rk-create https://github.com/anthropics/anthropic-sdk-python.git
/repo-knowledge:rk-create git@github.com:org/project.git
```

**Notes:**
- Warns to use `rk-update` if project name already exists
- Private repos require SSH key or token configured

---

## rk-search

**Trigger:** `/repo-knowledge:rk-search <query>`

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `query` | string (required) | Natural language query, any language |

**Behavior:**

1. Reads `_registry.md` to determine the search target (auto-selects for single project, prompts for multiple)
2. Dispatches Searcher Agent to execute a 4-layer search strategy
3. Returns matching document content
4. Writes the query term as an alias into `_index.md`

**Example:**

```
/repo-knowledge:rk-search how does authentication work
/repo-knowledge:rk-search window size
/repo-knowledge:rk-search JWT token validation
```

**Notes:**
- Supports cross-language queries (Chinese queries can match English documents)
- After each successful search, aliases in `_index.md` are updated (LRU, max 10)
- On cache miss, automatically falls back to source code search and saves new documentation

---

## rk-update

**Trigger:** `/repo-knowledge:rk-update <project-name>`

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `project-name` | string (required) | Registered project name |

**Behavior:**

1. Reads `_meta.md` to obtain `last_commit`
2. Runs `git pull` to fetch the latest code
3. Runs `git diff --name-only <last_commit>..HEAD` to get the list of changed files
4. Performs incremental processing on changed files: rebuild modified, add new, clean deleted
5. Updates `_meta.md` and `_registry.md`

**Example:**

```
/repo-knowledge:rk-update my-project
```

**Notes:**
- Suggests using `rk-create` if the project is not in the registry
- Reports "Already up to date" and exits when there are no new commits

---

## rk-list

**Trigger:** `/repo-knowledge:rk-list`

**Parameters:** None

**Behavior:**

Reads and displays `~/.repo-knowledge/_registry.md` in a formatted table.

**Example:**

```
/repo-knowledge:rk-list
```

Expected output:
```
| Project    | Git URL                    | Last Update | Docs |
|------------|----------------------------|-------------|------|
| my-project | https://github.com/org/... | 2026-04-14  | 129  |
```

**Notes:**
- Prompts to use `rk-create` to create the first knowledge base when the registry is empty

---

## rk-delete

**Trigger:** `/repo-knowledge:rk-delete <project-name>`

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `project-name` | string (required) | Project name to delete |

**Behavior:**

1. Asks the user to confirm deletion
2. Deletes `~/.repo-knowledge/<project>/` (cache directory)
3. Deletes `~/.repo-knowledge/_repos/<project>/` (cloned repository)
4. Removes the corresponding entry from `_registry.md`
5. Confirms deletion is complete

**Example:**

```
/repo-knowledge:rk-delete my-project
```

**Notes:**
- Operation is irreversible; the system requires confirmation before deleting
- Reports "No knowledge base found" if the project is not in the registry
