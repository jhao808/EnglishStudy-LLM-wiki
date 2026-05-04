# 语言设定与核心角色 (Global Rules)
- **对话语言**：与用户交流时始终使用简体中文。
- **Wiki 写作语言**：所有 `wiki/` 页面必须使用英文编写，包括标题、解释、总结、表格字段、复习题、答案、关联说明和日志中描述知识内容的部分。禁止在 wiki 页面中使用中文解释英文。
- **允许中文的位置**：`AGENTS.md`、`.agents/skills/` 的规则说明、与用户的聊天回复可以使用中文；`raw/` 原文保持原样。
- **角色定义**：你正在维护一个面向 **中高级英文水平学习者** 的 EnglishStudy LLM Wiki，参考水平约为 IELTS 7。它遵循 LLM Wiki 思路：`raw/` 保存原始学习材料，`wiki/` 保存由 LLM 编译、链接、复习化的英文学习知识层。
- **学习目标**：重点服务主题词汇、短语搭配、句子级表达、日常口语表达和写作迁移；不把普通单词拆成百科式孤立页面。

# 核心目录与权限边界
- `/raw/`：英文学习材料收件箱与归档层。原始文件内容不可变，禁止修改、覆盖或删除原文内容。
- `/raw/09-archive/`：已处理材料归档区。`/ingest` 默认不扫描 archive；只有用户明确指定 archive 内文件时才可读取。
- `/assets/`：图片、PDF、音视频等媒体资产。引用时使用 Obsidian 标准语法 `![[文件名称.png]]`。
- `/wiki/`：英文编译输出层。你可以在这里创建、更新、提炼英文学习知识，并维护索引、日志、链接和复习材料。

# Raw 结构
当前 raw 架构只保留高频输入类型，避免空目录和分流过细：

```text
raw/
  01-articles/
  02-transcripts/
  09-archive/
    YYYY-MM-DD/
      01-articles/
      02-transcripts/
```

目录职责：
- `raw/01-articles/`：英文文章、网页剪藏、阅读材料、课程文字材料。
- `raw/02-transcripts/`：视频、播客、演讲、口语材料转写稿；适合提取日常口语和听力表达。
- `raw/09-archive/YYYY-MM-DD/<原子目录>/`：已成功 ingest 的原始材料归档区，保留原始文件内容和原 raw 子目录结构。

归档规则：
- `/ingest` 只扫描 `raw/09-archive/` 之外的文件。
- 只有当 source 页、主题 vocabulary 页、场景 expressions 页、`index.md` 与 `log.md` 都成功更新后，才移动源文件到 `raw/09-archive/YYYY-MM-DD/<原子目录>/`。
- source 页 frontmatter 的 `sources:` 必须记录归档后的最终路径，例如 `raw/09-archive/2026-05-03/01-articles/foo.md`。
- 如果归档时发生重名，保留内容不变，并通过文件名追加短哈希或序号来避免覆盖。
- `.DS_Store`、隐藏文件和明显非学习材料必须忽略。

# Wiki 结构
当前 wiki 架构只保留 EnglishStudy 主线：

```text
wiki/
  index.md
  log.md
  sources/
  vocabulary/
  expressions/
  review/
    weekly/
    monthly/
  syntheses/
```

目录职责：
- `wiki/sources/`：每个 raw 材料 ingest 后生成一个英文结构化总结页，是最主要的知识沉淀单位。
- `wiki/vocabulary/`：按主题汇总高价值英文表达，例如 `technology.md`、`work.md`、`daily-life.md`。
- `wiki/expressions/`：按使用场景汇总英文表达，例如 `daily-speaking.md`、`opinion-giving.md`、`discussion.md`、`storytelling.md`、`academic-writing.md`。
- `wiki/review/`：保存按时间生成的英文复习材料，例如周复习、月复习。
- `wiki/syntheses/`：保存跨材料英文主题总结、学习路线或综合分析。

# Wiki 核心文件契约
1. **`wiki/index.md` 总目录**：
   每次新增 wiki 页面后，必须同步更新此文件。所有条目说明必须用英文。格式：`[[PageName]] — One-sentence description.`

   必须保留以下分类：
   ```markdown
   # Wiki Index

   ## Sources

   ## Vocabulary

   ## Expressions

   ## Review

   ## Syntheses
   ```

2. **`wiki/log.md` 操作日志**：
   只能追加写入。每次写入操作后记录：`## [YYYY-MM-DD] <action> | <brief summary>`。
   操作类型：`ingest`、`query`、`lint`、`sync`。
   日志内容必须使用英文。

   示例：
   ```markdown
   ## [2026-05-03] ingest | Imported an article about remote work
   - **Changes**: Added [[2026-05-03-summary-remote-work-article]]; updated [[work]], [[daily-speaking]], [[index.md]]
   - **Review cues**: Generated 8 testable expressions for weekly review
   - **Archived**: Moved source file to raw/09-archive/2026-05-03/01-articles/remote-work.md
   - **Conflicts**: None
   ```

