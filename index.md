---
type: moc
title: Index
updated: 2026-06-30
---

# Index — 全部页面目录 🗂️

> **内容导向的全量目录**:维基里**每一页**都在此列出(link + 一句话摘要 + 来源元数据),按 `type` 分类,**每次 ingest 更新**。
> 查询时**先读本页**定位相关页,再钻进去——规模适中(~100 信源 / 数百页)足够,无需 embedding RAG。

## 综合 / Syntheses

- [[ComfyUI-文本分割方案对比]] — SAM3 / HumanPartsUltra / GroundingDINO+SAM2 三方案选型对比（原创综合）
- [[角色LoRA数据集构建]] — 角色 LoRA 数据集:景别配比(收集张数 vs repeat 权重)、数量、背景范式、单图扩充、打标铁律 + 矛盾澄清（综合 3 篇,归档 `sources/20260619/`）
- [[扩散模型条件注入机制综述]] — U-Net→DiT、IP-Adapter decoupled cross-attention→in-context token、LoRA、注意力注入统一视角（综合,归档 `sources/20260619/`）

## 概念 / Concepts

- [[Windows-Python-中文编码]] — Windows 跑输出中文的 Python CLI 时用 `PYTHONUTF8=1 PYTHONIOENCODING=utf-8` 修乱码 / `UnicodeEncodeError`(调用方修复，无需改源码）
- [[Flux-LoRA-ai-toolkit训练实战]] — ai-toolkit 训 Flux.1-dev 角色 LoRA:24GB 基线配置、lr/alpha/步数/正则图来源分歧、触发词、防过拟合（归档 `sources/20260619/`）
- [[ComfyUI-角色多镜头一致性工作流]] — 全身设定表优先→裁剪→FaceDetailer 精修→衍生镜头;扩图 vs 裁剪方向口诀;一致性三重锁（归档 `sources/20260619/`）
- [[Flux提示词-景别词排序规范]] — 景别词紧贴主体放前段、不可堆叠;Flux 自然语言 vs SD tag 按底模分流（归档 `sources/20260619/`）
- [[AI视觉生成基础手册]] — 文生图→图生视频全维度:容器/质感/引擎/时间层 + 2026 视频模型对比;时间一致性最难（归档 `sources/20260619/`）

## 实体 / Entities

- [[SAM3-文本分割]] — SAM3 文本分割 + ComfyUI 原生集成、节点链、ungated 权重（归档 `sources/comfyui-sam3-docs.md`）

## 参考 / References

- [[本机开发环境]] — wudi 本机硬件与运行时快照（Ryzen 7 9700X · 64GB DDR5-6400 · RTX 5090 D v2 24GB · Python 3.14.5 · JDK 24 · Node v24）（归档 `sources/local-machine-env.md`）

## 信源 / Sources

- [[Karpathy-LLM-Wiki-模式]] — 本库 schema 所本的 LLM Wiki 模式(RAG vs 编译型知识库;Memex 谱系)（归档 `sources/karpathy-llm-wiki.md`）

## 导航 / MOC

- [[ComfyUI-分割与精修]] — "分割出 mask → 局部精修"主题枢纽(挂修脸/手/头发、底模识别)

## 其它入口 / Other entry points

- 时间线日志 → [[log]]
- 治理规则 / schema → [[CLAUDE]]
