---
type: moc
aliases: ["ComfyUI 分割与精修", "ADetailer 主题", "局部精修主题"]
tags: [comfyui, moc, 分割, 精修]
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

- [[修头发-Detailer工作流]] — 待写:分割出 hair mask → `MaskToSEGS → DetailerForEach` 局部重绘。
- [[FaceDetailer-面部手部精修]] — 待写:face / hand YOLO(bbox)+ SAM 精修,串行先脸后手。

## 相关识别工具

- [[底模识别-base-model-detector]] — 待写:读 safetensors 头识别底模 / 类型 / 精度。

## 待写 / TODO

- 实测 SAM3 vs HumanPartsUltra 的 hair mask 对比,结果回填 [[ComfyUI-文本分割方案对比]]。
- 把"修脸 / 修手 / 修头发"完整工作流沉淀成页,挂到本 MOC。
