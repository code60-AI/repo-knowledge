---
name: rk-list
description: "List all codebase knowledge bases in the registry. Shows project name, git URL, last update time, and document count. Use when user says 'list repos', 'show knowledge bases', 'what codebases do I have', or 'rk-list'."
---

# RK List — List All Knowledge Bases

Show all registered codebase knowledge bases.

## Process

1. Read `~/.repo-knowledge/_registry.md`
2. For each project row, check `~/.repo-knowledge/<project>/_meta.md` for `type` field:
   - `type: personal` → show as `(personal)` in the Git URL column
   - No type / `type: repo` → show the git URL as normal
3. Format and display the table to the user
4. If registry is empty or doesn't exist, tell the user:
   "No knowledge bases found. Use `/repo-knowledge:rk-create <git-url>` to create one, or `/repo-knowledge:rk-memo` to save personal knowledge."
