# EnglishStudy LLM Wiki

这是一个由 Agent 持续维护的个人英文学习 Wiki 系统。它参考 Karpathy 的 LLM Wiki 理念：把零散英文材料放入 `raw/`，再通过 ingest、query、lint 和 review 工作流，将其编译成结构化、可检索、可复习、可输出的 `wiki/` 学习知识层。

这个项目不只是一个静态 Obsidian vault。它的核心是 Agent 维护工作流：Agent 负责阅读材料、提炼表达、更新 wiki、检索知识和生成复习；Obsidian 主要作为人工浏览、编辑和回顾的界面。

本系统的重点不是积累孤立单词，而是沉淀 **主题表达、短语搭配、句子级表达、日常口语表达和写作迁移素材**。

## 核心理念

- `raw/` 是英文学习材料收件箱与归档层，保存原始材料。
- `wiki/` 是 Agent 编译后的学习知识层，供 Agent 检索、推理和生成复习，也供人通过 Obsidian 浏览和编辑。
- 每个 raw 材料 ingest 后主要生成一个 source 总结页，而不是拆成大量碎片词条。
- 高价值表达会继续汇总到主题 `vocabulary/` 和场景 `expressions/` 页面。
- `review/` 用于按时间生成复习题，例如“复习本周内容”。

## 目录结构

```text
EnglishStudy/
├── assets/                     # 图片、PDF、音视频等媒体资产
├── raw/                        # 原始英文学习材料
│   ├── 01-articles/            # 英文文章、网页剪藏、阅读材料
│   ├── 02-transcripts/         # 视频、播客、演讲、口语材料转写稿
│   └── 09-archive/             # 已 ingest 材料归档区
│       └── YYYY-MM-DD/
│           ├── 01-articles/
│           └── 02-transcripts/
├── wiki/                       # LLM 编译后的知识层
│   ├── index.md                # 全局索引
│   ├── log.md                  # ingest/query/lint 操作日志
│   ├── sources/                # 每个 raw 材料的一对一结构化总结
│   ├── vocabulary/             # 按主题汇总的表达库
│   ├── expressions/            # 按使用场景汇总的输出表达
│   ├── review/                 # 周/月复习材料
│   │   ├── weekly/
│   │   └── monthly/
│   └── syntheses/              # 跨材料综合总结
├── .agents/skills/             # Agent 工作流说明
│   ├── ingest/                 # 将 raw 编译到 wiki，并归档源文件
│   ├── query/                  # 检索 wiki，回答英文学习问题
│   └── lint/                   # 检查知识库健康度
├── AGENTS.md                   # 项目规则与 Wiki Schema
└── README.md
```

## 常用工作流

### `/ingest`

扫描 `raw/01-articles/` 和 `raw/02-transcripts/` 中未归档的材料，将其编译为：

- `wiki/sources/摘要-*.md`
- 对应主题的 `wiki/vocabulary/*.md`
- 对应场景的 `wiki/expressions/*.md`

处理成功后，源文件会移动到：

```text
raw/09-archive/YYYY-MM-DD/<原目录>/<文件名>
```

### `/query <问题>`

从 `wiki/index.md` 出发检索本地知识库，回答英文学习问题。例如：

- `/query 整理 work 主题的口语表达`
- `/query 这个表达怎么用：take something for granted`
- `/query 基于最近材料给我造 IELTS 7 水平例句`

### `/query 复习这周内容`

根据 `wiki/log.md` 找到本周 ingest 的材料，生成复习题，包括：

- 词汇/短语英译中与中译英
- 句子补全
- 表达替换
- 话题口语对话
- 简短写作 prompt

### `/lint`

检查知识库健康度：

- 死链、孤儿页、index 未同步
- source/vocabulary/expressions 页面结构是否完整
- raw archive 是否符合 `YYYY-MM-DD/<原目录>/` 结构
- 是否存在待 ingest 材料

## 页面设计原则

- Source 页是一篇材料的主要学习入口。
- Vocabulary 页按主题组织，不按单词组织。
- Expressions 页按输出场景组织，例如日常口语、观点表达、讨论、讲故事、学术写作。
- 所有 wiki 页面必须保留 `## 关联连接`，使用 Obsidian 双链形成知识网络。
- 中文负责解释和复盘，英文负责表达、例句和练习。

## 适用对象

这个由 Agent 持续维护的个人英文学习 Wiki 系统适合已经具备中高级英文基础、希望继续积累自然表达的学习者，尤其适合：

- IELTS 7 左右学习者
- 想提升日常口语自然度的人
- 想积累主题表达和写作句型的人
- 想把阅读/听力材料转化为可输出表达的人

## 参考

- [Karpathy LLM Wiki 理念](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [karpathy-llm-wiki-vault](https://github.com/jason-effi-lab/karpathy-llm-wiki-vault)
