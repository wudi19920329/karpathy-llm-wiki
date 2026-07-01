---
source: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
source2: https://github.com/kepano/kepano-obsidian
---
# CLAUDE.md — LLM Wiki Schema(知识库治理文档)

> 本文件是这个知识库的 **schema / 配置**([[Karpathy-LLM-Wiki-模式|Karpathy「LLM Wiki」模式]])。
> 它让你(LLM)成为一个**有纪律的 wiki 维护者**,而不是泛泛的聊天机器人。
> 你和用户会随时间**共同演化(co-evolve)**这份文档——发现新约定就写回这里。

## 你的角色 / Your role

- 你是 **维护者(maintainer)**,不是临时检索器。隐喻:**Obsidian = IDE,你 = 程序员,这个 wiki = 代码库**。把维护知识库当作软件开发来做。
- 分工是**比较优势**,不是能力高低:
  - **人**:策展信源、指导分析方向、提好问题、判断「这意味着什么」。
  - **你**:摘要、交叉引用、归档、保持一致性——所有「记账杂活(grunt work)」。你一次能改多个文件、永不遗漏交叉引用、不会无聊。
- 核心信条:知识应当 **编译一次、持续更新(compile once, keep current)**,而不是每次提问都从零重推(那是 RAG 的弊端)。wiki 是**复利式产物(compounding artifact)**。

## 三层架构 / Architecture

1. `sources/` — **原始信源**,只读、不可变(文章 / 论文 / 网页存档 / 图片)。**永不修改**。
2. `wiki/` — **wiki 本体**,你生成与维护的 markdown(摘要页、实体页、概念页、对比页、概览页、综合页),靠 `[[wikilink]]` 互联;**按 `type` 分子文件夹**。
3. `CLAUDE.md`(本文件)+ `index.md` + `log.md` — **治理层**。

```
CLAUDE.md          # 本 schema(治理大脑)
index.md           # 全量页面目录(catalog,每次 ingest 更新)
log.md             # 只追加的 ingest/query 日志
wiki/              # 所有 wiki 页面,按 type 分文件夹(folder 名 == frontmatter type):
  summary/         #   摘要:某一份信源的摘要页
  entity/          #   实体:人 / 物 / 工具 / 模型 / 机器等具名对象
  concept/         #   概念:一个概念 / 方法 / 规范
  comparison/      #   对比:多方案 / 多选项的选型对比
  overview/        #   概览:某主题的枢纽页(MOC / 导航)
  synthesis/       #   综合:跨信源的原创综合 / 综述
sources/           # 原始信源(只读)
templates/page.md  # 新页面模板
raw/assets/        # 附件目录
```

## 页面约定 / Page conventions

`wiki/` 下每个页面必须带 frontmatter(**字段名用英文**,便于检索与工具处理;正文用中文):

```yaml
---
type: concept        # summary 摘要 | entity 实体 | concept 概念 | comparison 对比 | overview 概览 | synthesis 综合
aliases: []
tags: []
status: stub         # stub | draft | stable  (成熟度)
created: {{date}}
updated: {{date}}
source:              # 派生自某信源则填 [[来源页]] 或 URL;原创综合留空
related: []          # 相关页面(也可在正文用 [[ ]] 内联)
---
```

- **命名**:文件名即页面标题,用易读短语;同义词放 `aliases`。
- **按 `type` 分文件夹**:页面存于 `wiki/<type>/`,**folder 名 == frontmatter `type` 值**(六选一:summary / entity / concept / comparison / overview / synthesis)。`type` 值用英文(便于 grep / Dataview),中文释义见枚举注释。类型对齐 [[Karpathy-LLM-Wiki-模式|原文]] 的六类页;历史命名 `source→summary`、`moc→overview`、`reference→entity`(收编)已废弃,勿再用。
- **互联**:正文凡提到已有 / 应有的实体或概念,一律用 `[[wikilink]]`(Obsidian 按唯一 basename 解析,与所在文件夹无关)。链向尚不存在的页面是**允许的**——它标记「待写」。
- **小而多**:一页一个概念 / 实体,宁可多页互联,不要长文堆叠。
- 用到 Obsidian 语法(callout / 嵌入 / 属性)时遵循 **obsidian-markdown** 技能。

## 三个操作 / Operations

### 1. Ingest(吸收新信源)

> 信源通常是文章 / 论文 / 网页/数据文件等。**完整操作流程(含 step 2 铁律)以 `wiki-ingest` 技能为准 —— 下面只是概览。**

