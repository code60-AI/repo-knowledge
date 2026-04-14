# RepoKnowledge

Progressive codebase knowledge base plugin for Claude Code.

Clone git repos, auto-generate documentation for every function/class, and build a searchable knowledge cache that grows smarter with every query.

## Features

- **rk-create** — Clone a git repo and generate full documentation index
- **rk-update** — Incrementally update docs based on git commit history
- **rk-search** — Semantic search with progressive alias learning
- **rk-list** — List all indexed codebases
- **rk-delete** — Remove a codebase knowledge base

## Usage

```bash
# Index a new repo
/repo-knowledge:rk-create git@github.com:org/project.git

# Search the knowledge base
/repo-knowledge:rk-search 窗口大小
/repo-knowledge:rk-search authentication middleware

# Update after new commits
/repo-knowledge:rk-update my-project

# List all knowledge bases
/repo-knowledge:rk-list

# Delete a knowledge base
/repo-knowledge:rk-delete my-project
```

## How It Works

1. **Create**: Clones repo → scans code → generates docs (function/class level) → builds index
2. **Search**: Reads index → Agent semantic matching → verifies relevance → returns docs
3. **Learn**: Every successful search adds an alias to the index, making future searches faster
4. **Update**: Checks git diff → only regenerates changed docs → keeps cache fresh

## Data Directory

```
~/.repo-knowledge/
├── _registry.md              # All registered projects
├── _repos/                   # Cloned git repositories
│   └── my-project/
├── my-project/               # Knowledge cache
│   ├── _meta.md              # Project metadata
│   ├── _index.md             # Searchable index with aliases
│   └── docs/                 # Cached documentation
│       └── src/
│           └── auth/
│               └── validate-token.md
```

## Local Development

```bash
claude --plugin-dir ./repo-knowledge
```
