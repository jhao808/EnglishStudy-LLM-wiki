---
name: ingest
description: 将 raw/01-articles 与 raw/02-transcripts 中的英文学习材料编译为 EnglishStudy Wiki。每个 raw 材料主要生成一个全英文 source 总结页，并把高价值主题表达汇总到 vocabulary/ 与 expressions/。成功处理后将源文件整体移动到 raw/09-archive/YYYY-MM-DD/<原子目录>/，避免重复扫描；绝不修改原文内容。
user-invocable: true
---

# ingest 技能：EnglishStudy 材料编译器

## 核心目标
你正在维护一个面向 **中高级英文水平学习者** 的 EnglishStudy LLM Wiki，参考标准约为 IELTS 7。`raw/` 是收件箱与归档层，`wiki/` 是全英文编译输出层。原始文件内容不可变，但文件在成功 ingest 后必须整体移动到 `raw/09-archive/YYYY-MM-DD/<原子目录>/`，避免下次重复扫描。

ingest 的主要产物不是大量孤立词条页，而是：
- 一个全英文结构化 `wiki/sources/YYYY-MM-DD-summary-*.md` 材料总结页。
- 少量按主题合并的 `wiki/vocabulary/{topic}.md` 页面。
- 少量按使用场景合并的 `wiki/expressions/{scenario}.md` 页面。

## 目录约定
- `raw/01-articles/` — 英文文章、网页剪藏、阅读材料、课程文字材料。
- `raw/02-transcripts/` — 视频、播客、演讲、口语材料转写稿。
- `raw/09-archive/YYYY-MM-DD/<原子目录>/` — 已成功 ingest 的源文件归档区，默认不扫描。
- `wiki/sources/` — 每个 raw 材料对应一个全英文 source 总结页。
- `wiki/vocabulary/` — 按主题汇总英文表达，例如 `technology.md`、`education.md`、`work.md`、`daily-life.md`。
- `wiki/expressions/` — 按使用场景汇总英文表达，例如 `daily-speaking.md`、`opinion-giving.md`、`discussion.md`、`storytelling.md`、`academic-writing.md`。
- `wiki/review/` — 保存按时间生成的英文复习材料；ingest 只生成复习线索，不主动创建 review，除非用户要求。
- `wiki/syntheses/` — 跨材料英文主题总结或学习路线。

## 触发逻辑
1. 用户执行 `/ingest`：扫描 `raw/01-articles/` 与 `raw/02-transcripts/` 下未归档的学习材料，跳过隐藏文件、系统文件和明显非文本文件。
2. 用户执行 `/ingest <path>`：仅处理指定 raw 文件；如果路径位于 `raw/09-archive/`，必须确认这是用户明确指定的重读/重建请求。
3. 用户说“摄入”“导入”“收入资料”“加入知识库”等，也触发本技能。

## 硬约束
- 禁止修改、覆盖或删除 `raw/` 下任何原始文件内容。
- 允许且必须在成功 ingest 后将源文件整体移动到 `raw/09-archive/YYYY-MM-DD/<原子目录>/`。
- 归档移动必须保留原 raw 子目录结构。例如 `raw/02-transcripts/a.md` 归档为 `raw/09-archive/2026-05-03/02-transcripts/a.md`。
- 归档移动必须发生在 source 页、vocabulary/expressions 汇总页、`index.md` 与 `log.md` 全部更新成功之后。
- `/ingest` 默认只扫描 `raw/09-archive/` 之外的待处理文件；archive 内文件只有用户明确指定时才读取。
- `wiki/` 中所有新增或更新内容必须使用英文，包括标题、解释、总结、表格字段、复习提示、关联说明和日志。
- 不要在 wiki 页面中使用中文解释英文；与用户聊天时仍使用简体中文汇报进度和结果。
- 每个生成或更新的 wiki 页面必须包含 YAML frontmatter 与 `## Related Links`。
- 必须更新 `wiki/index.md` 并追加 `wiki/log.md`。

## 编译流水线

### 步骤 1：定位材料
- `/ingest <path>`：读取指定文件。
- `/ingest`：只扫描 `raw/01-articles/` 与 `raw/02-transcripts/`，排除 `.DS_Store`、隐藏文件和无法提取文本的二进制文件。
- 文件是否仍在 archive 外就是主要待处理信号。
- 如果 archive 外文件已在现有 source 页的 `sources:` 中出现，说明上次归档可能失败，应报告并跳过，避免重复 ingest。

