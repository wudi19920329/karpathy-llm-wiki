---
type: concept
aliases:
  - flux打标工程师
  - Flux 打标 agent
  - flux 打标提示词
tags:
  - agent
  - prompt
  - lora
  - flux
  - tagging
status: draft
created: 2026-07-02
updated: 2026-07-02
source:
  - sources/agent角色/flux通用agent打标提示词.md
related:
  - "[[agent角色]]"
  - "[[z-image-agent打标工程师]]"
  - "[[LoRA打标]]"
  - "[[角色LoRA数据集构建]]"
  - "[[Flux-LoRA-ai-toolkit训练实战]]"
  - "[[JoyCaption]]"
---

# flux-agent打标工程师

## 摘要 / Summary

一份把 LLM 固化成「专业 Flux LoRA 打标工程师」的 **system prompt**([[agent角色]] 里的第二个角色),给 **Flux 家族**(T5 文本编码器、吃自然语言)的角色 LoRA 训练集生成**英文自然语言 caption**。它是 [[z-image-agent打标工程师]] 的**按底模分流镜像**:同一条黄金法则(标可变、省想绑定的)、同样对身份特征硬禁,但输出从「中文结构化短语」换成「英文自然语言 prose」,并按 FLUX 规范(无负面词、灯光优先、front-load、罕见 token 触发词)重写。

## 核心约定 / The convention

### 1. 绑定 / 可变 二分(同 z-image,身份特征硬禁)

- **固定(只绑触发词,caption 里绝不出现)**:脸型、五官、肤色、身材、想焊死的标志发型。**尤其禁止任何发色 / 瞳色颜色词**——`black hair` / `brown eyes` 一律禁用,一次都不行。
- **自由可变(必须英文详述)**:景别/构图、姿态、情绪、服装、配饰、道具、场景、灯光。

> 这条与 [[z-image-agent打标工程师]]、[[LoRA打标]]、[[角色LoRA数据集构建#6-打标铁律-flux-用自然语言没描述的会被绑定|角色LoRA数据集构建 §6]] 完全同源;身份特征的**强指令硬禁**直接呼应 [[JoyCaption]] 页的实测教训(VLM 罐头选项锁不住瞳/发色,必须用「禁用词 + 反例」强指令)。

### 2. 英文自然语言 prose(核心形态)

- 单段**流畅英文散文**,不是逗号短标签、不是中文。Flux 走 T5,只有自然语言长 caption 才喂得好([[Flux-LoRA-ai-toolkit训练实战#打标策略--captioning|Flux 训练实战 §打标策略]]:自然语言优先)。
- 长度约 30–80 词。

### 3. 软结构公式(引导,非死序)

重点前置(Flux 越靠前权重越高),自然铺陈不锁死:

```
触发词 + 类别词 → 姿态/动作 → 服装/配饰/道具 → 场景 → 灯光 → 机位/镜头
```

> 与 z-image 版的「固定 9 段字段序」不同:FLUX 讲究 front-load + prose 连贯,不用机械分段(core-principles:词序影响权重、自然语言最佳)。

### 4. 触发词:罕见 token 领头 + 嵌进句子

- **罕见乱码 token**(`sh4d0wh34rt` / `p3rs0n`)+ 类别词(woman/man),放句首但**嵌进完整自然句**——bghira:Flux 讨厌孤立触发词。**不用角色真名**(防 base 同名知识漏入)。见 [[Flux-LoRA-ai-toolkit训练实战#触发词]]。

### 5. 灯光必写 + 无负面词 + 无恒定画质样板

- **灯光必写**(FLUX 画质命门):golden hour / softbox / rim light 等。
- **无负面词**:禁止 `no X` / `without X`,改正向描述(FLUX 铁律)。
- **不尾随恒定画质样板**(`8k, masterpiece…`):每张都相同的词会被烘焙进模型;相机/胶片/镜头按需自然描述、允许各图不同。
- 禁主观评价词(beautiful/gorgeous);同类元素用词统一。

### 示例(prompt 内附)

```
sh4d0wh34rt woman, a close-up portrait facing the camera with a calm, composed
expression, wearing a deep purple high-neck halterneck cheongsam top with white
ruffled off-shoulder sleeves and a blue butterfly hair ornament, plain dark green
backdrop, soft diffused studio lighting with a gentle rim light, shot on an 85mm lens at f/2.0
```

## 与 z-image 版的对照(按底模分流)

| 轴 | [[z-image-agent打标工程师]] | **flux-agent打标工程师(本页)** |
|---|---|---|
| 语言 | 纯中文 | **纯英文** |
| 形态 | 结构化短语 + 固定字段顺序 | **自然语言 prose**(front-load,不锁死) |
| 触发词示例 | 真名 `diaochan`(与基线有张力) | **罕见 token `sh4d0wh34rt`**(对齐基线) |
| 画质词 | 尾部固定 `8k高清…` | **不加恒定样板**(对齐「恒定词会被烘焙」) |
| 负面词 | (未涉及) | **显式禁止 `no X`**(FLUX 铁律) |
| 黄金法则 / 身份硬禁 | 相同 | **相同** |

> [!info] 差异是「按底模分流」,不是矛盾
> z-image 是通义系原生中文底模 → 中文 + 结构化;Flux 走 T5 英文自然语言 → 英文 prose。同一黄金法则在不同底模上的两种落地。选型总览见 [[LoRA打标#打标风格按底模分流含语言|LoRA打标 §按底模分流]]。

## 变体(仅本页记录,未塞进 prompt)

- **二次元**:caption 加画风类别词(`anime illustration` / `2D anime style`)对冲 Flux 写实偏向,**或**刻意省略把 2D 焊进角色;必要时升 rank ≥32。见 [[Flux-LoRA-ai-toolkit训练实战#二次元专项]]。
- **WD14 + NL 双标**:库内进阶做法——自然语言喂 T5、WD14 tag 喂 CLIP-L,同时喂饱两个编码器。见 [[角色LoRA数据集构建#6-打标铁律-flux-用自然语言没描述的会被绑定|角色LoRA数据集构建 §6]]。

> [!important] Flux.1 vs Flux.2 与本页无关
> 打标是 **caption 风格问题**(共用 T5 自然语言),与底模版本无关,本页不引入任何 Flux.2 专属训练参数。训练超参、Flux.1-dev 语境见 [[Flux-LoRA-ai-toolkit训练实战]](含「Flux.1 vs Flux.2 不要混」铁律)。

## 关联 / Connections

- 所属枢纽 → [[agent角色]]
- 中文/结构化的姊妹角色 → [[z-image-agent打标工程师]]
- 打标黄金法则、按底模分语言选型 → [[LoRA打标]]
- Flux 角色 LoRA 的打标铁律、触发词矛盾澄清、二次元处理 → [[角色LoRA数据集构建#6-打标铁律-flux-用自然语言没描述的会被绑定|角色LoRA数据集构建 §6]]、[[Flux-LoRA-ai-toolkit训练实战#触发词]]
- 身份特征需强指令才锁得住 → [[JoyCaption]]

## 开放问题 / Open questions

- 本 prompt 由 LLM 按 FLUX 规范 + 库内结论生成,尚未在真实数据集上跑过端到端训练验证;caption 质量对 likeness/flexibility 的实际影响待实测(参照 [[Flux-LoRA-ai-toolkit训练实战]] mnemic 对照实验方法)。
- 纯 NL prose vs NL+WD14 双标,哪种对 Flux 角色 LoRA 更优,库内标注为「进阶做法」但无受控对照。
