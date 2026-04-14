# Epic & Features

## Epic

Enable AI coding assistants to deeply understand any codebase without reading every file -- by building a structured, queryable knowledge graph that captures code semantics, relationships, and design rationale at the function level.

---

## Feature 1: Knowledge Base Creation

Index any Git repository by cloning it locally, scanning all code files, and generating per-function/class documentation into a searchable cache.

### User Stories

**US-1.1**
> As a developer, I want to index a new Git repository with a single command, so that I can immediately start querying its code without reading every file.

**US-1.2**
> As a developer, I want each function and class to have its own documentation entry, so that I can get precise, focused answers rather than file-level summaries.

**US-1.3**
> As a developer, I want the knowledge base to be registered in a central registry, so that I can manage multiple codebases and know what's available.

---

## Feature 2: Semantic Search

Query the knowledge base using natural language in any language, with an Agent that semantically matches queries to documentation even when exact keywords don't match.

### User Stories

**US-2.1**
> As a developer, I want to search the codebase using natural language (including Chinese), so that I can find what I need without knowing exact function names.

**US-2.2**
> As a developer, I want the search to learn from my queries over time, so that repeated searches become faster and more accurate.

**US-2.3**
> As a developer, I want the system to fall back to searching source code when the cache doesn't have an answer, so that I never get a "not found" result for code that exists.

---

## Feature 3: Incremental Updates

Keep the knowledge base in sync with the codebase by detecting changed files via git history and regenerating only the affected documentation.

### User Stories

**US-3.1**
> As a developer, I want to update the knowledge base after new commits without re-indexing the entire repo, so that updates are fast regardless of repo size.

**US-3.2**
> As a developer, I want deleted files to be automatically removed from the knowledge base, so that I don't get results for code that no longer exists.

---

## Feature 4: Knowledge Base Management

List all indexed codebases and delete ones that are no longer needed.

### User Stories

**US-4.1**
> As a developer, I want to see all my indexed codebases in one place, so that I know what's available and when it was last updated.

**US-4.2**
> As a developer, I want to delete a knowledge base and all its cached data with one command, so that I can free up disk space when a project is no longer relevant.
