---
type: concept
aliases:
  - Detailer SEGS 调参
  - FaceDetailer 采样参数
  - Impact Pack Detailer 参数
  - DetailerForEach 调参
tags:
  - comfyui
  - impact-pack
  - detailer
  - 精修
  - 采样
status: draft
created: 2026-06-30
updated: 2026-06-30
source: "[[#来源 / Sources]]"
related:
  - "[[ComfyUI-分割与精修]]"
  - "[[ComfyUI-角色多镜头一致性工作流]]"
  - "[[底模识别-base-model-detector]]"
---

# Detailer 采样参数调优

## 摘要 / Summary

Impact Pack 的 **Detailer (SEGS)** / **[[FaceDetailer]]** / **DetailerForEach** 三个节点**共用同一组采样参数**(`guide_size / steps / cfg / sampler_name / scheduler / denoise / feather …`),都是"检测出区域 → 局部重绘精修"。本页讲这组参数的**调参优先级**:不要平均用力,`denoise` 才是命门,`cfg=1.0` 是认底模的信号。是 [[ComfyUI-分割与精修]] 主线里"精修:用 mask 重绘"的参数底座。

## 调参优先级 / Priority

> [!important] 一句话顺序
> **`guide_size` →（扫）`denoise` →（成对选）`sampler + scheduler` →（微调）`steps`**;`cfg` **只在确认底模是普通模型时才碰**。

### 1. denoise —— 唯一最关键参数

局部重绘里它决定"**改多少**"。**有效步数 ≈ `steps × denoise`**(例:0.4 × 40 ≈ 等效 16 步)。

| denoise | 效果 | 适用 |
|---|---|---|
| `0.25–0.35` | 只清理 / 加细节,**保持原脸 / 原结构** | 一致性优先 |
| `0.4–0.5` | 明显增细节,可能轻微改五官 | 通用基线 |
| `>0.55` | 基本重画,易接缝 / 换脸 / 丢身份 | 慎用 |

> [!note] 澄清:本页 0.3–0.45 vs [[ComfyUI-角色多镜头一致性工作流]] 的 ~0.5
> **非真矛盾,语境不同**——那页讲"**裁剪整张 only-masked 重画**"(denoise ~0.5,因为整块要重绘);本页讲"**SEGS 局部精修保身份**"(0.3–0.45 更稳)。两者同在 0.3–0.5 带内。
> **推荐基线 ≈ 0.4**(与节点默认一致):先固定其余参数,**只扫 `denoise` 0.3 / 0.4 / 0.5 看差异**,这是性价比最高的调法。

### 2. sampler_name + scheduler —— 成对调,别拆开

| 组合 | 特点 |
|---|---|
| `euler + simple` / `euler + beta` | 最稳、可预测,**默认首选** |
| `dpmpp_2m + karras` | 细节 / 锐度更好,老牌脸部细化常用 |
| `dpmpp_2m_sde + karras` | 更多纹理,但 `denoise` 别太高否则噪 |
| `dpmpp_3m_sde + exponential` | 高步数下质感好 |

脸部想更锐 → `dpmpp_2m + karras`;求稳 → 留 `euler`。

### 3. cfg —— `1.0` 是认底模的信号

> [!warning] cfg=1.0 意味着负面提示词**完全不起作用**
> 这是 **Flux / Turbo / Lightning / 蒸馏模型** 的用法。
> - 底模是 Flux / 蒸馏 → `1.0` 正确,**别动**,调高反而崩。
> - 底模是普通 SDXL / SD1.5 → `1.0` 太低,应回到 **`4–7`**,否则负面词与引导力全失效。
>
> **先确认底模类型再决定动不动 cfg**(怎么认 → [[底模识别-base-model-detector]])。

### 4. steps

`40` 偏高。配 0.4 denoise(等效 ~16 步),`20–30` 步通常无肉眼差异且更快;蒸馏模型甚至 `8–12` 步。

### 5. guide_size / feather / 其它

- **`guide_size`**(图中 512):检测框会**先放大到此尺寸再采样**,**对脸部清晰度影响极大**。脸小时提到 `768 / 1024` 提升明显(代价:显存 / 时间)。`max_size`(1024)是放大上限,`guide_size_for=bbox` 指按检测框而非裁剪框算。
- **`feather`**(图中 5):接缝羽化,换脸 / 接缝明显时加到 `10–20`;另有 `noise_mask_feather`(20)控噪声遮罩边缘。
- **`noise_mask` / `force_inpaint`**(enabled):保持开启,保证只在 mask 区域重绘且小区域也强制处理。
- **`cycle`**(1):多轮迭代精修,>1 会反复重采样(配低 denoise 用),通常留 1。
- **`seed` / `control_after_generate=fixed`**:与 [[ComfyUI-角色多镜头一致性工作流]] 一致——**精修必须固定 seed**,否则伤角色一致性。

## 关联 / Connections

- 上层主线(出 mask → 精修枢纽)→ [[ComfyUI-分割与精修]]
- 精修在工作流里的位置 + 固定 seed 要求 → [[ComfyUI-角色多镜头一致性工作流]]
- cfg 该不该调取决于底模 → [[底模识别-base-model-detector]]
- 待写:[[FaceDetailer]](节点总览:bbox+SAM、先脸后手串行)

## 来源 / Sources

原创综合(基于节点面板截图 + Impact Pack 通用知识)。归档:`raw/assets/detailer-segs-node-panel.png`(用户的 Detailer (SEGS) 面板,记录所讨论的默认值:steps 40 / cfg 1.0 / denoise 0.40 / euler+simple / guide_size 512 / feather 5)。

## 开放问题 / Open questions

- 表中 sampler/scheduler 偏好与 denoise 区间多为社区经验值,需按**底模(SDXL vs Flux vs 蒸馏)、题材、显存**实测。
- 截图 cfg=1.0 + 40 步的组合暗示底模可能是 Flux / 蒸馏,但 40 步对蒸馏偏高——**底模未确认前无法判定 steps 是否浪费**,确认后回填本页。
