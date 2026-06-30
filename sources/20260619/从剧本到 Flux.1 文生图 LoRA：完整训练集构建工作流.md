# 从剧本到 Flux.1 文生图 LoRA：完整训练集构建工作流

## TL;DR
- **可行且已成熟**：用 LLM 解析剧本提取角色视觉信息 → Flux.1 生成基准图 → 用 PuLID / Flux.1 Kontext / Redux 等做一致性扩充 → 筛选去重打标 → 用 ostris/ai-toolkit 训练 LoRA，整条链路在 2024–2026 年社区已被反复验证；推荐主路径为 **「Flux.1 Kontext 扩充 + JoyCaption Beta One 打标 + ai-toolkit 训练」**。
- **核心难点是角色一致性与"近亲繁殖"**：纯 AI 素材会触发 model collapse（输出趋同、细节崩坏、"Flux 脸/塑料感"被放大）；务必混入真实参考图、用 expanding（只增不替换）数据循环、严格人工筛选剔除崩图。
- **数据集小而精**：Flux 角色 LoRA 推荐 **15–30 张**高质量、强多样性图，1024px 多分辨率分桶，约 2000 步、rank 16、lr 1e-4（ai-toolkit 默认），24GB 显存即可本地训练。

## Key Findings

1. **Flux 偏好自然语言长描述打标**，而非 SDXL 的 booru tag；社区最佳实践是 JoyCaption（自然语言）+ WD14（tag）双流混合，分别喂给 T5 和 CLIP-L 两个文本编码器。
2. **角色不变特征"不打标"逻辑成立**：未在 caption 中描述的、所有图共有的特征会被"吸收"进触发词/LoRA；要学进 LoRA 的（脸、发型、标志性服装）不打，要可被 prompt 控制的（姿势、表情、背景、临时服装）要打。
3. **角色一致性方案各有取舍**：Flux.1 Kontext（[Pro]/[Max] 2025-05-29 发布、开源 [dev] 2025-06-26 发布）是目前生成训练集最强工具，单参考图即可换姿势/服装/场景，BFL CEO Robin Rombach 称其"achieved state-of-the-art character consistency across multi-turn edits while maintaining interactive inference speeds of 3-5 seconds at 1MP resolution"；PuLID 擅长脸部 ID 保持但全身构图弱；Redux 做变体但光照可能不自然；ControlNet 控结构；In-Context/OminiControl 是新方向。
4. **bootstrap 迭代法可行**：先用极少图训一个"粗 LoRA"，批量生成更多图，筛选后再训精修版——但要警惕近亲繁殖。
5. **工具链已收敛**：本地用 ComfyUI（生成/一致性）+ JoyCaption（打标）+ ai-toolkit/kohya-ss（训练）；云端用 Replicate（ostris trainer）、fal.ai、Civitai onsite trainer。

## Details

### 第一部分：剧本解析与角色信息提取

**剧本中的视觉信息往往是隐性、分散的**——人物外貌可能只在首次登场的动作描述（action line）里出现一句，性格则要从对白和行为推断。系统化方法借鉴学术界的 Dramatron / HoLLMwood / R2 框架：用**分层 prompt chaining**，先抽取角色清单，再逐角色聚合全剧出现的描述。

**推荐方法：用 LLM（Claude / GPT / Gemini）做两阶段解析**

- **阶段一（抽取）**：把剧本分块（按场景或章节，规避上下文窗口限制），让 LLM 抽取每个角色的"原始证据片段"——所有提到该角色外貌、服装、动作、情绪的原文行，附场景编号。这一步要求**只引用原文、不脑补**（防 LLM 幻觉，R2 框架称之为 hallucination-aware refinement）。
- **阶段二（视觉化合成）**：把某一角色的所有证据片段喂回 LLM，让它合成一份**结构化 character sheet**，并显式要求"将非视觉描述转化为视觉线索"。

