# User Stories & Use Cases

每个 User Story 来自 [epic.md](epic.md)，在此展开为具体的 Use Case，包含前置条件、主流程、异常流程和后置条件。

---

## US-1.1 -- 索引新仓库

### UC-1.1.1: 索引公开 GitHub 仓库

**前置条件**
- Claude Code 已加载 repo-knowledge 插件
- 用户有网络访问权限，可以克隆仓库

**主流程**
1. 用户输入 `/repo-knowledge:rk-create https://github.com/org/project.git`
2. 系统将仓库克隆到 `~/.repo-knowledge/_repos/project/`
3. 系统创建缓存目录 `~/.repo-knowledge/project/docs/`
4. 索引 Agent 扫描所有代码文件（*.ts、*.py、*.go 等）
5. 对每个发现的函数/类，Agent 生成文档条目并保存到缓存
6. 系统构建 `_index.md`，包含所有条目的单行摘要
7. 系统保存 `_meta.md`，记录 git URL、创建日期、最新提交哈希和文档数量
8. 系统将项目行追加到 `~/.repo-knowledge/_registry.md`
9. 系统报告："已索引 `project`：生成 N 份文档"

**异常流程**
- *克隆时网络失败*：系统报告克隆错误；不写入任何中间状态
- *空仓库*：系统报告"未找到代码文件"并跳过索引创建
- *注册表中已存在该项目*：系统警告用户并建议改用 `rk-update`

**后置条件**
- `~/.repo-knowledge/project/_index.md` 存在，包含所有文档条目
- 项目行已存在于 `_registry.md`
- 知识库已可供 `rk-search` 使用

---

## US-2.1 -- 自然语言搜索

### UC-2.1.1: 搜索在缓存中命中

**前置条件**
- `_registry.md` 中至少存在一个知识库
- 目标项目已构建 `_index.md`

**主流程**
1. 用户输入 `/repo-knowledge:rk-search 如何验证用户身份`
2. 系统识别目标项目（若只有一个则自动选择；有多个则提示用户）
3. 搜索 Agent 读取 `_index.md`
4. Agent 扫描别名——未找到直接别名匹配
5. Agent 使用语义理解对索引条目排序，选定 `validate-token` 为最佳匹配
6. Agent 读取 `validate-token` 的缓存文档
7. Agent 判断：文档回答了查询问题 ✓
8. Agent 将"如何验证用户身份"前置到 `_index.md` 中 `validate-token` 的别名列表
9. 系统将缓存文档内容返回给用户

**异常流程**
- *步骤 4 命中别名*：跳过语义排序，直接进入缓存文档验证
- *首个候选验证失败*：尝试下一个候选（最多 3 次）
- *3 个候选均失败*：进入 UC-2.1.2（源码回退）

**后置条件**
- 用户获得相关文档
- 新别名已添加到 `_index.md`，加速后续查找

### UC-2.1.2: 搜索回退到源码

**前置条件**
- 知识库存在，但缓存中没有相关条目

**主流程**
1. 经过 3 次缓存候选失败后，Agent 切换到源码搜索
2. Agent 使用 Glob 查找 `~/.repo-knowledge/_repos/project/` 中的所有代码文件
3. Agent 使用 Grep 对查询词执行至少 3 种同义变体搜索
4. Agent 读取匹配文件段落以获取上下文
5. Agent 按标准格式生成新文档条目
6. Agent 将新 `.md` 文件保存到正确的缓存目录
7. Agent 将新条目追加到 `_index.md`，以查询词作为第一个别名
8. 系统将新生成的文档返回给用户

**后置条件**
- 新文档已缓存；同概念的后续搜索将在 Layer 1（别名匹配）命中缓存

---

## US-3.1 -- 增量更新

### UC-3.1.1: 有新提交后进行更新

**前置条件**
- 项目已在注册表中，并记录了 `last_commit` 哈希
- 远程仓库自上次更新以来有新提交

**主流程**
1. 用户输入 `/repo-knowledge:rk-update project`
2. 系统从 `~/.repo-knowledge/project/_meta.md` 读取 `last_commit`
3. 系统在 `~/.repo-knowledge/_repos/project/` 中执行 `git pull`
4. 系统运行 `git diff --name-only <last_commit>..HEAD` 获取变更文件列表
5. 更新 Agent 处理每个变更文件：
   - **已修改**：重新生成该文件中所有函数/类的文档
   - **已新增**：生成新文档，将条目追加到 `_index.md`
   - **已删除**：删除对应的缓存 `.md` 文件，从 `_index.md` 移除条目
6. 系统更新 `_meta.md`，写入新的 `last_commit`、`last_update`、`total_docs`
7. 系统更新 `_registry.md` 中的项目行，写入新日期和文档数量
8. 系统报告汇总：修改/新增/删除的数量

**异常流程**
- *无新提交*：系统报告"已是最新"并退出
- *项目不在注册表中*：系统建议改用 `rk-create`

**后置条件**
- 缓存反映代码库的当前状态
- `_meta.md` 已更新 `last_commit` 哈希

---

## US-4.2 -- 删除知识库

### UC-4.2.1: 确认后删除

**前置条件**
- 项目存在于注册表中

**主流程**
1. 用户输入 `/repo-knowledge:rk-delete project`
2. 系统询问："确定要删除 `project` 的知识库吗？此操作将移除所有缓存文档。"
3. 用户确认
4. 系统删除 `~/.repo-knowledge/project/`（整个缓存目录）
5. 系统删除 `~/.repo-knowledge/_repos/project/`（克隆的仓库）
6. 系统从 `_registry.md` 中移除项目行
7. 系统确认："已删除 `project` 的知识库。"

