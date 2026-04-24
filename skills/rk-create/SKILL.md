---
name: rk-create
description: "Create a new codebase knowledge base from a git repository URL. Clones the repo, scans all code files, generates documentation for each function/class, and builds a searchable index. Use when user says 'create knowledge base', 'index a repo', 'add a codebase', or provides a git URL to index."
---

# RK Create — Initialize Codebase Knowledge Base

Create a new knowledge base from a git repository.

## Input
- Git repository URL (HTTPS or SSH)

## Process

### Step 1: Clone the Repository
```bash
cd ~/.repo-knowledge/_repos/
git clone <git-url> <project-name>
```
Derive `{project-name}` from the repo URL (e.g., `git@github.com:org/my-project.git` -> `my-project`).

### Step 2: Create Project Cache Structure
```bash
mkdir -p ~/.repo-knowledge/<project-name>/docs/
```

### Step 3: Scan and Index
Launch an **indexer Agent** to scan the codebase:

Use the Agent tool with this prompt:
```
You are a Codebase Indexer Agent.

Codebase path: "~/.repo-knowledge/_repos/{project-name}/"
Cache path: "~/.repo-knowledge/{project-name}/"

## Your Task
Scan the entire codebase and generate documentation for every function and class.

## Rules for Document Granularity
- One document per function
- One document per class (if class < 200 lines)
- If class > 200 lines, split by method — one document per method
- If function > 100 lines, split by logical sections

## For Each Component, Generate:

```markdown
---
cached: {YYYY-MM-DD}
source: {relative/path/to/file.ext}
---

## [Function/Class Name]
**File:** [relative/path/to/file.ext]

### What It Does
[1-2 sentence explanation in English and Chinese]

### Code
```[language]
[actual code]
```

### Parameters
- `param_name` (type): Description

### Returns
- (type): Description

### Usage Example
```[language]
[practical working example]
```

### Notes
- Caveats, edge cases, related functions
```

## Save Each Document
1. Organize by directory structure matching source:
   `~/.repo-knowledge/{project-name}/docs/{source-dir}/{kebab-case-name}.md`
2. After ALL docs are generated, create `_index.md`:
   ```markdown
   # {project-name} Knowledge Index

   - [component-name](docs/path/component-name.md) — English description（中文描述）
   ```

## Scan Strategy
1. Use Glob to find all code files (*.ts, *.js, *.py, *.go, *.java, *.rs, etc.)
2. Skip: node_modules, .git, dist, build, vendor, __pycache__
3. Read each file
4. Extract functions and classes
5. Generate doc for each
6. Save to cache
7. Build _index.md
```

### Step 4: Create Project Metadata
Save `_meta.md`:
```markdown
git_url: <git-url>
created: {YYYY-MM-DD}
last_update: {YYYY-MM-DD}
last_commit: {commit-hash}
total_docs: {count}
```

### Step 5: Register the Project
Append to `~/.repo-knowledge/_registry.md`:
```markdown
| {project-name} | {git-url} | {YYYY-MM-DD} | {doc-count} |
```

If `_registry.md` is empty, create it with header:
```markdown
# RepoKnowledge Registry

| Project | Git URL | Last Update | Docs |
|---------|---------|-------------|------|
| {project-name} | {git-url} | {YYYY-MM-DD} | {count} |
```