3. **页面 Frontmatter 规范**：
   所有生成的 wiki 页面必须包含 YAML 头部：
   ```yaml
   ---
   title: "Page Title"
   type: source | vocabulary | expression | review | synthesis
   tags: [english-study, topic-tag]
   sources: [raw/09-archive/YYYY-MM-DD/original-folder/file-name]
   level: upper-intermediate to advanced
   benchmark: IELTS 7
   focus: vocabulary | expression | speaking | writing | review
   last_updated: YYYY-MM-DD
   ---
   ```

4. **强制关联连接**：
   每个 wiki 页面必须包含 `## Related Links` 区域，使用 Obsidian 双链 `[[PageName]]` 链接到相关 source、主题词汇页、表达页、复习页或综合页。关联说明必须使用英文。

5. **矛盾处理原则**：
   如果新知识与旧知识冲突，不要静默覆盖。在相关页面中新建 `## Knowledge Conflicts` 区块，将两种说法都保留并做英文对比。

# Source 页面模板
每个 raw 材料主要生成一个 `wiki/sources/YYYY-MM-DD-summary-*.md`。日期优先使用归档路径中的日期，例如 `raw/09-archive/2026-05-03/...`；如果尚未归档，则使用 ingest 当天日期。source 文件名、frontmatter `title` 与归档日期必须保持一致。

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

## Material Overview
Summarize the topic, context, main ideas, and language-learning value of the material in English.

## Core Content Summary
Extract the main arguments, narrative structure, examples, or conversational context.

## Topic Vocabulary and Expressions
Prioritize phrases, collocations, and sentence-level expressions over isolated words.

| Expression | Meaning | Context | Transferable Example |
| --- | --- | --- | --- |

## Sentence-Level Expressions
Collect natural expressions that can be transferred into speaking or writing.

| Original Expression | Usage Note | Reusable Pattern |
| --- | --- | --- |

## Everyday Speaking Expressions
Extract natural, speakable expressions if the material supports spoken English practice.

## Writing-Ready Expressions
Extract useful expressions for opinions, contrast, concession, examples, and conclusions.

## Review Cues
Generate 5-10 transferable review cues. Each cue should identify an expression or pattern and suggest a new context where it could be reused. Do not copy full source sentences as future questions.

## Related Links
- [[topic-page]] — Related topic vocabulary.
- [[daily-speaking]] — Related speaking expressions.
```

# Vocabulary 与 Expressions 汇总规则
- `vocabulary/` 不按单词建页，而是按主题建页。
- `expressions/` 按使用场景建页。
- 只有高频、跨材料可复用、适合中高级英文学习者输出的表达才汇总到主题或场景页。
- 优先沉淀短语、搭配、句子级表达和可迁移模板，而不是孤立单词。
- 每条表达必须尽量包含英文释义、适用场景、例句和来源链接。

# 工作流指令
- `/ingest <路径>`：读取指定 raw 材料，生成一个英文结构化 `YYYY-MM-DD-summary-{source-slug}` source 总结页，把高价值表达合并到 `vocabulary/` 与 `expressions/`，更新 `index.md` 和 `log.md`，成功后归档源文件。
- `/query <问题>`：先读取 `wiki/index.md`，再读取相关 source、vocabulary、expressions、review 或 syntheses 页面，综合回答并用 `[[wikilink]]` 标注引用。用户回复用中文，但引用和总结 wiki 内容时不要把英文页面翻译成中文，除非用户明确要求。
- `/query 复习这周内容`：根据 `wiki/log.md` 的时间范围找到相关 ingest 记录，读取对应 source 页面，生成英文迁移式复习题。题目必须基于已有表达创造新句子、新场景、新对话或新写作任务，不能直接复制原文。
- `/lint`：扫描 wiki 与 raw 收件箱健康度，检查死链、孤儿页、index 未同步、知识冲突、归档结构、wiki 是否全英文，以及 EnglishStudy 页面质量。

# 默认学习偏好
- 用户当前水平定位为中高级英文水平学习者，参考标准约为 IELTS 7。
- 默认优先输出自然、地道、可迁移的句子级表达。
- 日常口语表达优先级高于冷僻词汇。
- 写作表达应关注观点、让步、对比、举例、总结等功能。
- 若用户没有指定主题，优先从材料本身推断主题，并使用较宽的主题页，例如 `work`、`technology`、`education`、`daily-life`。
