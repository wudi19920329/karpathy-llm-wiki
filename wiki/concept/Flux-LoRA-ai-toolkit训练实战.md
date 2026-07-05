---
type: concept
aliases:
  - ai-toolkit训练Flux角色LoRA
  - Flux角色LoRA训练参数
  - Flux LoRA training
tags:
  - lora
  - flux
  - ai-toolkit
  - training
status: stable
created: 2026-06-30
updated: 2026-06-30
source:
  - "[[#来源]]"
related:
  - "[[角色LoRA数据集构建]]"
  - "[[ComfyUI-角色多镜头一致性工作流]]"
  - "[[LoRA打标]]"
---

# Flux-LoRA ai-toolkit 训练实战

## 摘要 / Summary

在单卡 24GB(3090/4090)上用 Ostris **ai-toolkit** 训 [[LoRA|Flux.1-dev 角色 LoRA]] 的落地参数与调试法。基线就是官方 `train_lora_flux_24gb.yaml` 默认值,几乎不用大改。**Flux 与 SD 最大的区别是极易过拟合**——重点不是堆步数,而是每 250 步存档 + XYZ 网格对比 + 早停。数据集制作见 [[角色LoRA数据集构建]]。

## 基线配置 / Baseline (24GB)

官方 `config/examples/train_lora_flux_24gb.yaml` 默认即社区共识基线:

| 参数 | 值 | 备注 |
|---|---|---|
| network rank/alpha | `16 / 16` | 角色 16;焊死二次元画风或欠拟合→32;>32 留给风格 LoRA |
| optimizer | `adamw8bit` | 省显存默认;prodigy(lr=1 自调)/adafactor 为备选 |
| lr | `1e-4` | 见下方分歧 #2 |
| noise_scheduler | `flowmatch` | Flux 是 rectified flow,**采样器必须与之一致** |
| dtype / quantize | `bf16` / `true`(8bit) | 24GB 必开量化;接显示器再加 `low_vram: true` |
| resolution | `[512, 768, 1024]` | "flux enjoys multiple resolutions",自动分桶 |
| batch_size | `1` | |
| steps | `1500`(少样本 1000–2000) | 官方注释"500–4000 合理";见分歧 #4 |
| EMA | `use_ema: true, ema_decay 0.99` | 平滑、提升小数据集稳定性;紧显存可关 |
| caption_dropout_rate | `0.05` | 助泛化 |
| save_every | `250` | 多 checkpoint 便于挑选,`max_step_saves_to_keep` 调大 |
| train_text_encoder | `false` | 官方注释"probably won't work with flux" |
| cache_latents_to_disk | `true` | 省显存,必开 |

**显存/耗时**:24GB + 64GB 内存,开 8bit 量化 + gradient_checkpointing + cache_latents,512–1024 分辨率绰绰有余;~1500 步在 3090 上约 2–2.5 小时(含预处理)。

## 训练参数的来源分歧 / Cross-source tensions

> [!warning] 分歧 #2:学习率 1e-3 能不能用?
> - 〔ai-toolkit 落地方案〕:动漫/卡通 1e-4 **可能不收敛,需阶梯提到 0.001–0.002**(收敛更快但更易过拟合)。
> - 〔构建角色 LoRA 数据集〕:**"避免使用 1e-3——那是 SDXL 的约定"**,无脑抄会"产生垃圾"。
>
> **调和**:(a) 升 lr 是**条件性**建议(1e-4 欠拟合才上调);(b) **有效学习率 = lr × alpha/rank**,脱离 alpha/rank 谈数字没意义(那个 1e-3 案例是 Dim 4 / Alpha 32 的放大配置);(c) Flux 对 lr 极敏感,改千分之几就可能崩。
> **推荐基线**:先 `1e-4`;动漫欠拟合再**小步**阶梯上调到 `4e-4`(ai-toolkit Web UI 默认就是 4e-4),仍不收敛再谨慎试更高。**不要直接套 SDXL 的 1e-3。**

