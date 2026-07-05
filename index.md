---
type: overview
title: Index
updated: 2026-07-02
---

# Index — 全部页面目录 🗂️

> **内容导向的全量目录**:维基里**每一页**都在此列出(link + 一句话摘要 + 来源元数据),按 `type` 分类,**每次 ingest 更新**。
> 页面按 type 存于 `wiki/<type>/`(folder 名 == frontmatter type);查询时**先读本页**定位相关页,再钻进去——规模适中(~100 信源 / 数百页)足够,无需 embedding RAG。

## 摘要 / Summaries

- [[Karpathy-LLM-Wiki-模式]] — 本库 schema 所本的 LLM Wiki 模式(RAG vs 编译型知识库;Memex 谱系)（归档 `sources/karpathy-llm-wiki.md`）

## 实体 / Entities

- [[SAM3-文本分割]] — SAM3 文本分割 + ComfyUI 原生集成、节点链、ungated 权重（归档 `sources/comfyui-sam3-docs.md`）
- [[本机开发环境]] — wudi 本机硬件与运行时快照（Ryzen 7 9700X · 64GB DDR5-6400 · RTX 5090 D v2 24GB · Python 3.14.5 · JDK 24 · Node v24 · Go 1.26.4 · git 2.54 / gh 2.95 / docker 29.5 · Claude Code Pro 订阅（无 API key）；无 C/Rust 工具链 → 仅纯 Go 可构建）（归档 `sources/local-machine-env.md`，git/gh/docker 段 2026-07-01 补测、Claude Code 订阅段 2026-07-02 用户告知）
- [[F2-批量改名工具]] — Go 写的跨平台批量重命名 CLI（ayoisaiah/f2 v2.2.2）：默认 dry-run、`-x` 落盘、`{%03d}` 自增序号、`--sort natural`、`-u` 撤销;含整目录顺序编号配方（归档 `sources/f2-help-v2.2.2.md`）
- [[JoyCaption]] — 为训扩散模型而生的免费/开放/无审查打标 VLM（Beta One,bf16 ~17GB,24GB 可跑）;三种本地运行方式,vLLM 为本地自托管**无需 API key**;Descriptive/Booru 等多模式 + 可指令化打标（归档 `sources/joycaption-readme.md`）

## 概念 / Concepts

- [[Detailer-采样参数调优]] — Impact Pack Detailer (SEGS) / FaceDetailer / DetailerForEach 共用采样参数调参优先级:denoise 命门、cfg=1.0 认底模、guide_size 决定脸部清晰度（原创综合,归档 `raw/assets/detailer-segs-node-panel.png`）
- [[Windows-Python-中文编码]] — Windows 跑输出中文的 Python CLI 时用 `PYTHONUTF8=1 PYTHONIOENCODING=utf-8` 修乱码 / `UnicodeEncodeError`(调用方修复，无需改源码）
- [[Flux-LoRA-ai-toolkit训练实战]] — ai-toolkit 训 Flux.1-dev 角色 LoRA:24GB 基线配置、lr/alpha/步数/正则图来源分歧、触发词、防过拟合（归档 `sources/20260619/`）
- [[ai-toolkit-WebUI默认值陷阱]] — ai-toolkit Web UI 新建 job 默认值 ≠ 基线(rank 32 / steps 3000 / 样图占位词 / guidance 4 / cache·ema 关)的开跑前检查清单 + keep×steps 删档陷阱 + 改 job 的 API 备忘（原创综合,首次实证 m1r41 首训 2026-07-04）
- [[ComfyUI-角色多镜头一致性工作流]] — 全身设定表优先→裁剪→FaceDetailer 精修→衍生镜头;扩图 vs 裁剪方向口诀;一致性三重锁（归档 `sources/20260619/`）
- [[Flux提示词-景别词排序规范]] — 景别词紧贴主体放前段、不可堆叠;Flux 自然语言 vs SD tag 按底模分流（归档 `sources/20260619/`）
- [[AI视觉生成基础手册]] — 文生图→图生视频全维度:容器/质感/引擎/时间层 + 2026 视频模型对比;时间一致性最难（归档 `sources/20260619/`）
- [[LoRA打标]] — LoRA 打标通用入门:黄金法则(标可变/不标想绑定)、WD14 vs BLIP/BLIP2、触发词基础 + 2026 按场景工具选型表(二次元/写实/NSFW/低显存/云端 API,新增 Qwen3-VL、Florence-2、GPT-4o/Claude）+ 打标风格按底模分语言(SD/Flux/z-image）（归档 `sources/claude/LoRA打标的概念与应用.md` + `raw/assets/lora-tagging-tool-selection-2026.png`）
- [[z-image-agent打标工程师]] — 把打标黄金法则固化成硬约束的 LoRA 打标 agent prompt(z-image 底模:绑定/可变二分、中文结构化短语 + 固定字段顺序、触发词领头、结尾固定画质词);带 wiki 基线标注三处分歧(中文/真名触发词/固定画质词）（归档 `sources/agent角色/z-image-lora通用agent打标提示词.md`）
- [[flux-agent打标工程师]] — z-image 版的按底模分流镜像:同一黄金法则,输出英文自然语言 prose、罕见 token 触发词领头、灯光必写、无负面词 / 无恒定画质样板(Flux/T5 底模）（归档 `sources/agent角色/flux通用agent打标提示词.md`）

## 对比 / Comparisons

- [[ComfyUI-文本分割方案对比]] — SAM3 / HumanPartsUltra / GroundingDINO+SAM2 三方案选型对比（原创综合）

## 概览 / Overview

- [[ComfyUI-分割与精修]] — "分割出 mask → 局部精修"主题枢纽(挂修脸/手/头发、底模识别)
- [[agent角色]] — 可复用 agent 角色 prompt 的导航枢纽(把 LLM 固化成工作流专才);源目录 `sources/agent角色/`,当前收录 [[z-image-agent打标工程师]]、[[flux-agent打标工程师]]

## 综合 / Syntheses

- [[ComfyUI-工作流前查清单方案]] — 搭工作流前查清单:可用性看 `/object_info`(get_node_info)、磁盘文件看 `comfy model list`、插件看 cm-cli;以 ① 为权威写节点名（原创综合）
- [[角色LoRA数据集构建]] — 角色 LoRA 数据集:景别配比(收集张数 vs repeat 权重)、数量、背景范式、单图扩充、打标铁律 + 矛盾澄清（综合 3 篇,归档 `sources/20260619/`）
- [[扩散模型条件注入机制综述]] — U-Net→DiT、IP-Adapter decoupled cross-attention→in-context token、LoRA、注意力注入统一视角（综合,归档 `sources/20260619/`）
- [[剧本到AI漫剧生产流水线]] — 剧本→动态漫端到端六阶段:角色圣经/分镜→圣杯图→数据集+打标→LoRA 训练→批量出镜→I2V 装配;串联库内六页的生产顺序视图（原创综合）

## 其它入口 / Other entry points

- 时间线日志 → [[log]]
- 治理规则 / schema → [[CLAUDE]]