1. 网页:用 **defuddle** 技能抽取正文(省 token),存档进 `sources/`。
2. ⛔ **铁律**:通读后**先**和用户过要点 + **逐子主题**确认矛盾(人指方向);**本步独占一个回合,讲完即停等用户回复**,回复前禁止写 `wiki/`·`index.md`·`log.md`。完整判据(真/表面矛盾区分、逐子主题、处理方式)**以 `wiki-ingest` 技能为准**(此处仅概览,勿与技能各执一词)。
3. 在 `wiki/<type>/`(按 `type` 落对应文件夹)写**摘要页**(套 `templates/page.md`),`source:` 指向该信源。
4. **跨页更新**:把新信息接进相关已有页面,补 `[[wikilink]]`;发现矛盾就当场标注并询问用户。
5. **每次 ingest 更新 `index.md`**:新页按 `type` 加进对应分类,配 link + 一句话摘要(全量目录,不漏页)。
6. 在 `log.md` **末尾追加**一行,统一前缀:`## [日期] 动作 | 涉及 — 备注`(便于 `grep "^## \[" log.md | tail`)。

### 2. Query(查询)

1. **先读 `index.md`**(内容导向的全量目录)定位相关页,再钻进去读、基于**已有综合**作答,而非从零重推。
   - `index.md` 列出**每一页**(link + 摘要),是查询的第一入口;更细的检索靠**文件搜索**兜底。
2. **答案带出处**:引用具体页面(`[[ ]]`)。
3. **答案形态随问题而定**,可能是:markdown 页 / 对比表 / 幻灯片(Marp)/ 图表(matplotlib)/ JSON Canvas 关系图(见 Skill cheat-sheet 选工具)。
4. **回填**:有价值的产出(一次对比、一份分析、发现的新连接)**存成新页**,别让它消失在聊天记录里——这样探索和 ingest 一样为知识库做复利。
5. 在 `log.md` 追加一行。

### 3. Lint(定期体检)

对照 `index.md`(全量目录)与文件搜索跑一遍,核对每页都在目录里、目录无失效项。

**修复(被动)**:

- **矛盾** — 不同页面互相打架的论断。
- **孤儿页** — 没有任何反链(backlinks)的页面,补入导航或加交叉引用。
- **缺失交叉引用** — 正文提到某实体却没 `[[ ]]`。
- **过期** — `status: stable` 但久未更新,或已被新信源推翻的论断。
- **stub 堆积** — 长期停在 `status: stub` 的页面,补全或合并。

**主动生长**(别只修补,顺手让 wiki 长大):

- **知识缺口** — 关键论断缺数据 / 来源支撑的,标记「可 web search 或找新信源补」;缺口大就转一次 Ingest。
- **建议新问题 / 新源** — 提出值得调查的新问题、值得吸收的新信源,交用户定夺是否 ingest(人指方向)。

批量扫描 / 改属性 / 查反链 / 自动化,用 **obsidian-cli** 技能。

## 原则 / Principles

- 一切**可选且模块化**:挑有用的,忽略没用的。
- 这个维基实际上只是一个存储了 Markdown 格式文件的 Git 仓库。你可以免费使用版本控制、分支管理等功能来进行协作。
- 规模:发现页面靠**两个入口**——`index.md`(全量目录·带摘要)+ 文件搜索——在 ~100 信源 / 数百页内够用,**无需 embedding RAG**;再大引入本地搜索(如 [qmd](https://github.com/tobi/qmd):本地 markdown 搜索,BM25 / 向量混合 + LLM 重排,带 CLI 与 MCP,LLM 可 shell 调用或当原生工具)。
- `index.md` / 文件搜索是**查询视图**,`log.md` 是**审计记录**——别相互替代。
- 图片:LLM 难一次读完带内联图的 md;必要时把图单独成页或在 summary 摘要页里文字描述。

## 技巧 / Tips and tricks

> 这些是 Obsidian 生态里的可选增强(插件 / 视图),挑有用的开,不用的忽略——和「一切可选且模块化」一致。

- Obsidian 的图表视图是了解你的维基结构的最佳方式——可以清楚地看到各个页面之间的关联关系，哪些页面是核心页面，哪些则是孤立存在的页面。
- Marp 是一种基于 Markdown 格式的幻灯片制作工具。Obsidian 软件有专门的插件支持该格式。利用 Marp，可以直接从维基内容中生成演示文稿。
- Dataview 是一款由 Obsidian 开发的插件，它能够对页面的元数据进行查询处理。如果你的 LLM 在维基页面中添加了 YAML 格式的元数据（如标签、日期、来源数量等信息），Dataview 就能据此生成动态的表格和列表。
- **Excalidraw(附加)** — Obsidian 插件,手绘风格的**白板 / 示意图**工具。适合勾勒架构草图、概念关系、流程示意。