**把非视觉描述转化为视觉 tag 的 prompt 技巧**：让 LLM 做"性格→外在表现"的映射，例如：
- "孤僻、戒备" → 视觉化为：眉头微蹙、抿嘴、双臂环抱、与人保持距离、冷色调光线
- "贵族出身、傲慢" → 抬下巴、挺直脊背、华贵面料、对称构图
- "饱经风霜的老兵" → 面部疤痕、皱纹、风霜肤色、磨损的装备

**推荐的 character sheet 字段**（输出为 JSON / YAML 便于程序化处理）：

```yaml
character_id: "LIN_protagonist"
trigger_word: "l1nch4r"          # 唯一、无歧义
canonical_appearance:            # 不变特征→不打标→学进LoRA
  age: 28
  ethnicity: East Asian
  hair: "black, shoulder-length, slightly wavy"
  eyes: "dark brown"
  build: "slim, 170cm"
  distinctive: "small scar above left eyebrow"
default_outfit: "worn grey trench coat over black turtleneck"
alt_outfits:                     # 变装→需打标→可prompt控制
  - "formal black suit (gala scene)"
  - "hospital gown (ch.12)"
personality_visual_cues:         # 性格→表情姿态
  - "guarded: furrowed brow, crossed arms"
  - "rare warmth: soft half-smile"
associated_scenes:               # 背景多样性来源
  - "rain-soaked city streets at night"
  - "cluttered detective office"
  - "sterile hospital corridor"
```

**多角色防污染**：（1）**一角色一文件**，分别处理，绝不在同一 prompt 里同时合成多角色；（2）抽取阶段给每条证据标注归属角色，合成时只喂该角色证据；（3）生成基准图时**一次只生成一个角色、纯色或简单背景**，避免多角色同框导致特征串味（Civitai 训练指南明确警告：训练某角色时不要用其与其他角色同框的图）。

### 第二部分：基础参考图（"种子图"）生成

**目标**：用 Flux.1-dev + character sheet 生成首批高质量、有代表性的基准图。

**提示词风格——Flux 用自然语言，不用 tag 堆叠**。Flux.1 有两个文本编码器：T5-XXL（256 token，吃自然语言长句）和 CLIP-L（77 token）。RunDiffusion 给出的范式 caption 是完整句子，例如："a close-up portrait of [trigger] with short black hair and glasses, smiling gently, wearing a green sweater, standing indoors in front of a white bookshelf, soft natural lighting from the left"。

**质量与代表性策略——"覆盖矩阵"**。基于 Civitai "Zen Principle"（保持要训练的东西一致，其余全部多样化），基准图应覆盖：
- **景别**：特写脸部、肩部以上、cowboy shot（腰部以上）、全身
- **角度**：正面、3/4 侧、纯侧脸、略俯、略仰；社区"no-nonsense 角色 LoRA 指南"建议**至少 30% 侧脸/profile 镜头**
- **表情**：中性、微笑、严肃、说话、大笑（至少 4 种）
- **关键约束**：⚠️ **绝不裁掉头顶**——会导致生成时"蛋头/穹顶头"伪影（egg head distortion，ai-toolkit 社区高频坑）；面部应在 ≥70% 图中完整可见、占画面 ≥15% 面积。

**推荐采样配置（Flux.1-dev 文生图）**：
- **CFG（真实 CFG）= 1.0**；Flux Dev 用的是 guidance distillation，真正的调节杆是 **Distilled/Flux Guidance ≈ 2.5–3.5**（降低可减塑料感）
- **采样步数**：30–50 步（角色细节保持）；预览可 20 步
- **采样器**：Euler / 社区 realism 采样器；schedule 可试 Beta
- **负面提示词**：Flux Dev 基本不用 negative prompt（采样配置里 neg 留空）；如需用要走 true-CFG 模式
- **分辨率**：1024×1024 基准

**是否先训"粗 LoRA"（bootstrap）**：当只有 1–3 张参考图、且 Kontext/PuLID 一致性不够时，可行的迭代法：先用极少图（甚至 mnemic 的"单图 + attention masking"法）训一个粗 LoRA → 用它批量生成 → 人工筛选 → 训精修版。**但这是有风险的捷径**（见第七部分近亲繁殖）；若 Kontext 能直接产出一致素材，优先跳过粗 LoRA。

