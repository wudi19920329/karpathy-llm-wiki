---
type: synthesis
aliases: ["底图分割方案对比", "hair mask 方案对比", "ComfyUI 文本分割选型"]
tags: [comfyui, segmentation, 对比, 分割, 精修]
status: draft
created: 2026-06-30
updated: 2026-06-30
source:
related: ["[[SAM3-文本分割]]", "[[ComfyUI-分割与精修]]"]
---

# ComfyUI 文本分割方案对比

## 摘要 / Summary

在 ComfyUI 里给"头发"等目标做局部 mask(再喂给 Impact Pack `MaskToSEGS → DetailerForEach` 精修)有三条路。**结论:优先 [[SAM3-文本分割|SAM3]] 原生**(免维护、文本灵活);只要立刻能用且不想下模型,用已装的 **HumanPartsUltra**;**GroundingDINO + SAM2** 在 transformers 5.x 上脆,不再推荐。

## 要点 / Key points

| 维度 | [[SAM3-文本分割\|SAM3 原生]] ⭐ | HumanPartsUltra | GroundingDINO + SAM2 |
|---|---|---|---|
| 来源 | ComfyUI 核心(PR #13408) | ComfyUI_LayerStyle_Advance(已装) | comfyui-sam2(已装) |
| 装东西 | 下 1 权重(**非 gated**) | 无 | 无 |
| transformers 依赖 | **否**(核心内置) | 否(ONNX/解析模型) | **是**(2023 老代码,脆) |
| 文本提示 | ✅ 任意词(≤32 token) | ❌ 固定部位(`hair` 开关) | ✅ 任意词 |
| mask 质量 | 高(SAM 解码精修) | 头发整体好 + VITMatte 边缘 | 高(SAM 精修) |
| 更新后稳定 | 跟核心,稳 | 稳 | 可能再坏 |
| 额外能力 | 视频追踪 / 多目标 | 多种人体部位 | — |
| 适用 | 通用首选 | 人像 / 动漫人体部位 | 历史遗留 |

### 反例:为什么 GroundingDINO + SAM2 脆

`comfyui-sam2` 内置 2023 版 GroundingDINO,在 **transformers 5.8.1** 上连环报错(均因 transformers 5.x 大重构):

- `'BertModel' object has no attribute 'get_head_mask'` — 该方法被移除。补丁:`bertwarper.py:25` 用 `getattr(bert_model, "get_head_mask", lambda head_mask, n, is_attention_chunked=False: [None]*n)` 回退(GroundingDINO 永远传 head_mask=None,等价)。
- `get_extended_attention_mask(..., device)` 签名变了(第 3 位现为 `dtype`)→ `to(dtype=torch.device)` TypeError。补丁:`bertwarper.py:112` 去掉 `device` 参数(mask 已在正确 device,dtype=None 自动取 `self.dtype`)。

两处都是改插件 **vendored 文件**,**插件更新即被覆盖**——这就是它"脆"的本质,也是转向 SAM3 的根本原因。

### 备选:HumanPartsUltra(零摩擦)

`LayerMask: HumanPartsUltra`(LayerStyle_Advance,已装):勾 `hair`(其余关)→ 直接出 MASK,内置 VITMatte 边缘细化。无文本模型、无 transformers 依赖、无 gated 下载;缺点是只能选预定义部位,动漫准度取决于解析模型。适合"立刻要、不折腾"。

## 关联 / Connections

- 推荐方案详解:[[SAM3-文本分割]]
- 主题枢纽:[[ComfyUI-分割与精修]]
- 下游精修:[[修头发-Detailer工作流]](待写)
- 来源:[[comfyui-sam3-docs]]、本次会话排障(原创综合)

## 开放问题 / Open questions

- 同一张动漫图,SAM3 vs HumanPartsUltra 的 hair mask 实测对比(边缘 / 碎发 / 漏检)——未做,值得回填。
- SAM3 首次实测的显存 / 速度 vs SAM2,缺数据。
