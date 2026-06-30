---
type: source
aliases:
  - LLM Wiki
  - LLM Wiki 模式
  - Karpathy LLM Wiki
tags:
  - meta
  - knowledge-management
  - llm
status: draft
created: 2026-06-29
updated: 2026-06-30
source: "[[karpathy-llm-wiki]]"
related:
  - "[[CLAUDE]]"
---

# Karpathy LLM Wiki 模式

## 摘要 / Summary

一种用 LLM **增量构建并维护**个人知识库的模式:LLM 把原始信源读进来、抽取要点、**编译成一个持久、互联的 markdown wiki**,此后**只保持更新**而非每次提问从零重推。**本库(这个 vault)的 `CLAUDE.md` 就派生自这份 gist**——所以本页是这套库的「自我说明」。机械操作约定见 [[CLAUDE]],本页只沉淀**思想**。

## 核心论点 / Thesis:RAG vs 编译型知识库

- **RAG 的弊端**:每次提问都在原始文档里**临时检索、从零拼答**,知识**不积累**。要综合五篇文档的微妙问题,每次都得重新找、重新拼。
- **本模式**:知识 **编译一次、持续更新(compile once, keep current)**。交叉引用已经在、矛盾已经标注、综合已经反映你读过的一切。wiki 是**复利式产物(compounding artifact)**——每加一个源、每问一个问题都让它更富。
- 隐喻:**Obsidian = IDE,LLM = 程序员,wiki = 代码库**。

## 三层架构 + 三操作(机械层 → 见 [[CLAUDE]])

- **三层**:`sources/`(只读真相源)/ `pages/`(LLM 全权拥有的 wiki 本体)/ schema(`CLAUDE.md`,把 LLM 从聊天机器人变成**有纪律的维护者**)。
- **三操作**:**Ingest**(读源→和人过要点→写摘要页→更新 index→跨页补链→记 log,单源常触 10–15 页)/ **Query**(检索→带出处综合;**好答案回填成新页**,形态可为 md / 对比表 / Marp / matplotlib / canvas)/ **Lint**(查矛盾、过期、孤儿页、缺交叉引用、数据缺口)。
- 本库对这三操作的具体落地、页面 frontmatter 约定、Skill 选择,全部写在 [[CLAUDE]],此处不复述(避免与 schema 重复)。

## 比较优势 / 为何有效

分工是**比较优势**,不是能力高低:

- **人**:策展信源、指导分析方向、提好问题、判断「这意味着什么」。
- **LLM**:摘要、交叉引用、归档、保持一致——所有**记账杂活(grunt work)**。

人类放弃 wiki,是因为**维护负担增长快过价值**。而 LLM **不会无聊、不漏交叉引用、一次能改 15 个文件**——维护成本趋近于零,wiki 才能长期保持鲜活。

## 思想谱系 / Memex

呼应 Vannevar Bush 的 [[Memex]](1945):私人、主动策展的知识库,文档间的**关联轨迹(associative trails)**和文档本身一样重要。Bush 唯一没解决的是「**谁来做维护**」——这一环正好由 LLM 补上。

## 适用域 / Where it applies

个人成长追踪 · 长期研究(演化中的论点)· 读书伴随 wiki(角色/主题/线索,类似 fan wiki)· 团队内部 wiki(Slack/会议纪要/项目文档喂养)· 竞品分析 / 尽调 / 行程规划 / 课程笔记。

## 本库的实现选择(回填自对话)

> 这一节是 Query 讨论的**回填**——把探索沉淀成页,别让它消失在聊天记录里。

- **index = 全量目录,忠于原版**:本库一度把 `index.md` 改成人工策展的叙事主线、另设机械全量视图,后又**改回 Karpathy 原版**——`index.md` 就是全量目录(每页 link + 一句话摘要,查询先读它),不再设第二套视图层。
- **为什么不要 GUI 依赖的视图层**:曾试用 Obsidian Bases 做机械全量表,实测发现它是**查询定义、渲染依赖 GUI**——LLM 用文件工具读它只得**空配方、0 行**;真正给出全量页表的是**文件搜索(Glob / grep)**。结论:在「LLM 干活」语境里,`index.md` + **文件搜索**已足够,故移除该视图层(与原版一致)。

## 开放问题 / Open questions

- 规模超数百页后:是否引入本地搜索(如 [qmd](https://github.com/tobi/qmd),BM25/向量混合 + LLM 重排,带 CLI 与 MCP)替代「读 index」?

## 关联 / Connections

- 来源(归档):[[karpathy-llm-wiki]] · 原帖 <https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f>
- 机制实现(本库 schema):[[CLAUDE]]
- 时间线:[[log]] · 导航:[[index]]
- 相关概念:[[Memex]]