### 第三部分：角色一致性扩充（核心难点）

| 方案 | 原理 | 优点 | 缺点 | 是否需额外模型 |
|---|---|---|---|---|
| **Flux.1 Kontext (dev, 12B)** | 上下文图像编辑，单参考图换姿势/服装/场景 | 一致性最强、无需训练、社区公认"角色一致性不再是难题"；专门有 LoRA 数据集生成工作流；可在消费级 GPU 运行 | 小脸细节会糊；多属性同时改易失败；**超过 6 次迭代编辑后**官方文档警告会出现明显伪影与退化 | Kontext dev 权重 |
| **PuLID-Flux (v0.9.1)** | InsightFace+EVA-CLIP 抽脸部 ID，IDFormer 注入 | 脸部相似度高、tuning-free、单图即可 | 全身/大构图弱、常裁掉头脚；对部分男性脸效果差 | PuLID-Flux + InsightFace |
| **Flux.1 Redux** | 官方图像变体 adapter | 构图一致性好、做背景替换 | 像 IP-Adapter 但变化可控性弱；光照有时不自然 | Redux dev + SigCLIP Vision |
| **ControlNet (Union Pro 2.0)** | OpenPose/Depth/Canny 控结构 | 精确控姿势/构图、可叠加；做 character sheet | Union Pro 算力大；OpenPose 改主体难（需降 strength 至 ~0.2） | InstantX/Shakker Union 或 XLabs |
| **In-Context LoRA / OminiControl / Omini-Kontext** | token 序列条件注入 | 角色插入场景、多参考；学术前沿 | 较新、工作流复杂、需调 delta | 对应 LoRA 权重 |
| **视频帧抽取 (Wan2.1/2.2 I2V, Kling, Runway)** | I2V 生成连续帧再截图 | 天然多角度/表情连续；运动中身份 | 帧质量参差、需去模糊；面部小则身份弱 | 视频模型 |
| **bootstrap quick-LoRA** | 先训粗 LoRA 再批量生成 | 摆脱单参考图限制 | 近亲繁殖风险高 | 训练框架 |

**推荐组合**：以 **Flux.1 Kontext 为主力**生成不同姿势/服装/场景的同角色图（Weird Wonderful AI Art 提供的 WWAA Kontext 数据集生成工作流自带 50 条 prompt 变体），**PuLID 做脸部一致性兜底**（FLUX Redux + PuLID 组合工作流可兼顾构图多样性与脸部一致），**ControlNet OpenPose 用于精确摆姿/做 turnaround sheet**（已有 Flux Kontext Character Turnaround Sheet LoRA 可一图出 5 视角）。注意 Kontext 的多轮编辑上限：尽量从原始种子图单步生成各变体，而非在已编辑图上反复链式编辑（避免累积退化）。

**关键参数**：PuLID weight 0.5–0.8 给变化、1.0–1.5 强 ID；start_at/end_at 控注入时机。Kontext 用 cfg 1.5、LoRA strength 0.5–0.7，参考图建议纯色背景、≥1024px、RGB。

### 第四部分：训练集构建与筛选

**数量档位**（社区共识，Flux 比 SDXL 更省图）：
- **<15 张**：欠拟合，角色僵硬、千篇一律
- **15–20 张**：多数角色的"甜点区"
- **20–30 张**：复杂角色（特殊发型/纹身/配饰）理想区间；r/StableDiffusion 社区高赞贴建议从 20 起步
- **>30–50 张**：边际递减，需更多步数防过拟合；除非多样性同步提升

**多样性配比**：角度（≥5 种）、表情（≥4 种）、光照（≥3 种）、景别（特写/半身/全身均衡）、背景（现代训练观点：**适度变化背景反而帮助模型分离主体**，而非一律纯色）、服装（默认装为主，少量变装但注意一致性）。