### 步骤 2：读取源文件
- `.md` / `.txt`：完整读取。
- `.pdf`：仅当用户明确放入当前 raw 输入目录时尝试提取文本；如果无法提取，则创建只包含元信息和待处理说明的 source 页。
- 转写稿、写作批改、课程笔记都按文本材料处理；如果这类材料长期增多，再新增 raw 子目录。

### 步骤 3：生成 source 总结页
在 `wiki/sources/` 创建或更新 `YYYY-MM-DD-summary-{source-slug}.md`：
- `YYYY-MM-DD` 优先使用最终归档路径中的日期，例如 `raw/09-archive/2026-05-03/...`。
- 如果源文件尚未归档，先按 ingest 当天日期生成目标归档路径，再用同一日期命名 source 页。
- frontmatter `title` 必须等于不含 `.md` 的 source 文件名。
- 新 ingest 禁止生成旧格式 `summary-{source-slug}.md`；旧格式只可作为兼容读取对象。

```markdown
---
title: "YYYY-MM-DD-summary-source-slug"
type: source
tags: [english-study, topic]
sources: [raw/09-archive/YYYY-MM-DD/original-folder/file-name]
level: upper-intermediate to advanced
benchmark: IELTS 7
focus: expression
last_updated: YYYY-MM-DD
---

## Material Snapshot
Briefly identify the material type, topic, context, speaker or author stance, and language-learning value. Keep this section short; the source page is primarily an expression bank, not a content summary.

## Topic Vocabulary
Extract advanced topic-specific words, especially analytical, evaluative, abstract, or accuracy-enhancing vocabulary. Avoid basic words unless they form part of a high-value expression.

| Word | Part of Speech | Meaning | Source Context | Reusable Example |
| --- | --- | --- | --- | --- |

## Topic Expressions
Extract topic-related phrases, collocations, advanced expressions, and conceptual expressions.

| Expression | Meaning | Use Case | Reusable Example |
| --- | --- | --- | --- |

## Sentence-Level Expressions
This is the most important section. Extract high-value original sentences or sentence-length expressions directly from the raw material, especially topic-rich sentences, long spoken explanations, complex claims, contrast, concession, cause-effect reasoning, evaluation, and summary lines. A reusable structure is helpful but not required; do not reduce this section to short generic patterns.

| Original Sentence / Expression | Why It Matters | Function | Reusable Structure | Transferable Scenarios | Learner Example |
| --- | --- | --- | --- | --- | --- |

## Everyday Speaking Expressions
For transcripts, extract short, natural, spoken expressions that are useful in daily conversation, discussion, or speaking tests. For articles, keep this section short or omit low-value items.

## Writing-Ready Expressions
For articles, extract formal expressions for essays, reports, and argumentative writing. For transcripts, include only a small number of transferable opinion or discourse expressions; do not force academic writing from casual speech.

## Review Cues
Generate 5-10 transferable review cues. Each cue should identify an expression or pattern and suggest a new context where it could be reused. Do not copy full source sentences as future questions.

## Related Links
- [[topic-page]] — Related topic vocabulary.
- [[daily-speaking]] — Related speaking expressions.
```

### 步骤 4：识别主题与场景
从材料中判断 1-3 个主题，用于更新 `vocabulary/`：
- 常见主题：`technology`、`education`、`work`、`daily-life`、`culture`、`society`、`travel`、`health`。
- 如果主题不确定，使用较宽泛主题；不要创建过细主题页。

从表达功能判断 1-3 个场景，用于更新 `expressions/`：
- `daily-speaking`：日常口语、自然回应、寒暄、生活表达。
- `opinion-giving`：表达观点、同意、反对、让步。
- `discussion`：讨论、追问、澄清、回应。
- `storytelling`：讲经历、描述变化、表达感受。
- `academic-writing`：写作表达、论证、转折、举例、总结。

### 步骤 5：汇总到 vocabulary 主题页
将高价值、可跨材料复用的表达合并到 `wiki/vocabulary/{topic}.md`。

只收录：
- 高级 topic vocabulary，尤其是与主题强相关、分析性、评价性、抽象性或能提升表达准确度的单词。
- 高频主题搭配。
- 可迁移短语。
- 概念性表达。
- 适合中高级英文学习者口语或写作输出的自然话题表达。

