# 在 ai-toolkit 上训练 Flux.1-dev 二次元角色 LoRA 的完整落地方案（2025–2026）

## TL;DR
- **最优起点配置**：直接基于 Ostris 官方 `train_lora_flux_24gb.yaml`，改 `network linear/linear_alpha=16/16`（角色用 16，欠拟合或想把二次元风格"焊死"进 LoRA 时升到 32/32）、`lr=1e-4`（动漫/卡通收敛差时可提到 4e-4）、`optimizer=adamw8bit`、`noise_scheduler=flowmatch`、`quantize=true`（8bit）、`dtype=bf16`、`resolution=[512,768,1024]`、`batch_size=1`、`steps≈1000–2000`、每 250 步保存采样。24GB 显卡完全够用，开 `gradient_checkpointing` 与 8bit 量化即可。
- **少样本核心打法**：10–30 张高质量图，保持"角色一致、其余多样"；用裁剪（全身/半身/脸部特写/服饰道具）扩充少样本，开镜像翻转；打标用自然语言（JoyCaption/Florence-2）求相似度与灵活度，或省略打标 + 触发词求最强 likeness；触发词用罕见 token（如 ai-toolkit 占位符 `p3rs0n`/`trtcrd`，或 mnemic 的 `sh4d0wh34rt`）防止概念污染，且要自然嵌入句子，而非孤立首词。Flux 极易过拟合，宁可早停。
- **推理验证**：LoRA 权重 0.8–1.0，采样器 euler/flowmatch + simple/beta，步数 20–30，guidance 3.5（动漫想更柔和降到 2.5–3.0，切勿用经典 CFG 7–8），1024×1024；用 XYZ 网格对比多个 checkpoint，挑"相似度够 + 仍能改背景/服装"的那一档。

## Key Findings

1. **ai-toolkit 官方 24GB 配置就是社区共识的基线**。官方 `config/examples/train_lora_flux_24gb.yaml` 默认（原文确认）：LoRA rank/alpha=16/16、`('lr', 1e-4)`、`('optimizer', 'adamw8bit')`、`('noise_scheduler', 'flowmatch')`、bf16、8bit 量化、`('resolution', [512, 768, 1024]) # flux enjoys multiple resolutions`、batch 1、`('steps', 2000), # total number of steps to train 500 - 4000 is a good range`、EMA on（`ema_decay 0.99`）、`caption_dropout_rate 0.05`。这是为单卡 24GB 设计的，无需大改即可跑。

2. **Flux 角色 LoRA 极易过拟合，是它与 SD1.5/SDXL 最大的区别**。社区普遍报告 likeness（相似度）与 flexibility（灵活度）的权衡很尖锐，步数/学习率稍高就"僵化"——生成结果与训练图几乎一致、无法响应新 prompt。因此小数据集要靠"少步数 + 多 checkpoint 对比 + 早停"控制。

3. **打标在 Flux 上的逻辑被反转**：你"没有描述"的东西会被绑进触发词/角色。要让角色灵活（可换风格/服装），就描述这些可变属性；要把二次元画风"焊死"进角色，就省略画风描述。

4. **触发词在 Flux 里机制上"作用不大"**（因为文本编码器 T5 读的是上下文语义、且文本编码器不参与训练）。SimpleTuner 作者 bghira 在 Discussion #634 中直言："Flux does not like trigger words... T5 understands contextual meaning... A trigger word does nothing more than confuse the model. The best results i got were when it used semi-long captions in natural language."（Flux 不喜欢触发词……孤立触发词只会让模型困惑，半长自然语言句子效果最好。）但仍推荐用罕见乱码 token 防止 base 模型已有的同名知识"漏进来"。

5. **Flux 原生二次元知识偏弱、且偏向写实**。要得到干净的 2D 动漫观感，需主动用画风类别词（"anime illustration"/"2D anime"）对冲其写实偏向，或推理时叠加一个 anime 风格 LoRA。

## Details

### 1. 数据集准备（少样本）

**数量与质量**：Flux 在 10–50 张就能学好角色，社区角色 LoRA 的"甜点"约 20–25 张；低于 15 张时较难泛化，超过 30 张但多样性不足反而易过拟合。核心原则是**"训练目标保持一致，其余一切尽量多样"**——最好的小数据集是"图与图之间唯一的共同点就是你要训练的那个角色/风格"。