**质量筛选标准**：分辨率 ≥1024×1024；锐利无动态模糊；无水印/logo/压缩伪影；优先 PNG/无损格式；剔除主体过小（脸 <15% 画面）的图。

**剔除"看着对、细节崩"的图**：重点检查手指（数量/形态）、脸部对称性、眼睛（异色/畸形）、牙齿、耳朵、首饰几何。AI 生成素材尤其高发，必须**逐图人工过审**，一张坏图可能教坏 LoRA。

**去重（CLIP embedding 聚类）**：AI 批量生成易出近似图。标准流程：用 CLIP（或 DINOv2）抽 embedding → FAISS 近邻检索 → 按余弦相似度阈值（**0.90–0.95**）做连通分量聚类 → 每簇保留一张最高质量代表图。开源工具：NeMo-Curator 的 SemanticDeduplicationWorkflow、SemDeDup 算法；小数据集用 sentence-transformers + community detection 即可。

**分辨率与 bucketing**：Flux 原生支持多分辨率分桶，ai-toolkit 默认 `resolution: [512, 768, 1024]`，自动按长宽比分桶+缩放。1024 为基准；社区注释"flux enjoys multiple resolutions"。裁剪到固定几个安全长宽比（如 1:1、3:4、16:9），避免单图占满某个 bucket。

**数据增强**：水平翻转可用（除非角色有左右不对称标志，如单侧疤痕/发型）；裁剪是 Civitai 推荐的"从少图榨取更多"技巧（一张全身图裁出 cowboy shot、头肩、脸特写）。但增强不能替代真实多样性。

### 第五部分：打标（captioning）

**风格——Flux 用自然语言长描述**。Civitai "Flux Style Captioning" 训练日记直接对比：WD14 tag 是 SD1.5/SDXL 偏好，**自然语言（JoyCaption）是 Flux 的首选**。社区进阶做法（40+ LoRA 经验贴）：**WD14 + LLM 自然语言双标**，确保 CLIP-L 和 T5 两个编码器都拿到信息。

**自动打标工具对比**：

| 工具 | 类型 | 优点 | 缺点 |
|---|---|---|---|
| **JoyCaption Beta One** | 自然语言 VLM（专为训练扩散模型而生、免费开放无审查） | Flux 社区首选；多模式（Descriptive/Straightforward/Booru Tags）；支持指定触发词、可要求"不描述人物外貌特征"（专为角色 LoRA 设计） | 显存占用大；多主体/左右/OCR 偶错；约 1.5–3% glitch 率 |
| **Florence-2 (PromptGen v1.5)** | 轻量 VLM | 仅 1GB、极快、省显存；5 种模式 | 易漏小细节（瞳色/配饰）、可能幻觉 |
| **CogVLM2** | 大 VLM | 精度高、接近 GPT-4V | 重、慢 |
| **WD14 Tagger** | booru tag | 抓具体特征（发色/服装）准；适合做 CLIP-L 流 | tag 风格 prompt 僵硬，不适合单独用于 Flux |
| **GPT-4V / Claude Vision / Gemini** | 商用 VLM | 最准、可定制指令；适合人工辅助校对 | 付费、需 API |

**推荐**：JoyCaption Beta One（Straightforward 或 Descriptive 模式）为主，配合"refer to character as [trigger]"和"do NOT include physical characteristics of the character"两个内置指令；需要 tag 流时叠加 WD14。

**触发词设计**：用**唯一、无歧义的罕见 token**（如 `p3r5on`、`l1nch4r`），避免与自然语言词冲突。放在 caption 开头。ai-toolkit/kohya 支持在 caption 里写 `[trigger]` 占位、训练时自动替换。

**"角色不变特征是否打标"的争议（核心逻辑）**：主流共识——**要学进 LoRA 的不打，要可被 prompt 控制的要打**。AI Q&A Hub 的诊断：若 LoRA "学不到角色身份"，**最快的修复就是把所有角色外貌特征从 caption 里删掉，只留触发词代表**。反之，临时服装、姿势、表情、背景要打标，否则会被锁死进 LoRA（Aerith LoRA 案例：caption 写侧脸/表情/背景，但不写发型和服装）。