**异常流程**
- *用户拒绝确认*：系统取消操作，不删除任何内容
- *项目不在注册表中*：系统报告"未找到 `project` 的知识库"

**后置条件**
- `~/.repo-knowledge/` 下不再存在该项目的任何文件
- 项目行已从 `_registry.md` 移除

---

## US-1.2 -- 函数级文档

### UC-1.2.1: 每个函数独立生成文档条目

**前置条件**
- 已使用 `rk-create` 对仓库完成索引

**主流程**
1. 索引 Agent 读取一个源文件（例如 `src/auth/validate-token.ts`）
2. Agent 识别文件中的每个函数和类
3. 对每个函数，Agent 生成专属 Markdown 文档，包含：名称、文件路径、功能说明（中英文）、代码、参数、返回值、使用示例、注意事项
4. 文档保存到 `~/.repo-knowledge/<project>/docs/src/auth/validate-token.md`
5. 一行摘要条目追加到 `_index.md`

**异常流程**
- *函数超过 100 行*：拆分为逻辑段落，每段独立生成文档
- *类超过 200 行*：按方法拆分，每个方法独立生成文档

**后置条件**
- 每个函数/类（或大类的每个方法）对应一个 `.md` 文件
- `_index.md` 中每份文档均有一行摘要条目

---

## US-1.3 -- 统一注册表

### UC-1.3.1: 索引完成后项目注册

**前置条件**
- `rk-create` 已成功完成某项目的索引

**主流程**
1. 系统读取当前的 `~/.repo-knowledge/_registry.md`
2. 若注册表不存在，系统创建并写入表头行
3. 系统追加新行：`| project-name | git-url | YYYY-MM-DD | doc-count |`
4. 注册表现可被 `rk-list`、`rk-search`、`rk-update` 和 `rk-delete` 查询

**异常流程**
- *项目已在注册表中*：系统跳过追加；保留已有行

**后置条件**
- `_registry.md` 包含该项目的行
- 所有其他技能可通过 `_registry.md` 发现该项目

---

## US-2.2 -- 渐进式别名学习

### UC-2.2.1: 搜索成功后添加别名

**前置条件**
- 某次搜索查询已返回相关结果

**主流程**
1. 搜索 Agent 返回一份回答用户查询的缓存文档
2. Agent 将查询字符串前置到 `_index.md` 中目标条目的 `aliases` 列表
3. 若别名列表已有 10 条，最旧的（末尾）条目被移除（LRU 淘汰）
4. 下次使用相同或相似查询搜索时，Layer 1（别名匹配）立即触发

**异常流程**
- *用户指出结果有误*：Agent 从该条目移除新添加的别名

**后置条件**
- `_index.md` 条目的别名列表已前置该查询字符串
- 后续相同查询在 Layer 1 直接解析，无需语义排序

---

## US-2.3 -- 源码回退搜索

### UC-2.3.1: 缓存未命中触发源码搜索

**前置条件**
- 3 个缓存候选均未能回答查询（Layer 3 已耗尽）

**主流程**
1. 搜索 Agent 切换到 Layer 4（源码搜索）
2. Agent 使用 Glob 列出 `~/.repo-knowledge/_repos/<project>/` 中的所有代码文件
3. Agent 使用 Grep 对原始查询执行至少 3 种同义变体搜索
4. Agent 读取最相关的匹配代码段
5. Agent 按标准格式生成新文档条目（缓存日期、来源路径、代码、参数、返回值、示例）
6. Agent 将新 `.md` 文件保存到正确的缓存目录
7. Agent 将新条目追加到 `_index.md`，以查询词作为第一个别名
8. 系统将新生成的文档返回给用户

**后置条件**
- 之前未缓存概念的新文档已存入缓存
- 后续对同一概念的搜索在 Layer 1（别名匹配）解析

---

## US-3.2 -- 自动清理已删除文件

### UC-3.2.1: 已删除源文件从缓存中移除

**前置条件**
- 自上次 `rk-update` 后，仓库中有文件被删除

**主流程**
1. `rk-update` 运行 `git diff --name-only <last_commit>..HEAD`
2. 被删除文件出现在 diff 输出中
3. 更新 Agent 查找所有 `source:` frontmatter 与被删除文件路径匹配的缓存 `.md` 文件
4. Agent 从磁盘删除这些缓存 `.md` 文件
5. Agent 从 `_index.md` 移除对应条目
6. `_meta.md` 的 `total_docs` 计数相应递减

**后置条件**
- 没有任何缓存文档引用已删除的源文件
- `_index.md` 中不包含指向已删除文档的条目
- 搜索结果中不会出现已不存在的代码

---

## US-4.1 -- 列出所有知识库

### UC-4.1.1: 查看已注册的代码库

**前置条件**
- Claude Code 已加载 repo-knowledge 插件

**主流程**
1. 用户输入 `/repo-knowledge:rk-list`
2. 系统读取 `~/.repo-knowledge/_registry.md`
3. 系统格式化并展示注册表，显示：项目名称、git URL、最近更新日期、文档数量
4. 用户一览已索引的代码库及其新鲜度

**异常流程**
- *注册表为空或不存在*：系统显示"未找到知识库。请使用 `/repo-knowledge:rk-create <git-url>` 创建一个。"

**后置条件**
- 用户已获得所有已索引知识库的完整视图
- 未修改任何状态
