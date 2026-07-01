---
type: log
title: Log
updated: 2026-07-01
---

# Log — 吸收 / 查询时间线

> **只追加(append-only)**,新条目加到**末尾**(最新在最下)。每条用统一前缀
> `## [日期] 动作 | 涉及 — 备注`,便于 `grep "^## \[" log.md | tail -5` 取最近若干条。

## [2026-06-29] init | 知识库初始化(LLM Wiki schema)

## [2026-06-30] ingest | SAM3 文本分割 — 归档 sources/comfyui-sam3-docs.md;新建 [[SAM3-文本分割]]、综合 [[ComfyUI-文本分割方案对比]]、MOC [[ComfyUI-分割与精修]];对比页内含 GroundingDINO+SAM2 在 transformers 5.x 的脆性反例(折叠)

## [2026-06-30] ingest | 本机开发环境 — 归档 sources/local-machine-env.md;新建 [[本机开发环境]](type:reference);入 index 新分类 References;[[Windows-Python-中文编码]] 补反链消孤儿

## [2026-06-30] ingest | AI 图像/视频 8 篇(sources/20260619/) — 新建 6 页:[[角色LoRA数据集构建]]、[[Flux-LoRA-ai-toolkit训练实战]]、[[ComfyUI-角色多镜头一致性工作流]]、[[Flux提示词-景别词排序规范]]、[[AI视觉生成基础手册]]、[[扩散模型条件注入机制综述]];矛盾逐一确认后落 callout(配比张数 vs repeat 权重、lr 1e-3、alpha=rank vs /2、步数、正则图、触发词首位 vs 孤立、全身优先 vs 特写优先、tag/自然语言按底模分流、CFG 5-9 vs Flux guidance);[[ComfyUI-分割与精修]] MOC 补"角色生成"段接入

## [2026-06-30] meta | 流程优化 — 把"逐一确认矛盾点→逐一开干"固化进 wiki-ingest 技能(步骤 2/3)与 CLAUDE.md(Ingest 步骤 2.5);存 feedback 记忆 ingest-confirm-conflicts-per-subtopic(起因:本次漏查 B/C/D 子主题被用户点名)

## [2026-06-30] ingest | ComfyUI 搭工作流前查清单方案 — 原创综合(无外部信源):新建 [[ComfyUI-工作流前查清单方案]](synthesis),三类清单「谁最全」对比 + 搭工作流以 /object_info(get_node_info)为权威的固定流程;[[ComfyUI-分割与精修]] MOC 补反链;入 index 综合区。起因:用户要求该规则录 wiki 而非记忆文件


## [2026-06-30] ingest | Detailer-采样参数调优 — 新建 Detailer(SEGS)/FaceDetailer/DetailerForEach 共用采样参数调参页(denoise 优先/cfg=1.0 认底模/guide_size);挂入 ComfyUI-分割与精修 MOC,与角色多镜头一致性页互链,denoise 0.5 vs 0.3-0.45 以语境 callout 调和;归档 raw/assets/detailer-segs-node-panel.png
## [2026-06-30] ingest | 本机 Go 环境 — 探测 go version/env,扩档 sources/local-machine-env.md 加 Go 小节;[[本机开发环境]] 加 Go 表(go1.26.4/GOROOT/GOPATH/工具链)+ 两条 callout(CGO_ENABLED=0 且无 gcc→只能纯 Go 构建;GOPROXY 国际代理非 goproxy.cn)+ frontmatter tags 加 go + 开放问题加 cgo 工具链;index 摘要补 Go 1.26.4

## [2026-06-30] ingest | F2 批量改名工具 — 探测本机 f2.exe 版本(go version -m → ayoisaiah/f2/v2 v2.2.2,纯 Go),归档 sources/f2-help-v2.2.2.md(verbatim --help 快照);新建 [[F2-批量改名工具]](type:reference):心智模型(默认 dry-run/-x 落盘/-u 撤销)、核心标志表、替换变量、整目录顺序编号实战配方({%03d}+--sort natural,本机已验证);[[本机开发环境]] Go 工具行补 [[F2-批量改名工具]] wikilink + related + Connections(印证纯 Go 工具在 CGO 关闭机上 go install 即用);入 index References 区。单源无矛盾。起因:用户用 f2 改 ComfyUI 输出图后要求录 wiki

## [2026-07-01] ingest | 本机环境刷新(keep-current) — 重探本机;硬件+5 大运行时逐项与 2026-06-30 快照一致(无真矛盾,仅磁盘用量小幅陈旧已刷新)。用户拍板「不建新源、并成一节」:[[本机开发环境]] 新增「开发工具 / Dev tooling」段(git 2.54.0 / gh 2.95.0 / docker 29.5.3)+ 缺席工具链 note(rustc/cargo/gcc/clang/cmake 全缺→强化「仅纯 Go 可构建」);Python 段补 uv 托管解释器事实(仅 3.14.5,余为 download available;venv 为 per-project 无全局清单)答旧开放问题;CGO warning 扩 clang/cmake;frontmatter tags +git/docker、updated→07-01。sources/local-machine-env.md 保持 06-30 不动(git/gh/docker 未存档,页内注明补测来源)。index 摘要 + updated 同步

## [2026-07-01] refactor | pages→wiki 按 type 分文件夹 + type 枚举对齐原文 6 类 — git mv 15 页入 `wiki/{summary,entity,concept,comparison,overview,synthesis}/`(folder 名 == frontmatter type);type 迁移 source→summary·moc→overview·reference→entity(本机开发环境/F2)·synthesis→comparison(仅文本分割方案对比);ComfyUI-分割与精修 tag moc→overview。CLAUDE.md/templates 双语枚举注释(summary 摘要|entity 实体|concept 概念|comparison 对比|overview 概览|synthesis 综合)+ 目录树/路径 pages→wiki;三技能与 sources/README 路径 pages→wiki(写页按 type 落文件夹)。index 重排为 6 分类(+对比/Comparisons,信源/Sources→摘要/Summaries,导航/MOC→概览/Overview,删参考/References 并入实体/Entities)、frontmatter type moc→overview。起因:逐字比对 sources/karpathy-llm-wiki.md,对齐原文「The wiki」层命名与六类页