**构图多样性**：尽量覆盖正面、45°侧、纯侧、（可能的话）背面与俯/仰视；景别上要有全身、七分身（cowboy shot）、半身、脸部特写。表情、服装、背景都要变化——**没变化的元素会被当成角色的固有部分学进去**。

**少样本扩充技巧**：
- **裁剪**：把一张高清全身图裁成多张——脸部特写、肩部以上、cowboy shot，并单独裁出角色标志性的服饰/武器/挂件/腰带扣等小物件，这能教会模型还原细节。这是少样本角色 LoRA 的标准做法。
- **镜像翻转（flip augmentation）**：可开启以倍增样本，但角色有不对称特征（刘海方向、单边伤疤、单侧饰品、徽章文字）时要谨慎或关闭。
- **数据增强工具**：ai-toolkit/ComfyUI 的训练节点支持 color augmentation、flip、caption shuffle、caption dropout 等；外部可用 Albumentations 类工具做轻度增强，但避免过度扭曲。

**分辨率与 bucketing**：Flux 原生 1024，建议用多分辨率 `[512,768,1024]`，ai-toolkit 会自动缩放并分桶（bucket），**无需手动裁成正方形**。图片只会被缩小不会被放大，所以源图应≥1024。512 训练更快更省显存，细节差异在很多角色上肉眼难辨；要极致细节再上 1024。每个桶最好≥2 张图，避免只有单图的桶。

**正则化图（regularization images）——分情况**：
- 对**少样本角色 LoRA，通常不需要**，且会让步数/时间翻倍。多位资深训练者明确"不用"。
- 但 Flux 确实比 SD 更容易"遗忘"原模型能力（男女不分、角色特征漏进背景人物、无法再 prompt 其他风格）。社区有专门工作流主张正则图占数据集 20–50%、且正则图要尽量接近 base 模型自身生成的样子。**结论：默认不用；当你发现训练后出现明显风格污染/能力遗忘时，再引入正则图。**

### 2. 打标策略（captioning）

**自然语言 vs booru tags**：Flux 用 T5 文本编码器，偏好自然语言。mnemic 的角色打标对照实验（Shadowheart，30 张）结论：
- **JoyCaption 长自然语言 + 无触发词 → likeness（相似度）最佳**；
- **无打标 + 仅触发词 → 灵活度最佳**——mnemic 原文："I can use only the prompt 'A realistic photo of a Sh4d0wh34rt woman'... The model is flexible when it comes to costumes, actions, while still maintaining the character design. Gender-swapping is weaker than the JoyCaption models though."（仅靠触发词 prompt 即可生成，换服装/动作灵活，但变性别能力弱于 JoyCaption 版）；
- WD14 tag 式打标也能用，且会"自动多带一点动漫味"（因为匹配了 Flux 内部的动漫训练数据），对二次元角色反而有益。

**推荐做法（二次元角色，少样本）**：用自然语言为主，可混入少量 booru tag 描述明确的视觉属性（发色、瞳色、服饰）。打标工具：**JoyCaption（自然语言句子，likeness 强）**、**Florence-2（自然描述，写实向）**、**WD14 tagger（booru tag，二次元属性识别准）**。社区流行"WD14 先跑出准确 tag → 喂给 JoyCaption 当锚点 → 生成既准确又自然的句子"的组合流。

**触发词设计**：
- 用**罕见乱码 token**：ai-toolkit 的 `flux_train_ui.py` 占位符示例为 `p3rs0n`/`trtcrd`（提示语："uncommon word like p3rs0n or trtcrd, or sentence like 'in the style of CNSTLL'"）；mnemic 用 `sh4d0wh34rt`（"to make sure it doesn't bleed in the words 'shadow' and 'heart'"，避免拆词漏入）；fal 用 `txcl`。**不要用角色真名**，以免 base 模型已有同名知识漏入。
- **要自然嵌入句子**而非孤立首词（见上文 bghira 的明确论断），把触发词以流畅语言写进 caption 效果最好。
- 在 ai-toolkit 里可在 caption 里写 `[trigger]`，配合 config 的 `trigger_word` 自动替换。

**固有特征：描述还是省略？**
- 想让角色**灵活可变**（换发色/服装/风格）：就把这些可变属性都描述出来。
- 想让角色**外观锁死**（始终同一造型）：省略这些固有特征，它们会被绑进触发词。
- **二次元画风专项**：Flux 偏写实，要么在 caption 里加画风类别词（"anime illustration of…", "2D anime style"）来对冲写实偏向并保留改风格的灵活度；要么刻意省略画风词把 2D 观感焊进角色。fal 的风格 LoRA 经验也印证："Flux 偏向写实风格，我会加一个类别词（如 painting）帮它别生成写实图"。