**批量打标后人工校对**：自动打标必出错（瞳色、左右、配饰漏标/错标）。流程：批量生成 → 人工逐条核对关键身份特征是否被误打、临时元素是否漏打 → 用 GPT-4V/Claude 做二次校验。

### 第六部分：完整 pipeline 工具链推荐

**开源本地组合（推荐）**：
- **生成 + 一致性**：ComfyUI（Flux.1-dev + Kontext + PuLID + ControlNet Union Pro 2.0 节点）
- **打标**：JoyCaption Beta One（独立 Gradio app 或 ComfyUI-JoyCaption 节点，支持 GGUF 量化省显存）+ WD14
- **去重**：CLIP/DINOv2 + FAISS（或 NeMo-Curator）
- **训练**：**ostris/ai-toolkit**（社区当前最受欢迎、支持 block-level 训练、有 Gradio UI）；备选 kohya-ss/sd-scripts（最成熟、参数最全）、SimpleTuner、X-Flux

**一站式平台（能力边界）**：
- **Replicate**（ostris/flux-dev-lora-trainer）：基于 ai-toolkit，H100/H200，按秒计费，自带 Llava 打标；最省心
- **fal.ai**（flux-lora-fast-training）：**按训练运行计费，官网标价 $2/次（随步数线性增长）**，~10 分钟完成，API 友好；推理端 FLUX.1 [dev] with LoRAs 为 $0.035/megapixel
- **Civitai onsite trainer**：内置 JoyCaption（自然语言）/ WD14（tag）切换，选 Flux Dev + tag 会警告建议用自然语言；可直接发布
- **tensor.art / RunDiffusion**：预设 JSON 配置，适合新手

**本地硬件需求（Flux.1-dev LoRA）**：
- **最低 16GB**（需量化、低 VRAM 模式，慢且偶尔不稳）；4090 16GB 有 OOM 报告
- **推荐 24GB+**（RTX 4090/3090）：ai-toolkit `train_lora_flux_24gb.yaml` 默认即为此档
- 训练时长：4090 上 30 张图 2000 步约 2.5–3 小时；3080 约 4–5 小时
- Flux.1-schnell：用对应 schnell 配置；Flux.2-dev 训练需 80GB+ 级别（详见 Caveats）

**ai-toolkit 默认关键参数（`train_lora_flux_24gb.yaml`，本地 24GB 档，2026-06 实测官方文件）**：
- 网络：`linear: 16`、`linear_alpha: 16`（rank=alpha=16）
- 优化器/学习率：`optimizer: adamw8bit`、`lr: 1e-4`、`noise_scheduler: flowmatch`
- 步数/批：`steps: 2000`（注释建议 500–4000）、`batch_size: 1`、`gradient_accumulation_steps: 1`
- `train_unet: true`、`train_text_encoder: false`、`gradient_checkpointing: true`、`dtype: bf16`、`quantize: true`（8-bit）
- EMA 默认开：`use_ema: true`、`ema_decay: 0.99`
- 分辨率：`resolution: [512, 768, 1024]`；`caption_dropout_rate: 0.05`；`cache_latents_to_disk: true`
- 保存/采样：`save_every: 250`、`sample_every: 250`、采样 `guidance_scale: 4`、`sample_steps: 20`、`sampler: flowmatch`

**角色 LoRA 步数经验公式**（AI Q&A Hub）：`Steps = (N × 100) + 350`，N=图数；20 张约 2350 步。rank/dim 建议 16–32（过高 128+ 易角色串味）。network alpha 取 dim 的 1/2 或 1/4 给角色 LoRA 更多灵活性、减过拟合。

### 第七部分：常见坑与最佳实践

