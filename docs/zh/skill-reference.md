# Skill API 参考

所有 Skill 均通过 Claude Code 的 `/` 命令触发。

---

## rk-create

**触发：** `/repo-knowledge:rk-create <git-url>`

**参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| `git-url` | string (required) | HTTPS 或 SSH 格式的 Git 仓库地址 |

**行为：**

1. 从 URL 推断项目名（取最后一段，去掉 `.git`）
2. 克隆仓库到 `~/.repo-knowledge/_repos/<project>/`
3. 派发 Indexer Agent 扫描所有代码文件并生成文档
4. 保存 `_meta.md`、`_index.md`，注册到 `_registry.md`

**示例：**

```
/repo-knowledge:rk-create https://github.com/anthropics/anthropic-sdk-python.git
/repo-knowledge:rk-create git@github.com:org/project.git
```

**注意：**
- 项目名已存在时会提示使用 `rk-update`
- 不支持私有仓库（需要 SSH key 或 token 配置）

---

## rk-search

**触发：** `/repo-knowledge:rk-search <query>`

**参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| `query` | string (required) | 自然语言查询，任意语言 |

**行为：**

1. 读取 `_registry.md`，确定搜索目标（单项目自动选择，多项目时提示选择）
2. 派发 Searcher Agent 执行 4 层搜索策略
3. 返回匹配的文档内容
4. 将查询词作为别名写入 `_index.md`

**示例：**

```
/repo-knowledge:rk-search how does authentication work
/repo-knowledge:rk-search 窗口大小
/repo-knowledge:rk-search JWT token validation
```

**注意：**
- 支持跨语言查询（中文查询可匹配英文文档）
- 每次成功搜索后，`_index.md` 中的别名会更新（LRU，最多 10 个）
- 缓存未命中时自动回退到源码搜索并保存新文档

---

## rk-update

**触发：** `/repo-knowledge:rk-update <project-name>`

**参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| `project-name` | string (required) | 已注册的项目名 |

**行为：**

1. 读取 `_meta.md` 获取 `last_commit`
2. `git pull` 拉取最新代码
3. `git diff --name-only <last_commit>..HEAD` 获取变更文件列表
4. 对变更文件执行增量处理：修改重建、新增添加、删除清理
5. 更新 `_meta.md` 和 `_registry.md`

**示例：**

```
/repo-knowledge:rk-update my-project
```

**注意：**
- 项目不在注册表时建议使用 `rk-create`
- 无新提交时报告 "Already up to date" 并退出

---

## rk-list

**触发：** `/repo-knowledge:rk-list`

**参数：** 无

**行为：**

读取并格式化显示 `~/.repo-knowledge/_registry.md`，展示所有已注册项目的名称、Git URL、最后更新时间和文档数量。

**示例：**

```
/repo-knowledge:rk-list
```

预期输出：
```
| Project    | Git URL                    | Last Update | Docs |
|------------|----------------------------|-------------|------|
| my-project | https://github.com/org/... | 2026-04-14  | 129  |
```

**注意：**
- 注册表为空时提示使用 `rk-create` 创建第一个知识库

---

## rk-delete

**触发：** `/repo-knowledge:rk-delete <project-name>`

**参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| `project-name` | string (required) | 要删除的项目名 |

**行为：**

1. 向用户确认是否删除
2. 删除 `~/.repo-knowledge/<project>/`（缓存目录）
3. 删除 `~/.repo-knowledge/_repos/<project>/`（克隆的仓库）
4. 从 `_registry.md` 移除对应行
5. 确认删除完成

**示例：**

```
/repo-knowledge:rk-delete my-project
```

**注意：**
- 操作不可逆；删除前系统会要求确认
- 项目不在注册表时报告 "No knowledge base found"