**caption 长度与风格**：自然语言 1–3 句，描述构图、视角、姿态、表情、服装、背景；保持触发词数量在整个数据集一致。caption_dropout 0.05（5% 概率丢弃 caption）有助泛化。

### 3. ai-toolkit 配置文件（24GB 最优，可复制）

以下基于官方 `train_lora_flux_24gb.yaml`，针对二次元少样本角色调整：

```yaml
job: extension
config:
  name: "my_anime_char_v1"
  process:
    - type: 'sd_trainer'
      training_folder: "output"
      device: cuda:0
      trigger_word: "p3rs0n"        # 罕见 token；在 caption 里用 [trigger]
      network:
        type: "lora"
        linear: 16                  # 角色=16；想焊死二次元风格或欠拟合→32
        linear_alpha: 16            # 同 rank 或取一半
      save:
        dtype: float16
        save_every: 250             # 每 250 步存一个 checkpoint 便于挑选
        max_step_saves_to_keep: 8
      datasets:
        - folder_path: "/path/to/dataset"
          caption_ext: "txt"
          caption_dropout_rate: 0.05
          shuffle_tokens: false
          cache_latents_to_disk: true
          resolution: [512, 768, 1024]
      train:
        batch_size: 1
        steps: 1500                 # 少样本 10–30 张：~1000–2000
        gradient_accumulation_steps: 1
        train_unet: true
        train_text_encoder: false   # 官方注释: "probably won't work with flux"
        content_or_style: balanced  # 角色可用 balanced
        gradient_checkpointing: true
        noise_scheduler: "flowmatch"
        optimizer: "adamw8bit"
        lr: 1e-4                     # 动漫/卡通收敛差→可提到 4e-4
        # linear_timesteps: true    # 实验性曲线加权，可能改善结果
        ema_config:
          use_ema: true
          ema_decay: 0.99
        dtype: bf16
      model:
        name_or_path: "black-forest-labs/FLUX.1-dev"
        is_flux: true
        quantize: true              # 8bit 混合精度，24GB 必开
        # low_vram: true            # 若显卡同时接显示器，开此项更省显存(略慢)
      sample:
        sampler: "flowmatch"        # 必须与 train.noise_scheduler 一致
        sample_every: 250
        width: 1024
        height: 1024
        prompts:
          - "[trigger] anime illustration, full body, standing"
          - "[trigger] portrait, smiling, simple background"
        neg: ""                     # Flux 不用负面
        seed: 42
        walk_seed: true
        guidance_scale: 4
        sample_steps: 28
```

**关键参数解读**：
- **rank/alpha**：角色 16/16 足够（文件仅 ~30–80MB）；想把二次元画风一起训进、或角色造型复杂时升 32/32（有训练者反映 ai-toolkit 上 32/32 才出好结果）。>32 一般留给风格 LoRA，且过拟合风险升高。注：mnemic 的角色实验里用过极低 Dim 2–4（Alpha 16–32）也能成功，说明角色可在很低 rank 训练，但常规仍建议从 16 起步。
- **learning rate**：官方 YAML 默认 1e-4 稳；ai-toolkit 的 Web UI 快速训练（`flux_train_ui.py`）默认 `value=4e-4`、rank `value=16`。有训练者明确指出"Flux 训练动漫/卡通时 1e-4 不收敛，需提到 0.001–0.002，收敛更快但更易过拟合"。建议先 1e-4，欠拟合再阶梯式上调到 4e-4。**Flux 对 LR 极敏感——改动千分之几就可能崩，调时务必小步。** mnemic 实测对照：无打标+触发词版用 Unet LR 0.001（Dim 4/Alpha 32），WD14 版用 0.0005（Dim 2/Alpha 16），均 1050 步、AdamW8Bit。
- **optimizer**：adamw8bit 是省显存的默认；prodigy/adafactor 是自适应选项（prodigy 设 lr=1，让它自调），但 ai-toolkit 上 adamw8bit 是最稳妥共识。
- **timestep sampling**：Flux 用 flowmatch（rectified flow）。`linear_timesteps`（贝尔曲线加权）是实验性选项，对脸/角色这种中等噪声层重要的内容可能有益。
- **text encoder**：`train_text_encoder: false`——官方 YAML 注释即"probably won't work with flux"，且费显存，保持关闭。
- **cache latents**：`cache_latents_to_disk: true` 必开，省显存。
- **EMA**：`use_ema: true, ema_decay 0.99` 平滑学习、提升小数据集稳定性，但占额外显存；24GB 紧张时可关。
- **量化**：`quantize: true`（8bit）是 24GB 跑 Flux.1-dev 的关键；若同时驱动显示器再加 `low_vram: true`。

