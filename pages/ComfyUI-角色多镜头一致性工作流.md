---
type: concept
aliases:
  - 漫剧角色全镜头工作流
  - 角色多镜头一致性
  - ComfyUI consistent character
tags:
  - comfyui
  - lora
  - consistency
  - workflow
status: stable
created: 2026-06-30
updated: 2026-06-30
source:
  - "[[#来源]]"
related:
  - "[[角色LoRA数据集构建]]"
  - "[[Flux-LoRA-ai-toolkit训练实战]]"
  - "[[ComfyUI-分割与精修]]"
  - "[[Flux提示词-景别词排序规范]]"
---

# ComfyUI 角色多镜头一致性工作流

## 摘要 / Summary

漫剧/竖屏短剧要让同一角色在特写、半身、全身、多角度间**脸+服装+配色+画风全方位一致**。核心做法不是"先画脸再扩展",而是**先出一张全身设定表锁住四要素 → 裁剪 → 对脸做专门精修 → 衍生新镜头**。一致性是一套系统而非单一技术。

## 核心结论 / Key points

### 1. 生成顺序:全身设定表优先(地基)

> [!tip] 最优顺序
> **先全身角色设定表(character sheet / turnaround)→ 裁剪出半身/特写 → 对脸 [[FaceDetailer]] 精修**,而非先画脸再外扩。
> 理由:一张图里把"正/侧/背 + 全身比例 + 服装 + 配色 + 画风"一次性锁定,所有视角彼此互相一致;之后裁剪不丢设计信息。

- Mickmumpitz Consistent Character Creator 的分组顺序印证:Multiview → Upscale 1st → Upscale 2nd(精修脸)→ Emotions(Live Portrait)→ Lighting(IC-Light)——**脸是在多视图确定之后才逐块精修**。3.8 版已升级到 [[Qwen-Image-Edit]]-2511,含数据集导出工具。
- ComfyUI 官方模板 "360 Full-body Turnaround"、Flux Kontext Character Turnaround Sheet LoRA(单图出 5 视角)同理。
- **何时反过来(脸先行)**:用指令编辑模型(Qwen-Edit/Nano Banana)向外扩展身份,或为训 [[角色LoRA数据集构建|LoRA 数据集]] bootstrapping 时;但必须配身份保持工具,否则服装/比例会漂移。

