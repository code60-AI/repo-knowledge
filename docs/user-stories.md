# User Stories & Use Cases

Each User Story from [epic.md](epic.md) is expanded here into concrete Use Cases with preconditions, main flow, alternate flows, and postconditions.

---

## US-1.1 — Index a new repository / 索引新仓库

### UC-1.1.1: Index a public GitHub repo

**前置条件 / Preconditions**
- Claude Code is running with repo-knowledge plugin loaded
- User has network access to clone the repository

**主流程 / Main Flow**
1. User types `/repo-knowledge:rk-create https://github.com/org/project.git`
2. System clones the repo to `~/.repo-knowledge/_repos/project/`
3. System creates cache directory `~/.repo-knowledge/project/docs/`
4. Indexer Agent scans all code files (*.ts, *.py, *.go, etc.)
5. For each function/class found, Agent generates a documentation entry and saves to cache
6. System builds `_index.md` with one-line summaries for all entries
7. System saves `_meta.md` with git URL, creation date, last commit hash, and doc count
8. System appends project row to `~/.repo-knowledge/_registry.md`
9. System reports: "Indexed `project`: N docs generated"

**异常流程 / Alternate Flows**
- *Network failure during clone*: System reports clone error; no partial state is written
- *Empty repository*: System reports "No code files found" and skips index creation
- *Project already exists in registry*: System warns user and suggests using `rk-update` instead

**后置条件 / Postconditions**
- `~/.repo-knowledge/project/_index.md` exists with all doc entries
- Project row exists in `_registry.md`
- Knowledge base is ready for `rk-search`

---

## US-2.1 — Search with natural language / 自然语言搜索

### UC-2.1.1: Search finds answer in cache

**前置条件 / Preconditions**
- At least one knowledge base exists in `_registry.md`
- `_index.md` has been built for the target project

**主流程 / Main Flow**
1. User types `/repo-knowledge:rk-search 如何验证用户身份`
2. System identifies target project (auto-selects if only one; prompts if multiple)
3. Searcher Agent reads `_index.md`
4. Agent scans aliases — no direct alias match found
5. Agent uses semantic understanding to rank index entries; selects `validate-token` as best match
6. Agent reads cached doc for `validate-token`
7. Agent judges: doc answers the query ✓
8. Agent prepends `"如何验证用户身份"` to `validate-token`'s aliases in `_index.md`
9. System returns cached doc content to user

**异常流程 / Alternate Flows**
- *Alias match at step 4*: Skip semantic ranking, go directly to verify cached doc
- *First candidate fails verification*: Try next candidate (up to 3 attempts)
- *All 3 candidates fail*: Proceed to UC-2.1.2 (source code fallback)

**后置条件 / Postconditions**
- User receives relevant documentation
- New alias added to `_index.md` for faster future lookups

### UC-2.1.2: Search falls back to source code

**前置条件 / Preconditions**
- Knowledge base exists but cache doesn't contain a relevant entry

**主流程 / Main Flow**
1. After 3 failed cache candidates, Agent switches to source code search
2. Agent uses Glob to find all code files in `~/.repo-knowledge/_repos/project/`
3. Agent uses Grep with 3+ synonym variations of the query
4. Agent reads matching file sections for context
5. Agent generates new documentation entry in standard format
6. Agent saves new `.md` file to cache under correct directory
7. Agent appends new entry to `_index.md` with query as first alias
8. System returns newly generated doc to user

**后置条件 / Postconditions**
- New doc cached; future searches for same concept hit cache at Layer 1 (alias match)

---

## US-3.1 — Incremental update / 增量更新

### UC-3.1.1: Update after new commits

**前置条件 / Preconditions**
- Project exists in registry with a recorded `last_commit` hash
- Remote repository has new commits since last update

**主流程 / Main Flow**
1. User types `/repo-knowledge:rk-update project`
2. System reads `last_commit` from `~/.repo-knowledge/project/_meta.md`
3. System runs `git pull` in `~/.repo-knowledge/_repos/project/`
4. System runs `git diff --name-only <last_commit>..HEAD` to get changed file list
5. Update Agent processes each changed file:
   - **Modified**: regenerates docs for all functions/classes in that file
   - **Added**: generates new docs, appends entries to `_index.md`
   - **Deleted**: removes cached `.md` files, removes entries from `_index.md`
6. System updates `_meta.md` with new `last_commit`, `last_update`, `total_docs`
7. System updates `_registry.md` row with new date and doc count
8. System reports summary: modified/added/removed counts

**异常流程 / Alternate Flows**
- *No new commits*: System reports "Already up to date" and exits
- *Project not in registry*: System suggests using `rk-create` instead

**后置条件 / Postconditions**
- Cache reflects current state of the codebase
- `_meta.md` has updated `last_commit` hash

---

## US-4.2 — Delete a knowledge base / 删除知识库

### UC-4.2.1: Delete with confirmation

**前置条件 / Preconditions**
- Project exists in registry

**主流程 / Main Flow**
1. User types `/repo-knowledge:rk-delete project`
2. System asks: "Are you sure you want to delete the knowledge base for `project`? This will remove all cached documentation."
3. User confirms
4. System deletes `~/.repo-knowledge/project/` (entire cache directory)
5. System deletes `~/.repo-knowledge/_repos/project/` (cloned repo)
6. System removes project row from `_registry.md`
7. System confirms: "Knowledge base for `project` has been deleted."

