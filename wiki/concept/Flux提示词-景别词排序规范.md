---
type: concept
aliases:
  - Flux景别词排序
  - 构图词排序规范
  - Flux framing prompt order
tags:
  - flux
  - prompt
  - composition
status: stable
created: 2026-06-30
updated: 2026-06-30
source:
  - "[[#来源]]"
related:
  - "[[ComfyUI-角色多镜头一致性工作流]]"
  - "[[AI视觉生成基础手册]]"
---

# Flux 提示词:景别词排序规范

## 摘要 / Summary

在 [[Flux-Kontext|Flux]] 文生图里,"面部特写 / 半身照 / 全身立绘"等**景别(构图)词应紧跟主体、放在 prompt 前段**,不要拖到末尾;且**同一张图只能用一个景别词**(特写/半身/全身互斥)。核心依据是 Flux 对靠前 token 赋予更高权重。

## 核心结论 / Key points

### 1. 景别词放哪:前段,主体之后,不要放末尾

> [!quote] Black Forest Labs 官方
> "FLUX pays more attention to words and concepts mentioned earlier… Lead with the subject: Put the main thing first."(《Prompting Fundamentals》)
> "Framing controls how the subject is positioned… If FLUX keeps pulling too far back, make the subject clear first and move environmental details later."(《Building a Good Prompt》)

官方示例对比:
- 易失控(场景被放大):`Person standing inside a forest fire, …, close-up shot, realistic`
- 更可控:`Person with a strong determined expression, forest fire in the background, close-up shot, realistic`

fal.ai 同理:"把主体埋在长描述末尾会被降权,这是新手最常见的结构性错误。"

### 2. 三种景别不能同时出现

特写(面部)、半身(胸/腰以上)、全身(头到脚)是互斥取景范围。PixelPrompt:"矛盾指令(如 wide angle + extreme close-up)会让模型在两概念间**取平均**,产出构图混乱、比例失真。" → 一个 prompt 只选一个景别词;需要多景别就**固定 seed 与主体描述,分多次生成**。

### 3. 推荐写法与顺序

```
[景别词 + 主体] , [动作/姿态] , [外貌细节] , [风格] , [场景/背景] , [光照] , [相机/镜头参数]
```

- 用**自然语言英文整句**(Flux 偏好,而非 SD 式 booru tag 堆砌);BFL skills 公式 `[Subject]+[Action]+[Style]+[Context]+[Lighting]+[Camera]` 里"相机/技术"是**镜头型号/光圈/胶片**等细碎参数,放后段;**决定构图大小的景别词紧贴主体写**。

**景别对应英文词**:

| 景别 | 英文词 |
|---|---|
| 面部特写 | `close-up shot of [the woman's] face` · `intimate close-up` · `extreme close-up`(仅五官) |
| 半身 | `upper-body portrait` · `medium shot` · `cowboy shot`(约大腿以上) |
| 全身 | `full body shot` · `standing full body shot` · `full-body portrait` · `head-to-toe` |

示例(全身):`Full body shot of a young East Asian woman with long black hair, standing confidently with hands on hips, wearing a modern white dress, anime illustration style, in an outdoor sunny garden, soft natural light, shot on 35mm lens`

> [!note] 澄清 C-1:tag 式 prompt 不是错,要按底模分流
> [[ComfyUI-角色多镜头一致性工作流]] 里出现的 booru tag 式 prompt(`1girl, character sheet, full body, white background`)是给 **SDXL / Illustrious / Pony 二次元底模**用的(它们本就吃 tag);**本页的"自然语言整句"专指 Flux**。两者不矛盾——**底模决定 prompt 风格**。`(word:1.5)` / `::2` 权重语法只在 SD 系有效,Flux 无效。

## 常见问题 / Troubleshooting

| 问题 | 解法 |
|---|---|
| 镜头总被拉远/主体太小 | 进一步前置"主体+景别词",删减背景字数,景别词更具体(`standing full body shot`) |
| 全身图被切腿 | 换竖图比例(768×1344),强化"脚/鞋/站立"描述,减少拉近细节 |
| 景别完全不生效 | 检查是否误用 SD 权重语法 `(word:1.5)`/`::2`(Flux 无效);确认用英文整句而非 tag 堆砌 |
| 背景过糊但想清晰环境 | 增加背景细节字数或挂 anti-blur LoRA |

> [!note] 与 Flux Kontext 图像编辑的区别
> 用 [[Flux-Kontext]] 改现有图景别属于**编辑指令**(明确"改什么、保留什么",如"保持原风格,镜头拉近给猫一个特写"),不在"从零构图排序"范畴。

## 关联 / Connections

- 多景别生产里怎么保持角色一致 → [[ComfyUI-角色多镜头一致性工作流]]
- 景别/镜头/景深的影像学定义 → [[AI视觉生成基础手册#19. 景别 / 镜头景别(Shot Size)]]
- 数据集打标里按景别打标 → [[角色LoRA数据集构建#6. 打标铁律]]

## 来源 / Sources

`sources/20260619/Flux1提示词中构图词的排序规范.md`

## 开放问题 / Open questions

- "三种景别不能共存"基于摄影常识 + 社区实测,**官方未逐字禁止**。
- FLUX.1(T5+CLIP 双编码)与 FLUX.2(单一 VLM)文本编码器不同,但"语序靠前权重更大、主体前置"官方对两代均适用;多数景别措辞来自第三方教程(Next Diffusion/fal.ai/PixelPrompt 等),效果随版本/guidance/LoRA 变化,建议固定 seed 做对照。