> [!note] 澄清 X-1:本页"全身优先" vs 数据集页"特写优先"不冲突
> 本页讲的是**生产时的生成顺序**(一张全身设定表锁全方位一致);[[角色LoRA数据集构建#1. 景别优先级]] 讲的是**训练集的收集/锚定顺序**(先特写锚定身份)。**两个"顺序"是不同活动**,且本页也指出"为 LoRA 数据集 bootstrapping 时可脸先行"——按你当前在做哪件事选。

### 2. 方向判断口诀:扩图 vs 裁剪精修

> [!important] 露出更多身体 → 扩图;放大局部 → 裁剪精修
> - **特写 → 半身/全身(露出更多)= 扩图(outpainting)**。官方文档:"Outpainting is the same thing as inpainting"(扩图就是带遮罩的重绘)。
> - **全身 → 特写(露出更少,放大局部)= 裁剪 + 放大精修**,**不是扩图也不是普通重绘**(没有新区域要填)。

**扩图节点链**:`Pad Image for Outpainting`(补边+遮罩,设 feathering)→ `VAE Encode for Inpainting`(`grow_mask_by` 官方默认 **6**,范围 0–64;扩大区域常调到 64 防接缝)→ KSampler(denoise 0.95–1.0)→ VAE Decode。
- ⚠️ **原生 Flux 不擅长首轮扩图**,用 SDXL checkpoint,分辨率匹配 SDXL 尺寸。

**裁剪精修**:"only masked" 重绘(denoise ~0.5)把蒙版区裁出来整张分辨率重画再缩回 = ComfyUI 的 **[[FaceDetailer]]** 或裁剪→放大→img2img。Upscale 与 FaceDetailer **必须固定 seed** 否则伤一致性。

**现代工具 ComfyUI-Inpaint-CropAndStitch**(lquesada)两个方向通吃:`Inpaint Crop`(围绕蒙版裁+预缩放)/`Inpaint Stitch`(无损贴回)/`Extend Image for Outpainting`。推荐用 `InpaintModelConditioning` 替代 `VAE Encode (for Inpainting)` 以支持 denoise <1;用 `context_expand_factor`(如 2)给模型更多上下文。

### 3. 一致性技术的"三重锁"

主流方案:**[[IP-Adapter]] FaceID(脸)+ 角色 [[LoRA]](身体/服装/风格)+ [[ControlNet]](姿势)**。各手段分工:

| 方案 | 面部 | 服装 | 画风 | 场景 | 备注 |
|---|---|---|---|---|---|
| IPAdapter FaceID/Plus Face | 强 | 弱 | 中 | 快速起步 | 权重 0.7–0.85,过高"烧脸";依赖 InsightFace(商用需授权) |
| InstantID | 很强(写实) | 弱 | 弱 | 写实人物 | 二次元效果一般 |
| PuLID-Flux II | 很强 | 弱 | 中-强 | Flux 工作流 | 解决"模型污染",保画风好;8GB 可跑 |
| ReActor/FaceSwap | 强(事后) | 无 | 无 | 补救换脸 | 生成后贴脸 |
| **角色 LoRA** | 很强 | 很强 | 很强 | **长期高频** | 最可靠长期方案;见 [[Flux-LoRA-ai-toolkit训练实战]] |
| ControlNet | 控结构 | — | — | 锁姿势/构图 | 负责"动作"非"身份" |
| [[Qwen-Image-Edit]]-2511 | 强 | 强 | 强 | 单图多镜头/换装 | 指令编辑;可能改脸需配 FaceDetailer |
| [[Flux-Kontext]] | 强 | 强 | 强 | 指令换镜头/背景 | 适合中近景,**全身大幅扩展不可靠** |

**关键判断**:个人漫剧"全方位一致"单靠 IPAdapter 不够(它只锁脸);最稳是**角色 LoRA**(锁身体/服装/风格)或**指令编辑模型**(原生保服装配色)。二次元角色用专属 LoRA 往往比人脸识别类(InstantID/InsightFace)更稳。

### 4. 竖屏漫剧特殊考量

- **9:16 竖屏**:用 832×1216 / 768×1344 等 SDXL 竖屏尺寸;景别 prompt 词排序见 [[Flux提示词-景别词排序规范]]。
- **可复用角色资产库**:先建 4–6 张含正/侧/背/表情的 "CHAR 资产库" + 结构化文字设定;高频生产则训 LoRA + 固定触发词。
- **二次元底模**:Illustrious 系(WAI-Illustrious、Hassaku XL、Plant Milk)、Pony V6 XL、NoobAI-XL;需强编辑/多镜头用 Qwen-Image-Edit / Flux Kontext。

## 推荐实操路线 / Workflows

> [!example] 方案 1(二次元漫剧,SDXL 路线)
> 1. Illustrious/Pony 底模出全身三视图设定表(`1girl, character sheet, full body, front/side/back view, white background, [发色/发型/眼睛/服装]`),固定 seed。
> 2. FaceDetailer 对每视角脸精修(固定 seed)。
> 3. Inpaint Crop 裁半身/特写 → Upscale + img2img(denoise 0.4–0.5)精修。
> 4. 衍生镜头:ControlNet OpenPose + IPAdapter FaceID,**或** Qwen-Edit 指令("close-up, lock character/clothes")。
> 5. 高频生产 → 训角色 LoRA(见 [[Flux-LoRA-ai-toolkit训练实战]])。

> [!example] 方案 2(Flux/Qwen 指令编辑,最省事)
> Flux.1-dev/Qwen 出满意全身图 → PuLID-Flux II 锁脸 + Kontext/Qwen-Edit 指令换镜头(特写 "zoom in, keep identity";全身用 Zoom-Out LoRA / "zoom out, full body, lock character")→ 每镜 FaceDetailer + SeedVR2/SUPIR 放大。

**插件清单**:ComfyUI-Manager、IPAdapter_plus、PuLID-Flux2、Impact-Pack(FaceDetailer)、Inpaint-CropAndStitch、ControlNet(Union Pro/OpenPose)、AdvancedLivePortrait、Qwen-Image-Edit-2511、Flux Kontext dev、Kohya_ss。

## 何时切换策略 / Thresholds

- 单角色出图 **< 20–30 张** → IPAdapter/PuLID/指令编辑即可,不必训 LoRA。
- 单角色 **> 几十张 / 长期连载** → 训角色 LoRA。
- Kontext 在全身大幅扩展崩坏 → 退回 SDXL + 扩图,或从设定表重新裁切。

## 关联 / Connections

- 把生成的多角度图沉淀成训练集 → [[角色LoRA数据集构建]]
- "分割出 mask → 局部精修"主题枢纽 → [[ComfyUI-分割与精修]]
- 景别词怎么写 → [[Flux提示词-景别词排序规范]]
- 待写:[[Flux-Kontext]] · [[Qwen-Image-Edit]] · [[IP-Adapter]] · [[ControlNet]] · [[FaceDetailer]] · [[PuLID]]

## 来源 / Sources

`sources/20260619/ComfyUI 漫剧角色全镜头生成工作流：生成顺序与全方位一致性技术方案.md`

## 开放问题 / Open questions

- "全身优先"主要基于主流工作流作者(Mickmumpitz)与中文社区实践,**非决定性官方基准**;face-first 在指令编辑模型加持下也可行,应按手头模型灵活选。
- denoise / grow_mask_by / LoRA 步数等多为社区经验值,需按底模、显存、题材实测。