**显存预算**：单卡 24GB（3090/4090）+ 64GB 内存，开 8bit 量化 + gradient_checkpointing + cache_latents，batch 1，512–1024 分辨率，绰绰有余。一个 ~1500 步的训练在 3090 上约 2–2.5 小时（含预处理）。

### 4. 防过拟合与 checkpoint 选择

- **步数**：10–30 张图，约 1000–2000 步（官方注释"500–4000 合理"）。mnemic 的角色实验用了 1050 步。步数过少丢失角色，过多则"僵化"——无法 prompt 出训练集里没有的内容（经典反例："给猫眼睛射激光"在过训后完全失效）。
- **保存间隔**：每 250 步存一个（`save_every: 250`，`max_step_saves_to_keep` 调大），便于回溯挑选。
- **挑最佳 checkpoint**：在 ComfyUI 里用 **XYZ Plot** 把不同步数的 checkpoint 横向对比同一组 prompt（含一个训练集内、一个完全训练集外的 prompt，如"狗骑摩托/dog on a log"测试泛化）。挑"相似度足够好、但仍能改背景/服装/动作"的那一档——通常是明显过拟合前 500–1000 步的那个。
- **loss 观察**：希望看到 loss 下降后趋于平稳；但 Flux 的 loss 与画质相关性弱，**采样预览图比 loss 数值更可靠**。出现"loss 很低但样图开始劣化/与训练图雷同"即过拟合信号。
- **早停**：样图停止改善、开始与训练图雷同时立即停，不要硬跑到设定步数。

### 5. 训练后测试与推理

- **LoRA 强度**：角色 LoRA 用 **0.8–1.0**；太高压制风格、降低灵活度，太低角色不显。叠加风格 LoRA 时把风格降到 ~0.3–0.8，总和控制在 ≤1.0–1.2。
- **采样器/调度器**：ComfyUI 里 **euler + simple 或 beta** 是 Flux 事实标准；ai-toolkit 内部采样用 flowmatch（对应 rectified flow）。
- **步数**：20–30 为甜点；最终精修可上 35–50。
- **guidance**：Flux 是 guidance-distilled，**不要用经典 CFG 7–8**。默认 3.5；动漫想更柔和/更有插画感降到 2.5–3.0；想更贴 prompt 用 3.5–4。
- **分辨率**：1024×1024 或对应长宽比的桶。
- **在 ComfyUI 测试**：把 `.safetensors` 放进 `ComfyUI/models/loras`，用 LoraLoaderModelOnly 节点，prompt 里带触发词。社区有现成的"LoRA tester / XYZ 网格"工作流批量对比 epoch 与强度。
- **判断质量好坏**：①相似度——角色标志特征（脸/发/瞳/服饰）稳定还原；②灵活度——能换背景、姿态、服装、表情而不崩；③对训练集外 prompt 有响应；④无风格污染（不把无关元素带进画面）。likeness 与 flexibility 兼得才是好 LoRA。

### 6. 常见问题排查

- **角色相似度不够（欠拟合）**：增加步数；提高 LR（动漫尤其需要，向 4e-4→1e-3 阶梯试）；升 rank 到 32；检查打标是否把固有特征过度描述了（描述越多越不绑定）。
- **过拟合/僵化**：减步数、降 LR、降 rank、推理时把强度降到 0.5–0.6、回退到更早的 checkpoint；必要时加正则图。
- **风格污染 / 能力遗忘**（男女不分、特征漏进背景、无法再换风格）：这是 Flux 对小数据集敏感的典型问题——引入 20–50% 正则图、用罕见触发词、降低训练强度。
- **二次元观感不够 / 偏写实**：caption 里加画风类别词对冲 Flux 写实偏向；或推理叠加 anime 风格 LoRA（~0.3–0.8）；或省略画风词把 2D 焊进角色；必要时升 rank（≥32）把风格训得更牢。
- **色彩/细节问题（油腻/塑料感、"AI 味"）**：alpha 别设太高；LR 别太高（过高出静电噪点/烧图）；用 1024 训练保细节；预览步数太低会显"弱"，先把推理步数提到 28–50、guidance 调到 ~3.5 再判断，别误判训练失败。
- **likeness vs flexibility 权衡**（Flux 特有）：这是 Flux 角色 LoRA 的核心矛盾。少样本下用"无打标+触发词"偏 likeness、"详细自然语言打标"偏 flexibility；通过多 checkpoint 对比挑平衡点，而非一味加步数。

