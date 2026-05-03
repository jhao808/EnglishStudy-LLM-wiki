---
name: query
description: 在 EnglishStudy Wiki 中回答英文学习问题、整理主题表达、解释用法，并按时间范围生成复习题。必须先读取 wiki/index.md，再读取相关 source、vocabulary、expressions、review 或 syntheses 页面；回答使用 [[wikilink]] 标注本地来源。wiki 内容必须保持英文。
user-invocable: true
---

# query 技能：EnglishStudy 检索、讲解与复习助手

## 核心目标
将用户的问题转化为对本地 EnglishStudy Wiki 的检索与学习输出。默认用户定位为 **中高级英文水平学习者**，参考标准约为 IELTS 7。回答应重视主题表达、短语搭配、句子级表达、日常口语和写作迁移，而不是孤立单词释义。

## 触发场景
- 用户输入 `/query <问题>`。
- 用户询问“我的笔记里关于 X 怎么说”“整理 X 主题表达”“这个表达怎么用”。
- 用户要求“复习今天/本周/本月/最近几篇材料”。
- 用户要求生成口语对话、写作 prompt、表达替换、例句或测试题。

## 硬约束
- 必须先读取 `wiki/index.md`。
- 查询时间复习时必须读取 `wiki/log.md`，再定位相关 source 页面。
- 禁止只凭模型记忆回答本地知识库问题。
- 如果本地知识库没有相关内容，必须声明：`本地知识库中未找到相关内容，以下为通用知识回答：`
- 与用户聊天回复使用简体中文。
- 读取、引用、保存或生成任何 `wiki/` 内容时必须保持英文；不要把英文 wiki 内容翻译成中文，除非用户明确要求。
- 引用本地 wiki 信息时使用 `[[PageName]]`。

## 查询流水线

### 步骤 1：读取全局索引
读取 `wiki/index.md`，定位相关页面：
- `Sources`：材料总结页。
- `Vocabulary`：主题表达库。
- `Expressions`：场景表达库。
- `Review`：既有复习材料。
- `Syntheses`：跨材料综合总结。

### 步骤 2：判断问题类型
将用户需求归入一种或多种类型：
- **表达用法**：解释某个短语、搭配、句子模板如何使用。
- **主题整理**：整理 technology、work、education、daily life 等主题表达。
- **口语输出**：生成日常对话、IELTS-style speaking answer materials、自然回应。
- **写作输出**：整理观点表达、让步、对比、举例、总结句。
- **材料回顾**：基于某个 source 页总结语言点。
- **时间复习**：按今天、本周、本月或最近 N 篇材料生成测试。

### 步骤 3：深度读取相关页面
根据问题读取最相关的 source、vocabulary、expressions、review 或 syntheses 页面。

优先级：
- 主题问题：优先读取 `wiki/vocabulary/{topic}.md`，再读相关 source。
- 口语问题：优先读取 `wiki/expressions/daily-speaking.md`、`discussion.md`、`storytelling.md`。
- 观点表达：优先读取 `wiki/expressions/opinion-giving.md`。
- 写作问题：优先读取 `wiki/expressions/academic-writing.md`。
- 时间复习：先读 `wiki/log.md`，根据日期范围找 ingest 记录，再读对应 source 页。

### 步骤 4：生成学习型回答
回答给用户时可用中文解释任务结论，但核心英文学习内容应保持英文，例如：
- directly usable English expressions
- usage context
- upper-intermediate to advanced natural examples
- mini-dialogues or writing sentences when helpful
- `[[wikilink]]` source references

不要把回答写成词典释义。优先回答“怎么用”“在什么语境下用”“如何迁移到口语/写作”。

## 时间复习流程
当用户说“复习今天/本周/本月/最近 N 篇材料”时：
1. 读取 `wiki/log.md`。
2. 找到对应时间范围内的 `ingest` 记录。
3. 从记录中提取 `[[summary-*]]` source 链接。
4. 读取相关 source 页面。
5. 生成英文复习题。

复习题默认包括：
- **Phrase recall**：测试理解和输出。
- **Sentence completion**：测试搭配和语感。
- **Expression upgrading**：把普通表达升级成更自然表达。
- **Topic-based mini-dialogue**：围绕近期主题生成日常对话。
- **Writing prompt**：给一个短写作任务，要求使用近期表达。

如果复习内容有长期价值，询问用户是否保存到：
- `wiki/review/weekly/weekly-review-YYYY-Www.md`
- `wiki/review/monthly/monthly-review-YYYY-MM.md`

用户同意后才创建 review 页面、更新 index、追加 log。保存的 review 页面必须全英文。

## 高价值内容固化
如果回答形成了可复用主题总结、表达清单或复习材料，主动询问是否保存：
- 主题综合：保存到 `wiki/syntheses/`。
- 时间复习：保存到 `wiki/review/weekly/` 或 `wiki/review/monthly/`。
- 新表达库：合并到 `wiki/vocabulary/` 或 `wiki/expressions/`。

未得到用户确认前，不要写入文件。写入 wiki 时必须全英文。

## 记录操作日志
普通 query 回答后，若没有写入 wiki，可以不强制追加 log。

如果生成或更新了 review、synthesis、vocabulary 或 expressions 页面，必须追加英文日志：
```markdown
## [YYYY-MM-DD] query | <brief summary>
- **Output**: Saved to [[PageName]]
```

## 回答风格
- 对用户回复用中文，保持简洁。
- 英文学习材料、例句、表达清单、复习题保持英文。
- 对中高级学习者，默认提供自然但不生僻的表达。
- 口语表达要像真实对话，不要过度书面。
- 写作表达要清晰、可迁移，不堆砌复杂词。
