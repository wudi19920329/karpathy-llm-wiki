---
type: overview
aliases: ["ComfyUI 分割与精修", "ADetailer 主题", "局部精修主题"]
tags: [comfyui, overview, 分割, 精修]
status: draft
created: 2026-06-30
updated: 2026-06-30
source:
related: ["[[SAM3-文本分割]]", "[[ComfyUI-文本分割方案对比]]"]
---

# ComfyUI 分割与精修 🗺️

## 摘要 / Summary

ComfyUI 里"**检测 / 分割某区域 → 局部重绘精修**"这条主线的枢纽页(MOC)。把出 mask 的各种手段、Impact Pack 的 SEGS 精修、各类 Detailer 挂在这里,方便以后往里长。

## 分割:出 mask

- [[SAM3-文本分割]] — 核心原生、文本提示,**首选**。
- [[ComfyUI-文本分割方案对比]] — SAM3 / HumanPartsUltra / GroundingDINO+SAM2 选型对比。

## 精修:用 mask 重绘

- [[Detailer-采样参数调优]] — Detailer (SEGS) / FaceDetailer / DetailerForEach **共用采样参数**的调参优先级:`denoise` 命门、`cfg=1.0` 认底模、`guide_size` 决定脸部清晰度。
- [[修头发-Detailer工作流]] — 待写:分割出 hair mask → `MaskToSEGS → DetailerForEach` 局部重绘。
- [[FaceDetailer-面部手部精修]] — 待写:face / hand YOLO(bbox)+ SAM 精修,串行先脸后手。

## 应用场景:角色生成 / 训练

- [[ComfyUI-角色多镜头一致性工作流]] — "全身设定表 → 裁剪 → FaceDetailer 精修脸 → 衍生镜头",精修是其中一环;含扩图 vs 裁剪的方向口诀。
- [[角色LoRA数据集构建]] — 出图阶段用 FaceDetailer/ADetailer 修脸、inpainting 修手(而非数据集层死磕)。
- [[Flux-LoRA-ai-toolkit训练实战]] — 把多角度图沉淀成 LoRA 的训练参数与调试。

## 相关识别工具

- [[底模识别-base-model-detector]] — 待写:读 safetensors 头识别底模 / 类型 / 精度。
- [[ComfyUI-工作流前查清单方案]] — 搭工作流前先查清单:以 `/object_info`(`get_node_info`)为权威拿节点 / 模型枚举,辅以 `list_local_models` / `list_installed_nodes`。

## 待写 / TODO

- 实测 SAM3 vs HumanPartsUltra 的 hair mask 对比,结果回填 [[ComfyUI-文本分割方案对比]]。
- 把"修脸 / 修手 / 修头发"完整工作流沉淀成页,挂到本 MOC。