**1. 近亲繁殖 / model collapse**：用 AI 生成图训 LoRA、再用该 LoRA 生成图训下一代，会陷入"自噬循环"——输出趋同、分布尾部（罕见特征）消失、细节退化（学界称 Habsburg AI / MAD）。Shumailov 等在 *Nature* 631(8022):755–759（2024 年 7 月，DOI 10.1038/s41586-024-07566-y）证实"indiscriminate use of model-generated content in training causes irreversible defects... tails of the original content distribution disappear"；相关研究（Bohacek & Farid）显示 Stable Diffusion 即使仅混入 3% 合成数据递归重训也会崩坏。**缓解**：
- **expanding 数据循环**（只增不替换）：Briesch/Alemohammad 等证明，持续往原始真实数据上叠加新数据，50 代内不退化；而全合成或平衡替换循环会退化
- **混入真实参考图**（混合方案）：哪怕少量真实照片也显著延缓崩坏；Bohacek & Farid 证明仅用真实数据微调可逆转崩坏
- 控制合成比例（业界经验阈值常引用 ≤20%）、每代严格人工筛选

**2. 过拟合判断与处理**：症状——输出与训练图几乎一致、忽略 prompt 变化、不打触发词角色脸也到处出现（character bleeding）。处理：减步数/降 rank（128→16–32）；推理时 LoRA 强度降到 0.5–0.6；加 200–500 张 base model 生成的类别正则图（class prompt 如 "photo of a person"）锚定"人"的概念；训练时盯样图，一旦样图停止改善并开始与训练图雷同就停。

**3. 脸一致但服装/身体漂移**：根因常是 caption 把临时服装/身体特征也"学死"或数据多样性不足。修复：临时服装、姿势必须打标（让其可被 prompt 控制）；增加身体/全身镜头多样性；ai-toolkit 可设 block-level 训练只针对前 15 个 single transformer block（负责外貌编码），减少过度训练。

**4. 多角色剧本：分训 vs 合训**：
- **推荐分训**（一角色一 LoRA）：身份最干净、互不污染、可独立调强度、可组合（多 LoRA 同时加载）；缺点是多角色同框场景需 regional prompt / ControlNet 拼合
- **合训一个多角色 LoRA**：仅当角色总是成组出现、且你能用不同触发词严格区分时；风险是角色间特征串味，调试难
- **决策线**：>2 个主要角色、或角色需单独复用 → 分训；固定 2 人组合且总同框 → 可试合训

**5. 风格污染 / "塑料感、AI 感"被放大**：Flux.1 本身有"塑料/蜡质皮肤"和"Flux 脸/Flux 下巴（同质化脸型）"问题，LoRA 会放大它。**缓解**（多来自 Civitai "Flux Realism Walkthrough" 等社区实践，非官方）：
- 生成阶段降 Distilled/Flux Guidance（甜点约 2.5；真实 CFG 保持 1.0）；采样器试 "[Forge] Flux Realistic" + Beta schedule
- 叠加 realism LoRA（权重 ≤0.8），社区常用组合：**Amateur Photography [Flux Dev] @0.8 + iPhone Photo [FLUX] @0.8 + Phlux @0.4 + UltraRealistic @0.4**；针对"Flux 下巴"有专门的 **Chin Fixer 2000**；皮肤纹理有 **FluxRealSkin、Flux Skin Detailer、Flux Skin Texture** 等
- 用 de-distilled / RLHF 微调的 checkpoint 替代原版 Flux Dev：腾讯 **SRPO**（RLHF 微调 Flux.1-dev，引入真实皮肤微纹理、去油光感；满精度约 47GB，社区有 FP8/BF16/GGUF 量化版）；或官方 **FLUX.1 Krea [dev]**（BFL 与 Krea AI 合作，官方定位"overcomes the oversaturated 'AI look' to achieve new levels of photorealism"）
- 后处理：img2img 低 denoise（0.35–0.45）补皮肤纹理；Ultimate SD Upscale（beta57 调度）+ 皮肤专用 upscaler（如 1x-ITF-SkinDiffDetail-Lite-v1）；Detail Daemon Sampler + Fast Film Grain
- 训练数据里混入真实照片，避免 LoRA 学到纯 AI 质感

## Recommendations

**分阶段执行（端到端示范流程）**：