### 7. 2025–2026 社区最佳实践与共识

- **Ostris 官方推荐**：直接用 `train_lora_flux_24gb.yaml` 默认（rank 16、lr 1e-4、adamw8bit、flowmatch、8bit 量化、多分辨率、EMA on、caption_dropout 0.05）。ai-toolkit 已成为 Flux LoRA 最主流训练器之一。其 Web UI 快速训练默认 LR 为 4e-4、rank 16。
- **rank/alpha 共识**：角色 16（高端 32），风格 >32；alpha 取等于 rank 或一半。实测在 ai-toolkit 上有时需 32/32 才出好结果。
- **打标共识**：自然语言优先（T5 友好）；二次元可混 WD14 tag；"没描述的会被绑定"是铁律。
- **触发词共识**：罕见 token、自然嵌入句子、不用真名；机制上 Flux 触发词作用弱但能防污染。
- **步数/LR 共识**：少样本 1000–2000 步、1e-4 起步；动漫/卡通可能需更高 LR；早停 + checkpoint 对比是标准流程。
- **推理共识**：强度 0.8–1.0、euler/flowmatch、20–30 步、guidance 3.5（动漫降到 2.5–3）。
- **注意区分 Flux.1 与 Flux.2**：2025 末–2026 出现大量 Flux.2（dev 32B / klein 9B）教程，其参数（如 rank 32 默认、网络维度 128/64/64/32、differential output preservation、需 32–80GB 显存）针对 Flux.2，**不与 Flux.1 LoRA 互通**；本方案聚焦 Flux.1-dev，Flux.2 的"原则"可借鉴但具体数值不要照搬。

## Recommendations

**阶段一（首训，安全基线）**：20–25 张高质量图，自然语言打标（JoyCaption），罕见触发词自然嵌入；官方 24GB 配置改 rank 16/16、lr 1e-4、steps 1500、save_every 250、resolution [512,768,1024]、quantize on、EMA on。跑完用 XYZ 网格对比所有 checkpoint。

**阶段二（按结果调）**：
- 相似度不够 → 升 LR 到 4e-4（动漫尤其）、或 rank 32、或加步数到 2500；
- 过拟合/僵化 → 回退早 checkpoint、降 LR、推理强度降到 0.6；
- 二次元味不够 → caption 加画风类别词 / 推理叠 anime 风格 LoRA / rank 升 32；
- 风格污染 → 加 20–50% 正则图。

**阶段三（发布前）**：固定种子在多组训练集外 prompt 上验证灵活度；确定推荐强度（通常 0.8–0.9）与推理设置（euler+beta，28 步，guidance 3.5）写进模型说明。

**改变决策的阈值/基准**：
- 若样图在 <800 步就与训练图雷同 → 数据集多样性不足或 LR 过高，降 LR 或补图；
- 若 2000 步仍不像 → 提 LR/rank，或检查打标是否过度描述固有特征；
- 若推理需 >1.1 强度才出角色 → 训练欠拟合，重训而非靠强度硬拉。

## Caveats
- **许可证**：Flux.1-dev 为非商用许可，基于它训练的 LoRA 继承该限制；商用需走 schnell 或官方授权。
- **来源混杂 Flux.2**：2026 年大量新教程是 Flux.2 的，参数不通用，已在正文标注区分。
- **数值非唯一解**：LoRA 训练有随机性，LR/步数/rank 的"最优"依数据集而变；本方案给的是经社区与官方验证的稳健区间与起点，仍需按你的具体素材做 1–2 轮 checkpoint 对比微调。
- **触发词机制存争议**：部分训练者报告 Flux 上即使不打触发词、LoRA 加载即生效；触发词更多用于防污染与"聚焦"，而非硬开关。
- **个别实验结论需谨慎外推**：如 mnemic 的极低 rank（Dim 2–4）能成功，是单一角色实验，不代表所有角色都适用极低 rank。