> [!warning] 分歧 #3:alpha = rank 还是 rank/2?
> - 〔构建角色 LoRA 数据集〕:**alpha = dim**(Ostris 默认),把 `alpha = dim/2` 标为**"旧 SD 约定"**。
> - 〔LoRA 训练集制作完全指南〕/〔从剧本到 Flux LoRA〕:alpha = dim 的 **1/2(或 1/4)**。
>
> alpha 缩放有效学习率,故此项与 #2 联动。
> **推荐基线**:Flux 上以 **`alpha = rank`(16/16)** 为主(两篇 Flux 实操篇都指向 Ostris 官方);`dim/2` 作为"给角色更多灵活度/减过拟合"的备选,但记得它会同时降低有效 lr。

> [!warning] 分歧 #4:步数启发式差约 3 倍
> 以 20 张图计:〔从剧本到 Flux LoRA〕`(N×100)+350` ≈ **2350 步**;〔构建角色 LoRA 数据集〕`~40 步/图` ≈ **800 步**;〔指南〕**≥1500–3000**;〔ai-toolkit〕**1000–2000**(mnemic 实测 1050)。
> **推荐基线**:不主推任何单一公式,设 `steps 1000–2000` 上限,**以"每 250 步看样图、停止改善即早停"为准**。Flux 收敛快,常在明显过拟合前的 500–1000 步就到最佳。

> [!warning] 分歧 #5:正则化图(reg images)默认用不用?
> - 〔指南〕/〔从剧本到 Flux LoRA〕:把 200–500 张正则图当**标准抗过拟合/抗 character bleeding 手段**。
> - 〔ai-toolkit 落地方案〕:少样本角色 LoRA **"通常不需要"**,会让步数翻倍,多位资深训练者明确不用。
>
> **调和**:语境不同——ai-toolkit 篇是 **Flux 少样本**,指南偏通用/含 SDXL/真人。
> **推荐基线**:**Flux 少样本默认不用**;当训练后出现明显风格污染/能力遗忘(男女不分、特征漏进背景人物、无法再换风格)时,再引入 **20–50%** 正则图(且正则图要接近 base 模型自身生成的样子)。

## 打标策略 / Captioning

> 打标的通用概念(定义、WD14/BLIP 方法对比、格式示例)见 [[LoRA打标]];本节是 ai-toolkit/Flux 训练实战中的具体权衡与触发词矛盾澄清。

- **自然语言优先**(T5 友好);二次元可混 WD14 tag(会"自动多带一点动漫味",对二次元有益)。
- mnemic 角色对照实验(Shadowheart,30 张)的关键权衡:
  - **[[JoyCaption]] 长自然语言 + 无触发词 → likeness(相似度)最佳**;
  - **无打标 + 仅触发词 → flexibility(灵活度)最佳**(换服装/动作灵活,但变性别能力弱)。
- 铁律:**没描述的属性会被绑进触发词/角色**。想灵活就描述可变属性;想锁死造型就省略它们。

### 触发词

- 用**罕见乱码 token**:ai-toolkit 占位符 `p3rs0n`/`trtcrd`;mnemic 用 `sh4d0wh34rt`(防拆成 shadow+heart 漏入);fal 用 `txcl`。**不用真名**(防 base 模型同名知识漏入)。
- ai-toolkit 里 caption 写 `[trigger]`,配 config 的 `trigger_word` 自动替换。

> [!warning] 矛盾 B-1:触发词放"开头"还是"自然嵌入"?
> - 〔构建角色 LoRA 数据集〕/〔从剧本到 Flux LoRA〕:触发词**置 caption 首位 / 开头**。
> - 〔ai-toolkit 落地方案〕(SimpleTuner 作者 bghira):"Flux does not like trigger words… **孤立触发词只会让模型困惑**,半长自然语言句子效果最好"——Flux 机制上触发词作用偏弱(T5 读上下文语义、且文本编码器不训练)。
>
> **调和**:"首位"与"自然嵌入"不冲突——**触发词放开头,但整条 caption 仍是自然语言句子**;bghira 反对的是"caption 只有一个孤立触发词、没有任何自然语言"。
> **推荐基线**:`[trigger] woman, <自然语言描述构图/姿态/服装/背景…>`——触发词领头 + 自然语言句子主体。

