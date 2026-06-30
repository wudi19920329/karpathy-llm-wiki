---
type: entity
aliases: [SAM3, "Segment Anything 3", "SAM 3.1 Multiplex", SAM3_Detect]
tags: [comfyui, segmentation, sam, 分割]
status: draft
created: 2026-06-30
updated: 2026-06-30
source: "[[comfyui-sam3-docs]]"
related: ["[[ComfyUI-文本分割方案对比]]", "[[ComfyUI-分割与精修]]"]
---

# SAM3 文本分割

## 摘要 / Summary

SAM3(Segment Anything 3)是 Meta 的统一**可提示分割**基础模型,能用**文本**或视觉提示(点 / 框 / 掩码)在图像与视频中检测、分割、追踪目标。相比 SAM2,新增"用一句话穷举分割某开放词汇概念的所有实例"的能力。**ComfyUI 已原生集成**(PR #13408),无需任何自定义插件——这使它成为 ComfyUI 里给"头发"等区域出 mask 时**最干净、免维护**的方案。详见选型:[[ComfyUI-文本分割方案对比]]。

## 要点 / Key points

- **原生节点**:核心 `comfy_extras/nodes_sam3.py` 提供 `SAM3_Detect`(图像)、`SAM3_VideoTrack` / `SAM3_TrackToMask`(视频)。`SAM3_Detect` 输入 image + conditioning(文本)→ 输出 **MASK + BOUNDING_BOX**。
- **权重(非 gated)**:`sam3.1_multiplex_fp16.safetensors`,来自 HF `Comfy-Org/sam3.1`,放 `ComfyUI/models/checkpoints/`。Meta 原版在 HF 是 gated(要申请),**Comfy-Org 重打包版可直接下**。
- **节点链**:
  - `Load Checkpoint(sam3.1) → MODEL + CLIP`
  - `CLIPTextEncode(CLIP, "hair") → CONDITIONING`
  - `SAM3_Detect(model, image, conditioning, threshold=0.5, refine_iterations=2) → MASK`
  - 下游接 Impact Pack:`MaskToSEGS → DetailerForEach`(局部重绘精修)。
- **提示词限制**:文本 **≤ 32 token**,越短越具体越好(如 `hair`、`cat, dog`);逗号分隔可多目标。
- **能力**:文本驱动、图像 + 视频、多目标、开放词汇。
- **为什么用它**:取代 [[GroundingDINO]] + SAM2 那套——后者在 transformers 5.x 上反复崩、需改插件内部文件;SAM3 跟随 ComfyUI 核心更新,**免打补丁**。

> [!note] 跟随核心
> SAM3 是 ComfyUI **核心**功能,不是 vendored 插件代码,因此随 ComfyUI 升级一起维护,不会像旧 GroundingDINO 那样被 transformers 升级打穿。

## 关联 / Connections

- 选型对比:[[ComfyUI-文本分割方案对比]]
- 主题枢纽:[[ComfyUI-分割与精修]]
- 来源:[[comfyui-sam3-docs]] · [官方教程](https://docs.comfy.org/tutorials/utility/video-segment-sam3) · [🤗 Comfy-Org/sam3.1](https://huggingface.co/Comfy-Org/sam3.1)

## 开放问题 / Open questions

- `SAM3_Detect.model` 是 `MODEL` 类型——需核对官方模板:是标准 `CheckpointLoaderSimple` 同时输出 SAM3 的 CLIP,还是有专用加载器?
- 动漫 / 插画上 SAM3 对 "hair" 的分割质量 vs `HumanPartsUltra`?尚未实测(见对比页待办)。