来源与同步要求：
- `vocabulary/` 可以包含比 source 初稿更多的表达，但新增表达必须直接来自 raw 原文，或是对 raw 原文表达的明确模板化抽象。
- 如果在汇总到 `vocabulary/` 时发现了新的高价值表达，必须同步补回对应 source 页的 `## Topic Vocabulary`、`## Topic Expressions` 或 `## Sentence-Level Expressions`。
- 不得只凭模型常识向主题页添加没有 raw 依据的表达。

### 步骤 6：汇总到 expressions 场景页
将表达按用途合并到 `wiki/expressions/{scenario}.md`。优先合并 sentence-level expressions、daily speaking expressions、opinion/discussion moves and writing-ready expressions。每条表达尽量包含 English expression, meaning, usage context, natural example or mini-dialogue, and source link.

来源与同步要求：
- `expressions/` 可以包含比 source 初稿更多的口语、观点或写作表达，但新增表达必须直接来自 raw 原文，或是对 raw 原文句式的明确模板化抽象。
- 如果在汇总到 `expressions/` 时发现了新的高价值表达，必须同步补回对应 source 页的 `## Everyday Speaking Expressions`、`## Writing-Ready Expressions` 或 `## Sentence-Level Expressions`。
- 汇总页中的 source link 必须指向已经同步包含该表达的 source 页。

### 步骤 7：更新 index 与 log
更新 `wiki/index.md`：
- `Sources` 注册 source 页。
- `Vocabulary` 注册新建或更新的主题页。
- `Expressions` 注册新建或更新的场景页。

追加 `wiki/log.md`，内容必须用英文：
```markdown
## [YYYY-MM-DD] ingest | Imported <material name>
- **Changes**: Added/updated [[YYYY-MM-DD-summary-source-slug]], [[topic]], [[scenario]]; updated [[index.md]]
- **Review cues**: Generated N testable expressions for weekly review
- **Archived**: Moved source file to raw/09-archive/YYYY-MM-DD/<original-folder>/<file>
- **Conflicts**: None
```

### 步骤 8：归档源文件
在确认 source、vocabulary/expressions、`index.md` 与 `log.md` 全部完成后，移动源文件到 `raw/09-archive/YYYY-MM-DD/<原子目录>/`。

归档要求：
- 只移动整个文件，不修改文件内容。
- 必须保留原 raw 子目录结构。
- 如果目标目录中已有同名文件，不得覆盖；应使用短哈希或序号生成唯一文件名。
- source 页 frontmatter 的 `sources:` 应指向归档后的最终路径。
- 如果归档失败，必须在最终回复和 log 中说明。

## 冲突处理
如果新材料与既有解释、表达用法或来源信息冲突：
1. 不要静默覆盖。
2. 在相关页面添加 `## Knowledge Conflicts`，保留两种说法。
3. 在 log 中记录冲突页面。
4. 如果冲突影响学习结论，暂停并向用户说明。
5. 冲突未处理完成前，不要归档源文件。

## 质量标准
- Source 页应能独立复习一篇材料。
- Source 页应是单篇材料可沉淀表达的母版；凡是进入 `vocabulary/` 或 `expressions/` 的新增表达，都应能在 source 页相关模块中找到对应表达或模板。
- Source 页的重点不是内容总结，而是可复用高级词汇、话题表达、完整句子级表达、口语表达或写作表达。
- Topic Vocabulary 必须包含高价值单词级词汇；不要只提取短语，也不要收 good/bad/work/life 等过于基础的词。
- Sentence-Level Expressions 是最重要模块，必须优先包含 raw 原文中的高价值完整句子或长句表达；句型结构只是辅助分析，不是入选前提。
- 避免只列 `What I need you to realize is...` 这类过短、过泛的模板；除非该表达本身非常自然、高频且有明确话题价值。
- Transcript 应重点提取 Everyday Speaking Expressions；article 应重点提取 Writing-Ready Expressions。
- Vocabulary 页应像“主题词汇与表达库”，而不是随机单词表。
- Expressions 页应像“输出工具箱”，而不是短语堆积。
- Review cues 必须能转化为迁移式题目，例如 expression transfer, scenario reframing, expression upgrading, mini-dialogue creation, writing transfer.
- Review cues 不应复制原文整句；只能保留目标表达、用法功能和可迁移的新场景。
- 每个 source 页的 Review Cues 应尽量包含新场景建议，例如 workplace, casual conversation, study, travel, health, technology, relationships。