## 二次元专项 / Anime specifics

Flux 原生二次元知识偏弱、且偏写实。要得到干净 2D 观感:
- caption 加画风类别词("anime illustration" / "2D anime style")对冲写实偏向(保留改风格的灵活度);**或**刻意省略画风词把 2D 焊进角色;
- 推理时叠加 anime 风格 LoRA(强度 ~0.3–0.8);
- 必要时升 rank ≥32 把画风训得更牢。

## 推理验证 / Inference & checkpoint selection

- **LoRA 强度** 0.8–1.0(叠风格 LoRA 时风格降 0.3–0.8,总和 ≤1.0–1.2)。
- **采样器** euler + simple/beta(ComfyUI 事实标准);**步数** 20–30(精修 35–50)。
- **guidance** Flux 是 guidance-distilled,**绝不用经典 CFG 7–8**;默认 3.5,动漫想更柔/更插画感降到 2.5–3.0。
- **挑 checkpoint**:ComfyUI **XYZ Plot** 横向对比各步数,prompt 含一个训练集内 + 一个完全集外("dog on a log"测泛化),挑"**相似度够 + 仍能改背景/服装/动作**"那档(通常是明显过拟合前 500–1000 步)。
- **看样图而非 loss**:Flux loss 与画质相关性弱,样图开始与训练图雷同 = 过拟合信号 → 早停。

## 常见问题排查 / Troubleshooting

| 症状 | 处理 |
|---|---|
| 相似度不够(欠拟合) | 增步数;阶梯升 lr(动漫尤甚,4e-4→1e-3);升 rank 32;检查是否过度描述固有特征 |
| 过拟合/僵化 | 减步数、降 lr、降 rank;推理强度降 0.5–0.6;回退更早 checkpoint;必要时加正则图 |
| 风格污染/能力遗忘 | 加 20–50% 正则图;罕见触发词;降训练强度 |
| 偏写实/二次元味不够 | caption 加画风词;叠 anime LoRA;省画风词;升 rank ≥32 |
| 塑料感/"AI 味" | alpha 别太高;lr 别太高(过高烧图);用 1024 训;推理先把步数提到 28–50、guidance ~3.5 再判断 |

> [!important] Flux.1 vs Flux.2 不要混
> 2025 末–2026 大量 Flux.2(dev 32B / klein 9B)教程,其参数(rank 32 默认、网络维度 128/64/64/32、differential output preservation、需 32–80GB 显存)**针对 Flux.2,不与 Flux.1 LoRA 互通**。本页聚焦 Flux.1-dev。

## 关联 / Connections

- 数据集张数配比、景别、背景、打标方法论 → [[角色LoRA数据集构建]]
- 在 ComfyUI 里用训好的 LoRA 出多镜头 → [[ComfyUI-角色多镜头一致性工作流]]
- LoRA 的低秩原理、与 DreamBooth/Textual Inversion 对比 → [[扩散模型条件注入机制综述#四、LoRA(Low-Rank Adaptation)]]
- 本机显卡能否跑(RTX 5090 D 24GB)→ [[本机开发环境]]
- ⚠️ **Web UI 新建 job 的默认值 ≠ 本页基线**(rank 32 / steps 3000 / 样图占位词 / keep×steps 删档陷阱)→ [[ai-toolkit-WebUI默认值陷阱]]

## 来源 / Sources

主依据 `sources/20260619/在 ai-toolkit 上训练 Flux.1-dev 二次元角色 LoRA 的完整落地方案（2025–2026）.md`;训练参数分歧另比对〔LoRA 训练集制作完全指南〕〔从剧本到 Flux.1 文生图 LoRA〕〔构建角色LoRA数据集〕三篇(同目录)。

## 开放问题 / Open questions

- 许可证:Flux.1-dev 为**非商用**,基于它训的 LoRA 继承该限制,商用需走 schnell 或官方授权。
- 数值非唯一解:lr/步数/rank 的"最优"依数据集而变,本页给的是稳健起点,仍需 1–2 轮 checkpoint 对比微调。
