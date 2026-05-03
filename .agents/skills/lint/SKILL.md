---
name: lint
description: EnglishStudy Wiki 全局健康检查。扫描 wiki/ 与 raw 收件箱，检测死链、孤儿页、index 未同步、知识冲突、归档结构、wiki 是否全英文，以及 source/vocabulary/expressions/review/syntheses 是否符合英文学习知识库质量标准。只报告问题，修复必须经过用户确认。
user-invocable: true
---

# lint 技能：EnglishStudy Wiki 健康巡检

## 核心目标
检查当前精简架构是否健康：`raw/01-articles` 和 `raw/02-transcripts` 作为待处理输入，`raw/09-archive/YYYY-MM-DD/<原子目录>/` 作为归档，`wiki/` 只保留 sources、vocabulary、expressions、review、syntheses 五类英文知识输出。

## 触发条件
- 用户输入 `/lint`、`/scan`、`/health`。
- 用户询问“知识库健康状况如何”“检查知识库状态”。
- 用户要求检查 EnglishStudy Wiki 是否适合复习。

## 硬约束
- 默认仅读取和报告，禁止修改、删除、重命名任何文件。
- 修复必须等待用户明确确认。
- 不修改 `raw/` 原文内容；如需检查 raw 积压，只统计 archive 外路径和处理状态。
- 生成给用户的报告用简体中文；检查对象 `wiki/` 内容应全英文。

## 巡检流水线

### 第 1 步：目录结构检查
确认只存在以下核心目录：
- `raw/01-articles/`
- `raw/02-transcripts/`
- `raw/09-archive/`
- `wiki/sources/`
- `wiki/vocabulary/`
- `wiki/expressions/`
- `wiki/review/weekly/`
- `wiki/review/monthly/`
- `wiki/syntheses/`

如果发现旧目录，例如 `raw/02-papers/`、`raw/04-meeting_notes/`、`wiki/concepts/`、`wiki/entities/`，报告为冗余目录。

### 第 2 步：索引一致性检查
1. 读取 `wiki/index.md`。
2. 扫描 `wiki/` 下所有 `.md` 文件，排除 `index.md` 和 `log.md`。
3. 提取 index 中注册的 `[[PageName]]`。
4. 找出 index 已注册但文件不存在、文件存在但未注册的问题。
5. 检查 index 是否包含分类：`Sources`、`Vocabulary`、`Expressions`、`Review`、`Syntheses`。

### 第 3 步：双链健康检查
1. 扫描所有 wiki `.md` 文件，提取 `[[wikilink]]`。
2. 链接目标不存在时标记为死链。
3. 统计被引用页面，排除 self-reference。
4. 找出从未被其他页面引用的孤儿页面。
5. 检查每个页面是否包含 `## Related Links`。

### 第 4 步：Frontmatter 检查
检查每个 wiki 页面是否包含：
- `title`
- `type`
- `tags`
- `sources`
- `level`
- `benchmark`
- `focus`
- `last_updated`

允许 `wiki/index.md` 和 `wiki/log.md` 无 frontmatter。

### 第 5 步：Wiki 全英文检查
扫描 `wiki/` 下所有 `.md` 文件，检查正文、标题、表格字段、关联说明和日志内容是否存在明显中文字符。

允许例外：
- raw 文件路径或文件名中天然包含中文。
- 用户明确要求保存的引用原文如果本身包含中文。

如发现中文解释、中文总结或中文表格字段，标记为“语言规则违规”。

### 第 6 步：Source 页面质量检查
对 `wiki/sources/summary-*.md` 检查是否包含：
- `## Material Overview`
- `## Core Content Summary`
- `## Topic Vocabulary and Expressions`
- `## Sentence-Level Expressions`
- `## Everyday Speaking Expressions`
- `## Writing-Ready Expressions`
- `## Review Cues`
- `## Related Links`

同时检查 source 页是否链接到至少一个 `vocabulary` 主题页或 `expressions` 场景页。

