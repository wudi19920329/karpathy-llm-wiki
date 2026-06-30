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
2. `pages/` — **wiki 本体**,你生成与维护的 markdown(摘要页、实体页、综合页),靠 `[[wikilink]]` 互联。
3. `CLAUDE.md`(本文件)+ `index.md` + `log.md` — **治理层**。

```
CLAUDE.md          # 本 schema(治理大脑)
index.md           # 全量页面目录(catalog,每次 ingest 更新)
log.md             # 只追加的 ingest/query 日志
pages/             # 所有 wiki 页面
sources/           # 原始信源(只读)
templates/page.md  # 新页面模板
raw/assets/        # 附件目录
```

## 页面约定 / Page conventions

`pages/` 下每个页面必须带 frontmatter(**字段名用英文**,便于检索与工具处理;正文用中文):

```yaml
---
type: concept        # concept | entity | source | synthesis | moc
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
- **互联**:正文凡提到已有 / 应有的实体或概念,一律用 `[[wikilink]]`。链向尚不存在的页面是**允许的**——它标记「待写」。
- **小而多**:一页一个概念 / 实体,宁可多页互联,不要长文堆叠。
- 用到 Obsidian 语法(callout / 嵌入 / 属性)时遵循 **obsidian-markdown** 技能。

## 三个操作 / Operations

### 1. Ingest(吸收新信源)

> 信源通常是文章 / 论文 / 网页/数据文件等。

1. 网页:用 **defuddle** 技能抽取正文(省 token),存档进 `sources/`。
2. 通读后**先和用户口头过一遍要点**(人来指方向),再动笔。
2.5. **逐一确认矛盾点(强制)**:多篇 / 重叠信源,**按子主题逐一**扫"跨信源"与"与已有页面"两类冲突,区分真矛盾与表面冲突(量纲 / 语境 / 底模不同),按严重度排序逐一讲给用户;**每个子主题都要单独过,不能只查第一个就开写**。处理方式(推荐基线+并列 / 中立罗列 / 只写基线)与页面粒度**先由用户拍板**,再**逐页开干**,把拍板结论用 callout 落到对应小节,避免多页自相矛盾。
3. 在 `pages/` 写**摘要页**(套 `templates/page.md`),`source:` 指向该信源。
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
- 图片:LLM 难一次读完带内联图的 md;必要时把图单独成页或在 source 页里文字描述。

## 用哪个技能 / Skill cheat-sheet

| 场景                          | 技能                           |
| --------------------------- | ---------------------------- |
| 抓网页正文                       | `obsidian:defuddle`          |
| 写 / 改页面、wikilink、callout、属性 | `obsidian:obsidian-markdown` |
| 关系图 / 概念地图                  | `obsidian:json-canvas`       |
| 批量扫描 / 改属性 / 查反链 / 自动 lint  | `obsidian:obsidian-cli`      |

## 技巧 / Tips and tricks

> 这些是 Obsidian 生态里的可选增强(插件 / 视图),挑有用的开,不用的忽略——和「一切可选且模块化」一致。

- **Obsidian 的 graph view(关系图视图)** — 最直观地看清 wiki 的**形状**:谁连着谁、哪些页面是**枢纽(hub)**、哪些是**孤儿页(orphan)**。它是 Lint 找孤儿页、补缺失交叉引用的好帮手,也是「两个入口」之外的一种**可视化发现**方式。
- **Marp** — 基于 markdown 的幻灯片格式,Obsidian 有插件支持。可直接从 wiki 内容生成演示文稿——对应 Query 第 3 步「答案形态」里的**幻灯片(Marp)**产出。
- **Dataview** — Obsidian 插件,对页面 **frontmatter** 跑查询。我们的页面都带 YAML frontmatter(`type` / `status` / `tags` / `created` / `updated` 等),Dataview 能据此生成**动态表格与列表**(本库走 Karpathy 原版思路:`index.md` 全量目录即导航,Dataview 仅是想要动态视图时的可选项,别拿来替代 `log.md` 审计日志)。
- **Excalidraw(附加)** — Obsidian 插件,手绘风格的**白板 / 示意图**工具。适合勾勒架构草图、概念关系、流程示意——比 JSON Canvas 更自由随性的视觉表达。它存的 `.excalidraw.md` 仍是 markdown,照样进 git、白拿版本史。
