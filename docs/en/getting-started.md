# Getting Started

## Prerequisites

- Claude Code CLI installed
- Plugin support available

## Installation

```bash
# Clone the repo
git clone https://github.com/code60-AI/repo-knowledge.git
cd repo-knowledge

# Launch Claude Code with plugin
claude --plugin-dir .
```

The SessionStart hook auto-creates the data directory on first launch:

```bash
mkdir -p ~/.repo-knowledge/_repos
touch -a ~/.repo-knowledge/_registry.md
```

---

## Step 1: Index Your First Repo

```
/repo-knowledge:rk-create https://github.com/org/my-project.git
```

Expected output:
```
Cloning into 'my-project'...
[Indexer Agent] Scanning 42 files...
[Indexer Agent] Generated 128 docs
Knowledge base created: my-project (128 docs)
```

SSH format also works:
```
/repo-knowledge:rk-create git@github.com:org/my-project.git
```

> **Note:** SSH format requires an SSH key configured on this machine and added to your GitHub account.

---

## Step 2: Your First Search

```
/repo-knowledge:rk-search how does authentication work
```

Or in Chinese:
```
/repo-knowledge:rk-search 认证逻辑在哪里
```

Expected output:
```
Searching my-project...
[Layer 2 semantic match] → authenticate-user.md

## authenticateUser
**File:** src/auth/authenticate.ts
...
```

---

## Step 3: Update After New Commits

When the repo has new commits:

```
/repo-knowledge:rk-update my-project
```

Expected output:
```
Pulling latest changes...
Changed files: 3
  Modified: src/auth/authenticate.ts (2 docs regenerated)
  Added: src/utils/hash.ts (1 new doc)
  Deleted: src/auth/legacy.ts (2 docs removed)
Updated: my-project (129 docs, last commit: a1b2c3d)
```

---

## Step 4: Manage Knowledge Bases

List all:
```
/repo-knowledge:rk-list
```

Expected:
```
| Project    | Git URL                              | Last Update | Docs |
|------------|--------------------------------------|-------------|------|
| my-project | https://github.com/org/my-project... | 2026-04-14  | 129  |
```

Delete:
```
/repo-knowledge:rk-delete my-project
```

System asks for confirmation before deleting.

---

## Tips

- Search supports any language, including mixed CN+EN
- After first search, similar queries find it faster
- Large repos (1000+ files) may take a few minutes to index