### 第 7 步：Vocabulary 与 Expressions 质量检查
对 `wiki/vocabulary/*.md` 检查是否包含 topic expressions, sentence-level expressions, examples, source links, and `## Related Links`。

对 `wiki/expressions/*.md` 检查是否包含 English expression, meaning, usage context, example or mini-dialogue, source link, and `## Related Links`。

### 第 8 步：Review 与时间复习检查
1. 读取 `wiki/log.md`。
2. 检查 ingest 记录是否包含 source 链接，例如 `[[summary-*]]`。
3. 检查 ingest 记录是否包含 review cues。
4. 检查 review 页面或 Review Cues 是否以迁移式任务为主，而不是复制 source 原句。
5. 如果最近有多次 ingest 但没有 review 页面，只提示“可生成复习”，不判为错误。

Review 质量检查标准：
- 是否包含新场景、新句子、新对话或新写作任务。
- 是否避免直接复制 raw 原文或 source 页完整例句。
- 是否覆盖 expression transfer、scenario reframing、expression upgrading、mini-dialogue creation、writing transfer 中至少 3 类。
- 是否优先考察句子级表达和表达迁移，而不是孤立单词记忆。

### 第 9 步：Raw 收件箱与归档检查
1. 扫描 `raw/09-archive/` 之外的 raw 文件，排除隐藏文件和 `.DS_Store`。
2. 将 archive 外文件视为待 ingest 积压。
3. 检查 archive 内文件是否使用 `raw/09-archive/YYYY-MM-DD/<原子目录>/` 结构。
4. 检查 `wiki/sources/` 中的 `sources:` 是否指向 `raw/09-archive/YYYY-MM-DD/<原子目录>/` 下的最终归档路径。
5. 如果 source 页仍指向 archive 外路径，标记为“归档状态不一致”。

### 第 10 步：知识冲突审查
1. 全局搜索 `## Knowledge Conflicts`。
2. 提取冲突页面和简要描述。
3. 标记为认知技术债。

## 报告输出规范
输出结构化报告：

```markdown
## EnglishStudy Wiki 健康体检报告 — YYYY-MM-DD

### 绿灯项
- [运行良好的项目]

### 黄灯项
- **冗余目录**：发现 N 个旧目录。
- **未同步索引**：N 个页面存在但未注册。
- **孤儿页面**：N 个页面缺少入链。
- **待 ingest 积压**：raw/ 中 archive 外还有 N 个待处理文件。
- **归档结构问题**：N 个 archive 文件没有使用 `YYYY-MM-DD/<原子目录>/` 结构。
- **归档状态不一致**：N 个 source 页仍指向 archive 外路径。

### 红灯项
- **语言规则违规**：[页面名称] 包含中文解释、中文标题或中文表格字段。
- **复习题质量问题**：[页面名称] 大量复制原句，缺少新场景或迁移式任务。
- **死链**：[来源页面] -> [[不存在的目标页面]]
- **source 结构缺失**：[页面名称] 缺少关键英文模块。
- **表达库质量问题**：[页面名称] 缺少例句、语境或来源。
- **未解决知识冲突**：[页面名称]

### 下一步行动
1. 是否需要我修复 index 未同步问题？
2. 是否需要我补齐 source/vocabulary/expressions 页面结构？
3. 是否需要我基于最近 ingest 生成周复习？
```

## 修复规则
只有用户明确要求修复时，才可以执行写入操作。

可修复项：
- 补齐 index 分类和注册条目。
- 给页面添加缺失的 `## Related Links` 占位。
- 补齐 source/vocabulary/expressions 的标准英文标题结构。
- 追加 `wiki/log.md` 记录 lint 修复。
- 在用户确认后，将已经有对应 source 页且 log 显示成功 ingest 的 archive 外文件移动到 `raw/09-archive/YYYY-MM-DD/<原子目录>/`。

不可自动修复项：
- 不凭空编造英文表达、例句或来源。
- 不修改 raw 文件内容。
- 不删除含有用户内容的页面。
- 不静默覆盖冲突内容。
