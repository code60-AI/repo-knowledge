# 快速上手

## 前置条件

- Claude Code CLI 已安装
- 已安装插件支持

## 安装

```bash
# 克隆仓库
git clone https://github.com/code60-AI/repo-knowledge.git
cd repo-knowledge

# 以插件模式启动 Claude Code
claude --plugin-dir .
```

首次启动时，SessionStart Hook 会自动创建数据目录：

```bash
mkdir -p ~/.repo-knowledge/_repos
touch -a ~/.repo-knowledge/_registry.md
```

---

## Step 1: 索引第一个仓库

```
/repo-knowledge:rk-create https://github.com/org/my-project.git
```

预期输出：
```
Cloning into 'my-project'...
[Indexer Agent] Scanning 42 files...
[Indexer Agent] Generated 128 docs
Knowledge base created: my-project (128 docs)
```

SSH 格式也支持：
```
/repo-knowledge:rk-create git@github.com:org/my-project.git
```

> **注意：** SSH 格式需要在本机配置好 SSH key 并添加到 GitHub 账户。

---

## Step 2: 第一次搜索

```
/repo-knowledge:rk-search how does authentication work
```

或用中文：
```
/repo-knowledge:rk-search 认证逻辑在哪里
```

预期输出：
```
Searching my-project...
[Layer 2 semantic match] → authenticate-user.md

## authenticateUser
**File:** src/auth/authenticate.ts
...
```

---

## Step 3: 新提交后更新

当仓库有新提交时：

```
/repo-knowledge:rk-update my-project
```

预期输出：
```
Pulling latest changes...
Changed files: 3
  Modified: src/auth/authenticate.ts (2 docs regenerated)
  Added: src/utils/hash.ts (1 new doc)
  Deleted: src/auth/legacy.ts (2 docs removed)
Updated: my-project (129 docs, last commit: a1b2c3d)
```

---

## Step 4: 管理知识库

列出所有知识库：
```
/repo-knowledge:rk-list
```

预期输出：
```
| Project    | Git URL                              | Last Update | Docs |
|------------|--------------------------------------|-------------|------|
| my-project | https://github.com/org/my-project... | 2026-04-14  | 129  |
```

删除知识库：
```
/repo-knowledge:rk-delete my-project
```

系统会要求确认后再删除。

---

## 提示

- 搜索支持任意语言，中英混用均可
- 首次搜索某个概念后，下次用相似词会更快找到
- 大型仓库（1000+ 文件）索引可能需要几分钟
