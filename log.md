---
type: log
title: Log
updated: 2026-06-30
---

# Log — 吸收 / 查询时间线

> **只追加(append-only)**,新条目加到**末尾**(最新在最下)。每条用统一前缀
> `## [日期] 动作 | 涉及 — 备注`,便于 `grep "^## \[" log.md | tail -5` 取最近若干条。

## [2026-06-29] init | 知识库初始化(LLM Wiki schema)

## [2026-06-30] ingest | SAM3 文本分割 — 归档 sources/comfyui-sam3-docs.md;新建 [[SAM3-文本分割]]、综合 [[ComfyUI-文本分割方案对比]]、MOC [[ComfyUI-分割与精修]];对比页内含 GroundingDINO+SAM2 在 transformers 5.x 的脆性反例(折叠)

## [2026-06-30] lint | index + 全 5 页 — 修:补 [[Windows-Python-中文编码]] 入 index(原缺失,且填了空的 Concepts 占位);待用户定夺:该页孤儿(指向不存在的 boss-* 簇)、若干待写悬空链([[Memex]]/[[GroundingDINO]]/三个 Detailer/识别页)、SAM3 vs HumanPartsUltra 实测数据缺口

## [2026-06-30] ingest | 本机开发环境 — 归档 sources/local-machine-env.md;新建 [[本机开发环境]](type:reference);入 index 新分类 References;[[Windows-Python-中文编码]] 补反链消孤儿

## [2026-06-30] ingest | AI 图像/视频 8 篇(sources/20260619/) — 新建 6 页:[[角色LoRA数据集构建]]、[[Flux-LoRA-ai-toolkit训练实战]]、[[ComfyUI-角色多镜头一致性工作流]]、[[Flux提示词-景别词排序规范]]、[[AI视觉生成基础手册]]、[[扩散模型条件注入机制综述]];矛盾逐一确认后落 callout(配比张数 vs repeat 权重、lr 1e-3、alpha=rank vs /2、步数、正则图、触发词首位 vs 孤立、全身优先 vs 特写优先、tag/自然语言按底模分流、CFG 5-9 vs Flux guidance);[[ComfyUI-分割与精修]] MOC 补"角色生成"段接入

## [2026-06-30] meta | 流程优化 — 把"逐一确认矛盾点→逐一开干"固化进 wiki-ingest 技能(步骤 2/3)与 CLAUDE.md(Ingest 步骤 2.5);存 feedback 记忆 ingest-confirm-conflicts-per-subtopic(起因:本次漏查 B/C/D 子主题被用户点名)