**阶段 0 — 剧本解析（半天）**
1. 剧本分块 → Claude/GPT 抽取每角色原文证据片段（标场景号、只引原文）
2. 逐角色合成 YAML character sheet（含 trigger_word、不变特征、变装、性格视觉线索、关联场景）
3. 人工校对 sheet

**阶段 1 — 基准图（1–2 小时/角色）**
4. ComfyUI Flux.1-dev，按 character sheet 写自然语言 prompt，CFG=1.0 / guidance≈2.5–3 / 30–50 步 / 1024²
5. 按"覆盖矩阵"生成正脸/侧脸/全身/半身、多表情，纯色或简单背景，挑 3–5 张最佳作"种子"

**阶段 2 — 一致性扩充（半天/角色）**
6. 把种子图喂 Flux.1 Kontext（WWAA 数据集工作流 + 50 条姿势/服装/场景变体 prompt）批量生成；每个变体从原始种子单步生成，避免链式多轮编辑累积退化
7. PuLID 兜底脸部一致性；ControlNet OpenPose 补特定姿势/turnaround
8. **混入真实参考图**（若有），目标 expanding 而非纯合成

**阶段 3 — 筛选打标（半天）**
9. 人工逐图过审，剔除手/脸/眼崩图
10. CLIP+FAISS 去重（阈值 0.92），每簇留一张
11. 收敛到 15–30 张强多样性图，1024px 多分辨率
12. JoyCaption Beta One 自然语言打标（指定 trigger、不描述不变特征）+ WD14 tag 流；人工校对

**阶段 4 — 训练与验证（3 小时）**
13. ai-toolkit `train_lora_flux_24gb.yaml`：rank 16、lr 1e-4、adamw8bit、步数=(N×100)+350、EMA on
14. 每 250 步看样图，样图停止改善即停（防过拟合）
15. 推理测试：触发词 on/off 对比、LoRA 强度 0.5–1.0 扫描、多 prompt 验证泛化
16. 不满意则回到阶段 2/3 调数据（不要先调超参）

**会改变上述建议的阈值/基准**：
- 若样图在 <1000 步就与训练图雷同 → 过拟合，减步数/降 rank/加正则图
- 若不打触发词角色脸也出现 → character bleeding，降 rank 到 16–32、加 100–200 张正则图
- 若角色身份学不到（生成的是路人脸）→ 删 caption 里的外貌描述、增多样性、增步数
- 若塑料感严重 → 降 guidance、叠 realism LoRA、混真实图、换 SRPO / Flux Krea
- 若 >2 主角需复用 → 分角色各训一个 LoRA

## Caveats

- **Flux.2 / Klein / Krea 是新变体**：FLUX.2 [dev]（2025-11-25 发布，32B flow-matching transformer + Mistral-Small-3.2-24B 文本编码器）满精度推理官方注明需 "H100-equivalent GPU"（约 62GB 级别），LoRA 训练更重，量化后才可在 RTX 4090 上跑；FLUX.2 支持最多 **10 张**多参考图合成、输出最高 4MP，对一致性是利好但属较新能力。本方案以 Flux.1-dev 为主轴，Flux.2 相关硬件需谨慎评估。
- **部分数值是社区经验非官方**：步数公式、guidance 甜点值、realism LoRA 权重、SRPO 评测等来自 Civitai/Reddit 实践者与单一测评者，非 Black Forest Labs 官方文档，应以小批实验验证。
- **一致性方案演进极快**：自 2024.8 Flux 发布以来，Kontext（2025）、各类 In-Context/OminiControl 方案仍在快速迭代，部分工作流（如 omini-kontext）较新、文档不全、需自行调参。
- **model collapse 的严重性有学术争议**：部分研究（"Model Collapse Does Not Mean What You Think"）指出在真实+合成数据累积场景下崩坏可被避免，不必过度恐慌；但对小数据集角色 LoRA，混入真实图、严格筛选仍是稳妥做法。
- **法律/伦理**：用真实人物照片（PuLID 等）需获肖像权同意；Flux.1-dev 为非商用许可（dev），商用需确认授权。