**异常流程 / Alternate Flows**
- *User declines confirmation*: System cancels, nothing is deleted
- *Project not in registry*: System reports "No knowledge base found for `project`"

**后置条件 / Postconditions**
- No files remain for the project under `~/.repo-knowledge/`
- Project row removed from `_registry.md`

---

## US-1.2 — Per-function documentation / 函数级文档

### UC-1.2.1: Each function gets its own doc entry

**前置条件 / Preconditions**
- A repository has been indexed with `rk-create`

**主流程 / Main Flow**
1. Indexer Agent reads a source file (e.g., `src/auth/validate-token.ts`)
2. Agent identifies each function and class in the file
3. For each function, Agent generates a dedicated Markdown doc with: name, file path, what it does (CN+EN), code, parameters, return value, usage example, notes
4. Doc is saved to `~/.repo-knowledge/<project>/docs/src/auth/validate-token.md`
5. A one-line entry is appended to `_index.md`

**异常流程 / Alternate Flows**
- *Function > 100 lines*: Split into logical sections; each section gets its own doc
- *Class > 200 lines*: Split by method; each method gets its own doc

**后置条件 / Postconditions**
- One `.md` file exists per function/class (or per method for large classes)
- `_index.md` has a one-line entry for each doc

---

## US-1.3 — Central registry / 统一注册表

### UC-1.3.1: Project registered after indexing

**前置条件 / Preconditions**
- `rk-create` has completed successfully for a project

**主流程 / Main Flow**
1. System reads current `~/.repo-knowledge/_registry.md`
2. If registry doesn't exist, system creates it with header row
3. System appends a new row: `| project-name | git-url | YYYY-MM-DD | doc-count |`
4. Registry is now queryable by `rk-list`, `rk-search`, `rk-update`, and `rk-delete`

**异常流程 / Alternate Flows**
- *Project already in registry*: System skips append; existing row is preserved

**后置条件 / Postconditions**
- `_registry.md` contains a row for the project
- All other skills can discover the project via `_registry.md`

---

## US-2.2 — Progressive alias learning / 渐进式别名学习

### UC-2.2.1: Alias added after successful search

**前置条件 / Preconditions**
- A search query returned a relevant result

**主流程 / Main Flow**
1. Searcher Agent returns a cached doc that answers the user's query
2. Agent prepends the query string to the target entry's `aliases` list in `_index.md`
3. If the aliases list has 10 entries, the oldest (last) entry is removed (LRU eviction)
4. On the next search with the same or similar query, Layer 1 (alias match) fires immediately

**异常流程 / Alternate Flows**
- *User indicates result was wrong*: Agent removes the newly added alias from the entry

**后置条件 / Postconditions**
- `_index.md` entry has the query prepended to its aliases list
- Subsequent identical queries resolve at Layer 1 without semantic ranking

---

## US-2.3 — Source code fallback / 源码回退搜索

### UC-2.3.1: Cache miss triggers source code search

**前置条件 / Preconditions**
- All 3 cache candidates failed to answer the query (Layer 3 exhausted)

**主流程 / Main Flow**
1. Searcher Agent switches to Layer 4 (source code search)
2. Agent uses Glob to list all code files in `~/.repo-knowledge/_repos/<project>/`
3. Agent runs Grep with at least 3 synonym variations of the original query
4. Agent reads the most relevant matching code sections
5. Agent generates a new documentation entry in standard format (cached: date, source: path, code, params, returns, example)
6. Agent saves the new `.md` file to the correct cache directory
7. Agent appends a new entry to `_index.md` with the query as the first alias
8. System returns the newly generated doc to the user

**后置条件 / Postconditions**
- A new doc exists in cache for the previously uncached concept
- Future searches for the same concept resolve at Layer 1 (alias match)

---

## US-3.2 — Auto-remove deleted files / 自动清理已删除文件

### UC-3.2.1: Deleted source file removed from cache

**前置条件 / Preconditions**
- A file was deleted in the repository since the last `rk-update`

**主流程 / Main Flow**
1. `rk-update` runs `git diff --name-only <last_commit>..HEAD`
2. The deleted file appears in the diff output
3. Update Agent finds all cached `.md` files whose `source:` frontmatter matches the deleted file path
4. Agent deletes those cached `.md` files from disk
5. Agent removes the corresponding entries from `_index.md`
6. `_meta.md` `total_docs` count is decremented accordingly

**후置条件 / Postconditions**
- No cached docs reference the deleted source file
- `_index.md` contains no entries pointing to deleted docs
- Search will never return results for code that no longer exists

---

## US-4.1 — List all knowledge bases / 列出所有知识库

### UC-4.1.1: View registered codebases

**前置条件 / Preconditions**
- Claude Code is running with repo-knowledge plugin loaded

**主流程 / Main Flow**
1. User types `/repo-knowledge:rk-list`
2. System reads `~/.repo-knowledge/_registry.md`
3. System formats and displays the registry table showing: project name, git URL, last update date, doc count
4. User sees at a glance which codebases are indexed and how fresh they are

**异常流程 / Alternate Flows**
- *Registry is empty or doesn't exist*: System displays "No knowledge bases found. Use `/repo-knowledge:rk-create <git-url>` to create one."

**后置条件 / Postconditions**
- User has a complete view of all indexed knowledge bases
- No state is modified
