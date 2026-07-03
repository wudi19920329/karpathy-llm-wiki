---
type: concept
aliases:
  - LoRA tagging
  - 打标
  - caption 打标
tags:
  - lora
  - tagging
  - dataset
status: draft
created: 2026-07-02
updated: 2026-07-02
source:
  - sources/claude/LoRA打标的概念与应用.md
  - raw/assets/lora-tagging-tool-selection-2026.png
related:
  - "[[角色LoRA数据集构建]]"
  - "[[Flux-LoRA-ai-toolkit训练实战]]"
---

# LoRA打标

## 摘要 / Summary

**LoRA 打标**是训练 [[LoRA]] 时为图片集编写文字描述/标签的过程,为模型提供"图片+文字"配对数据。标注质量直接决定 LoRA 效果:标准确 → 模型学会精准提取目标特征;标混乱 → 关联错误或过拟合。本页是通用入门层;Flux 角色 LoRA 场景下更细的黄金法则、矛盾澄清与推荐基线见 [[角色LoRA数据集构建#6-打标铁律-flux-用自然语言没描述的会被绑定|角色LoRA数据集构建 §6]] 与 [[Flux-LoRA-ai-toolkit训练实战#触发词|Flux-LoRA-ai-toolkit训练实战 §触发词]]。

## 要点 / Key points

### 核心逻辑(黄金法则)

**标出不想让 LoRA 学习/固化的特征,不标想让模型"绑定"学会的特征。**

例:训练一个总穿红衣服、扎马尾的角色 LoRA:
- 想让 LoRA 只学脸/身形,红衣服和马尾可自由更换 → 标注 `red dress`、`ponytail`
- 想让红衣服和马尾也焊死在角色上(生成时自动带出) → 不标注这些特征

这与 [[角色LoRA数据集构建]] 的结论完全一致:"标注你想可变的;省略你想锁进触发词的"——**触发词 = 你没标注的那部分**。

### 打标方法

| 方法 | 说明 |
|---|---|
| 手动打标 | 人工逐张写描述,最准确但耗时 |
| **WD14 Tagger** | 基于 Danbooru 标签体系,tag 式输出(如 `1girl, red_dress, smile`),适合二次元风格 |
| **BLIP / BLIP2** | 自然语言描述式输出(如 "a girl wearing a red dress smiling") |

自动打标常搭配 Kohya_ss、SD-Scripts 等 LoRA 训练工具的配套脚本使用。**无论哪种自动工具,都必须人工复核**——一张错标即可教坏 LoRA(与 [[角色LoRA数据集构建]] 结论一致)。

### 工具选型建议(按场景,2026)

2026 年主流做法不是"哪个更好就只用哪个",而是**按数据集类型选择、甚至组合使用**:

| 场景 | 推荐方案 |
|---|---|
| 二次元/动漫角色 LoRA | WD14(标签)+ JoyCaption Booru 模式,或 [[Qwen3-VL]] |
| 写实人像/复杂场景 | [[Qwen3-VL]](细节和语义理解更强)或 [[Florence-2]]+WD14 组合 |
| 需要 NSFW 内容打标 | [[JoyCaption]](原生支持)或 Qwen3-VL 破限版 |
| 显存有限的设备 | Qwen3-VL 轻量版(4B,4-6GB)或 JoyCaption GGUF 量化版(4-6GB) |
| 追求最高质量、不差钱 | GPT-4o/Claude 等云端 API(质量最好但成本高,且有内容限制) |

> [!info] 与上文二元框架的关系
> 不冲突,是同一判据的**细化**:二次元仍以 tag 式(WD14)为主;写实场景从笼统的"BLIP"细化为更强的 [[Qwen3-VL]] 或 [[Florence-2]]+WD14 组合;新增 NSFW、低显存、追求质量不差钱三种此前未覆盖的场景。

来源:归档截图 `raw/assets/lora-tagging-tool-selection-2026.png`(截图末行文字被裁切,大意为"告知具体 LoRA 类型可给更细建议",非核心结论,未收录)。

### 格式示例

Tag 式(常见于动漫风格 LoRA):
```
1girl, solo, red dress, ponytail, smile, outdoor
```

自然语言式(常见于写实/Flux 类模型):
```
a young woman with a ponytail wearing a red dress, smiling, standing outdoors
```

这与库内已有结论吻合:Flux 偏好自然语言长 caption(走 T5 编码器),SD1.5/SDXL 偏 booru tag(WD14);进阶做法是 WD14 + 自然语言双标,同时喂饱 CLIP-L 与 T5 两个编码器。

### 打标风格按底模分流(含语言)

打标的**格式、语言**都随底模走,大致三支:

| 底模 | 打标风格 |
|---|---|
| SD1.5 / SDXL | 英文 booru tag(WD14):`1girl, solo, red dress` |
| Flux | 英文自然语言长句(T5);二次元可混 WD14——见 [[flux-agent打标工程师]] 这份打标 agent prompt |
| **z-image**(通义系原生中文底模) | **中文结构化短语 + 固定字段顺序**——见 [[z-image-agent打标工程师]] 这份打标 agent prompt |

> [!info] 中文打标不与英文建议冲突
> z-image 对中文语义/文本编码更友好,中文打标是「按底模分语言」的合理分支,不推翻 Flux/SD 的英文方案。换底模就换回对应语言与风格。



### 常见问题

- **触发词(trigger word)**:通常在标签/caption 开头加一个独特词(如角色名缩写),用于生成时"召唤"该 LoRA 学到的特征。放开头这一点与 [[Flux-LoRA-ai-toolkit训练实战]] 中"矛盾 B-1"调和后的推荐基线一致(触发词领头 + 自然语言句子主体)。
- **标签数量适中**:太少 → 模型泛化差;太多 → 稀释关键特征的学习权重。
- **一致性**:同一数据集内标注风格要统一,否则模型学习会混乱。

## 关联 / Connections

- 进阶/Flux 角色 LoRA 专精版打标铁律 + 矛盾澄清 → [[角色LoRA数据集构建]]
- 触发词位置的矛盾与调和(bghira/SimpleTuner 观点 vs 社区惯例) → [[Flux-LoRA-ai-toolkit训练实战]]
- JoyCaption 详情(模型/显存/caption 模式/本地运行三条路,**vLLM 为本地自托管、无需 API key**;实测证据:canned extra_option 锁不住瞳色/发色,需用自定义 prompt)→ [[JoyCaption]]
- 把黄金法则固化成硬约束的中文打标 agent(z-image 底模,结构化短语 + 固定字段顺序)→ [[z-image-agent打标工程师]]([[agent角色]] 枢纽)
- 待写:[[LoRA]] · [[WD14-Tagger]] · [[BLIP]] · [[Kohya-ss]] · [[Qwen3-VL]] · [[Florence-2]] · [[Z-Image]]
- 来源:`sources/claude/LoRA打标的概念与应用.md`、`raw/assets/lora-tagging-tool-selection-2026.png`

## 开放问题 / Open questions

- WD14 与 BLIP/BLIP2 的实际打标准确率、速度对比未见受控评测,来源仅为概念性介绍。
- 单一信源(一段简短问答),尚缺具体打标操作步骤(如批量运行 WD14 的命令行示例);后续如有更细的实操信源可扩充或提升为 stable。
- "Qwen3-VL 破限版"指代不明确(社区微调/prompt 越狱?),截图未展开说明,需后续信源澄清。
