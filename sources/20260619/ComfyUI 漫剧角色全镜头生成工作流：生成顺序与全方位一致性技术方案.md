# ComfyUI 漫剧角色全镜头生成工作流：生成顺序与全方位一致性技术方案

## TL;DR
- **最优生成顺序是"先全身定妆 / 角色设定表（character sheet）→ 裁剪精修出半身和特写"**，而非先画脸再扩展。先用一张完整立绘锁定服装、配色、比例和画风四要素，再裁出局部并对脸做专门的高清精修（FaceDetailer），这是 Mickmumpitz 等主流工作流和中文漫剧社区一致推荐的"地基"做法。
- **重绘 vs 扩图要分方向用：从特写"扩展"到半身/全身用扩图（outpainting，本质是带遮罩的重绘）；从全身"缩"到特写不是扩图也不是重绘，而是裁剪+放大精修。** 2024-2025 的新范式是用 Flux Kontext / Qwen-Image-Edit 这类指令编辑模型直接"换镜头"，免遮罩。
- **2025 年个人漫剧创作者的最佳组合**：二次元底模（Illustrious / Pony / NoobAI 系）出画风 + 角色 LoRA 锁身份与服装 + Qwen-Image-Edit-2511 或 Flux Kontext 做多镜头/多角度衍生 + FaceDetailer 精修脸。纯 IPAdapter/PuLID 适合快速起步，长期高频生产应训练专属角色 LoRA。

## Key Findings

1. **没有"一种技术打天下"——一致性是一套系统**。面部一致靠 IPAdapter FaceID / InstantID / PuLID 或角色 LoRA；身体/服装一致主要靠角色 LoRA 或指令编辑模型；画风一致靠固定底模+风格 LoRA+固定 seed；构图/姿势一致靠 ControlNet。主流"三重锁"方案是 IPAdapter FaceID（脸）+ 角色 LoRA（身体/服装/风格）+ ControlNet（姿势）。

2. **生成顺序：全身设定表优先**。Mickmumpitz 的 Consistent Character Creator 工作流分组顺序为：Multiview（生成多视图）→ Upscale 1st Pass（放大多视图）→ Upscale 2nd Pass（精修脸部与细节）→ Emotions（用 Live Portrait 节点加表情）→ Lighting（用 IC-Light Changer 打光）→ 合成角色表，即先锁完整设计再逐块精修脸。中文漫剧社区把"正侧背+表情+UUID标注"的四视图设定图称为"最通用也最推荐先上的一层地基"。

3. **重绘/扩图的本质**：ComfyUI 官方 Inpaint Examples 文档逐字写道："You can also use similar workflows for outpainting. Outpainting is the same thing as inpainting. There is a 'Pad Image for Outpainting' node to automatically pad the image for outpainting while creating the proper mask."（即扩图就是重绘的一种）。特写→全身=扩图（Pad Image for Outpainting + VAE Encode for Inpainting）；全身→特写=裁剪+放大（FaceDetailer 或 only-masked 重绘）。

4. **2024-2025 新技术显著降低门槛**：Qwen-Image-Edit-2511、Flux Kontext、PuLID-Flux 让"单图生成多镜头多角度并保持身份"变得简单，部分场景已可替代复杂的 IPAdapter+ControlNet 拼装流程。Mickmumpitz 的 Consistent Character Creator 3.8 已升级为基于 Qwen-Image-Edit-2511：据 RunComfy 逐字介绍，"Built around Qwen‑Image‑Edit‑2511, this version adds richer turnarounds, multiple scene variants, close‑ups, try‑on and pose paths, and a dataset export utility."

5. **底模选择决定画风一致性的下限**：漫剧二次元画风首选 Illustrious 系（如 WAI-Illustrious、Hassaku、Plant Milk）、Pony 系、NoobAI 系 SDXL 微调模型；要更强语义理解和编辑能力可用 Flux/Qwen。

## Details

### 一、生成顺序：先全身定妆，后裁剪精修（核心结论）

**推荐方案 A：全身角色设定表（character sheet / turnaround）优先 → 裁剪 → 脸部精修。**

这是权重最高的主流做法。理由是：一次性在同一张图里把"正面、侧面、背面 + 全身比例 + 服装 + 配色 + 画风"全部确定下来，能保证所有视角彼此互相一致（mutually consistent）。之后裁剪不会丢失任何设计信息，只需对裁出的小区域做放大和脸部精修即可。

- Mickmumpitz 的 Consistent Character Creator 工作流模块顺序明确体现这一点（RunComfy 官方分组说明逐字）：Multiview（"Generates multiple views"）→ Upscale 1st Pass（"Upscales the multiple views"）→ Upscale 2nd Pass（"Enhances the face and details"）→ Emotions（"Adds facial expressions… using advanced Live Portrait Nodes"）→ Lighting（"with the IC Light Changer model"）。脸是在身体/多视图确定**之后**才逐块精修的。
- ComfyUI 官方模板 "360 Full-body Turnaround" 同样支持"上传角色，一键生成 360 度全身和特写视图"。
- Flux Kontext Character Turnaround Sheet LoRA（作者 reverentelusarca）可把单图转为 5 视角（正/侧/3-4/背）设定表，专为插画/二次元角色设计。
- 中文漫剧社区（知乎多篇）核心原则一致："把不会变的身份层固化下来，把每个镜头要变的动作/表情/场景拆出来分别控制"，先建 4-6 张参考图的"CHAR 角色资产库"，再分镜出图。

**何时反过来（方案 B：脸先行）**：当使用 Qwen-Image-Edit、Nano Banana 这类指令编辑模型把身份"向外扩展"，或在为训练 LoRA 准备数据集时，可以先生成满意的脸+表情，再扩展到半身、全身。风险是全身设计未在前期锁定，服装/比例可能漂移，因此必须配合身份保持工具（Qwen Edit / PuLID / IPAdapter FaceID）或 LoRA 训练。Civitai 的 LoRA 数据集指南《HOW TO CREATE A PERFECT (or almost) DATASET FOR A CHARACTER LORA》逐字建议：附带工作流可生成约 45 张图，"a good balance between close-ups, half-body shots, and full-figure images"，并输出 1024×1536："I find a vertical format ideal: by including the upper body in the frame, the dataset will produce better consistency results when training."

**结论**：漫剧创作者应采用方案 A——先出一张高质量全身设定表锁定全方位一致性，再裁剪 + 对每个裁切区做脸部精修/放大。方案 B 仅在用现代指令编辑模型或做 LoRA 数据集 bootstrapping 时采用。

### 二、重绘（inpainting）vs 扩图（outpainting）：分方向使用

ComfyUI 官方文档明确"扩图就是重绘的一种"（见 Key Findings 第 3 条逐字引文），扩图只是用 Pad Image for Outpainting 节点自动在边缘补一圈空白并生成对应遮罩，再用重绘填充。

**方向一：特写 → 半身/全身（露出更多身体）= 扩图（outpainting）**。这是正确的方向。规范节点链：
1. **Pad Image for Outpainting**（类名 `ImagePadForOutpaint`）——设置上下左右补边像素和 `feathering`（羽化）参数，输出补边图 + 遮罩。
2. **VAE Encode for Inpainting**——有 `grow_mask_by` 参数。据 ComfyUI Community Manual，该参数**官方默认值为 6**（最小 0、最大 64）："The default value is 6, with a minimum of 0 and a maximum of 64."；扩图较大区域时社区实践常调到 64 以防接缝。
3. KSampler（扩图填空白，denoise 一般高，约 0.95-1.0）→ VAE Decode。
- 模型注意：SDXL/专用重绘模型扩图效果远好于原生 Flux 首轮扩图；"Flux 模型不擅长首轮扩图，所以用 SDXL checkpoint"。分辨率要匹配 SDXL 尺寸以免多手多脚。

**方向二：全身 → 特写（露出更少，放大局部）= 裁剪 + 放大精修，不是扩图也不是普通重绘**。因为没有新区域被揭示，不需要填空白。
- Stable Diffusion Art《Inpainting: A complete guide》（作者 Andrew）逐字指出"only masked"（仅蒙版）重绘的意义正在于此："Inpainting only masked fixes the face. (denoising strength: 0.5)… The most common use case of the only masked option is to regenerate faces in finer detail. … Setting denoising strength to 0.5 is a good starting point."——它把蒙版区裁出来用整张分辨率重画再缩回。
- ComfyUI 里等价做法是 **FaceDetailer**（Impact Pack）或裁剪→放大→img2img。Mickmumpitz 工作流就是裁出姿势后用"Upscale 2nd Pass 精修脸部"，且 Upscale 和 Face Detailer 节点必须用**固定 seed**否则影响一致性。

**现代工具 ComfyUI-Inpaint-CropAndStitch（lquesada）**同时服务两个方向：
- `✂️ Inpaint Crop`：围绕蒙版裁剪（可带 context 区域），自动预缩放、为扩图扩展、填洞、growing/blurring 遮罩、缩放到目标分辨率。
- `✂️ Inpaint Stitch`：把重绘结果无损贴回原图未蒙版区。
- `✂️ Extend Image for Outpainting`：专门把扩图也纳入 Crop-and-Stitch 的能力（缩放、模糊、混合、回贴）。
- 推荐（lquesada GitHub README 逐字）："Use 'InpaintModelConditioning' instead of 'VAE Encode (for Inpainting)' to be able to set denoise values lower than 1."；denoise≈1 完全替换内容、≈0.8 保留部分、0.1 仅边缘匹配；用 `context_expand_factor`（如 2）或 `context_expand_pixels`（如 100）给模型更多上下文。

### 三、ComfyUI 保持角色一致性的主流技术手段对比

| 方案 | 面部一致 | 服装一致 | 画风一致 | 适用场景 | 备注 |
|---|---|---|---|---|---|
| **IPAdapter FaceID / Plus Face** | 强（脸） | 弱 | 中 | 快速起步、单参考图 | 权重 0.7-0.85，太高会"烧脸"僵硬；SDXL 推荐 PLUS FACE 预设 |
| **InstantID** | 很强（写实脸） | 弱 | 弱 | 写实人物、单图 | 基于 InsightFace；二次元效果一般，易过相似、表情不灵活 |
| **PuLID / PuLID-Flux / PuLID-Flux II** | 很强 | 弱 | 中-强 | Flux 工作流、需保画风 | PuLID-Flux II 解决"模型污染"问题，保画风更好；可 8GB 显存跑 |
| **ReActor / FaceSwap** | 强（事后换脸） | 无 | 无 | 修复已生成图、批量换脸 | 在生成后把脸贴上去，适合补救不一致 |
| **角色 LoRA** | 很强 | 很强 | 很强 | 长期高频、专属 IP | 最可靠的长期方案；15-30 张多角度图，rank 16-64，1000-3000 步 |
| **ControlNet（OpenPose/Canny/Lineart/Depth）** | 无（控结构） | 无 | 无 | 锁姿势、构图、线稿 | 与上面方案配合，负责"动作"而非"身份" |
| **Reference-only / Reference latent** | 中 | 中 | 中 | 轻量参考 | Flux 用 ReferenceLatent 链式条件 |
| **Qwen-Image-Edit-2511** | 强 | 强 | 强 | 单图多镜头/多角度/换装 | 指令编辑，多人一致性强，内置社区 LoRA；可能改脸需配 FaceDetailer |
| **Flux Kontext** | 强 | 强 | 强 | 指令换镜头/换背景 | 适合中近景，对全身大幅扩展不可靠 |

**关键判断**：
- 个人漫剧"全方位一致性"（脸+服装+配色+画风）单靠 IPAdapter 不够，因为 IPAdapter FaceID 只锁脸。最稳是**角色 LoRA**（锁身体、服装、风格）或**指令编辑模型**（Qwen Edit/Kontext，原生保服装配色）。
- IPAdapter / PuLID / InstantID 三者对比中，PuLID 通常被评为最新最强的人脸保持方案；但二次元角色用专属 LoRA 往往更稳。

### 四、漫剧/竖屏短剧的特殊考量

- **竖屏 9:16**：分镜需不同景别（特写 close-up、中景 medium shot、全身 full body shot、远景 long shot）。出图时用 `--ar 9:16` 或对应分辨率（如 832×1216 / 768×1344 SDXL 竖屏尺寸）。
- **景别 prompt 词**：close-up（特写脸）、bust shot/upper body（半身）、full body shot（全身）、cowboy shot（七分身）、wide/long shot（远景）。可用 ComfyUI-AdvancedCameraPrompts、Qwen-Camera-Selector 等节点辅助生成镜头描述。
- **可复用角色资产库**：先建"CHAR 资产库"——4-6 张含正/侧/背/表情的设定图 + 一段结构化文字设定（发色、发型、服装、配饰），后续每个分镜"挂参考图"。高频生产则训练角色 LoRA + 固定触发词。
- **底模推荐**：
  - 二次元/日漫漫剧：Illustrious 系（WAI-Illustrious、Hassaku XL、Plant Milk、Diving-Illustrious）、Pony Diffusion V6 XL、NoobAI-XL。
  - 需强编辑/多镜头：Qwen-Image / Qwen-Image-Edit-2511、Flux.1 dev + Kontext。
  - 厚涂/写实漫剧：Flux.1 dev、SDXL 写实系（Juggernaut XL）。

### 五、推荐的完整实操工作流

**方案 1（推荐给二次元漫剧创作者，SDXL 路线）：**
1. **底模出设定表**：选 Illustrious/Pony 二次元底模，用 prompt（`1girl, character sheet, full body, front view, side view, back view, white background, [发色/发型/眼睛/服装/配饰]`）出一张全身三视图设定表。固定 seed。
2. **脸部精修**：FaceDetailer（Impact Pack）对每个视角的脸放大重画，固定 seed 保一致。
3. **裁剪出局部**：用 Image Crop / Inpaint Crop 裁出半身、特写区域；裁后用 Upscale + img2img（denoise≈0.4-0.5）精修。
4. **衍生新镜头**：
   - 需要全新姿势/角度 → 用 ControlNet OpenPose + IPAdapter FaceID（脸）锁身份。
   - 或用 Qwen-Image-Edit-2511：上传设定图，指令"close-up portrait, lock character, lock clothes"出特写；"full body shot, lock character"出全身。
5. **（高频生产）训练角色 LoRA**：用上面生成的 15-30 张多角度图，Kohya_ss 训练（rank 32 / alpha 16 / lr 1e-4 / 约 1500-2000 步），之后任意分镜挂 LoRA + 触发词。

**方案 2（Flux/Qwen 指令编辑路线，最省事）：**
1. 用 Flux.1 dev 或 Qwen-Image 出一张满意的角色全身图。
2. PuLID-Flux II 锁脸 + Flux Kontext / Qwen-Image-Edit-2511 指令换镜头：
   - 特写："zoom in, close-up portrait of the character, keep identity"
   - 全身扩展：Flux Kontext Zoom-Out LoRA 或 Qwen "zoom out, full body, lock character/clothes/pose"
3. 每镜 FaceDetailer + SeedVR2/SUPIR 放大精修。

**方案 3（一键多视图，适合先建资产库）：**
- 直接用 Qwen Edit 2509/2511 MultipleAngles 工作流（fal 的 multiple-angle LoRA + VNCCS Visual Camera Control 节点用图形拨盘选角度），单图生成正/侧/3-4/特写/俯视等多视角，再分别精修。

**关键插件/模型清单**：ComfyUI-Manager、ComfyUI_IPAdapter_plus、PuLID_ComfyUI / ComfyUI-PuLID-Flux2、ComfyUI-Impact-Pack（FaceDetailer）、ComfyUI-Inpaint-CropAndStitch、ControlNet（Union Pro / OpenPose）、ComfyUI-AdvancedLivePortrait（表情）、Qwen-Image-Edit-2511 模型组、Flux Kontext dev、Kohya_ss（LoRA 训练）。

## Recommendations

1. **第一步（立刻做）**：选定一个二次元底模（推荐 Illustrious 系如 WAI-Illustrious），固定 seed，出一张全身三视图角色设定表，锁定服装/配色/画风/比例。这是你的"地基"。
2. **第二步**：用 FaceDetailer 把设定表里每个视角的脸精修一致，然后裁剪出半身/特写区域，对裁切区放大 + img2img 精修（denoise 0.4-0.5）。
3. **第三步（衍生镜头）**：短期用 IPAdapter FaceID + ControlNet OpenPose 组合，或直接上 Qwen-Image-Edit-2511 指令换镜头（特写用"zoom in / close-up, lock character"，全身用 zoom-out）。
4. **第四步（达到量产门槛后）**：当一个角色要出超过几十张图时，**训练专属角色 LoRA**（Kohya_ss，15-30 张多角度图，rank 32/alpha 16/lr 1e-4/约 1500 步），此后所有分镜挂 LoRA + 触发词，一致性和效率最高。
5. **方向判断口诀**：露出更多身体（特写→全身）用**扩图**（Pad Image for Outpainting，grow_mask_by 起始默认 6、扩大区域可调至 64，denoise 0.95+，SDXL 底模）；放大局部（全身→特写）用**裁剪+FaceDetailer/放大**（denoise 0.5），不要用扩图。

**何时切换策略的阈值**：
- 单角色出图 < 20-30 张 → 用 IPAdapter/PuLID/指令编辑即可，不必训 LoRA。
- 单角色出图 > 几十张、或要长期连载 → 训练角色 LoRA。
- 若指令编辑模型（Kontext）在全身大幅扩展时崩坏 → 退回 SDXL + 扩图，或重新从设定表裁切。

## Caveats

- **Flux Kontext 的局限**：实践者反复报告 Kontext "不适合全身镜头，设计用于中景/近景肖像"，特写→全身的大幅扩展可能崩坏。涉及大面积身体揭示时，掩码扩图仍比指令编辑可靠。
- **Qwen-Image-Edit 的局限**：可能出现长宽比漂移、未蒙版区被重渲染/偏色、脸被"抹平"。建议输入用推荐尺寸（1024×1024 等），并配 FaceDetailer 补救脸部。
- **InstantID/ReActor/IPAdapter FaceID 依赖 InsightFace，需商用许可**；商用漫剧需注意授权合规。
- **二次元角色用人脸识别类方案（InstantID/InsightFace）效果通常不如写实脸**，二次元一致性更依赖角色 LoRA 和底模本身。
- **生成顺序的"全身优先"结论**主要基于主流工作流作者（Mickmumpitz 等）和中文漫剧社区实践，而非某条决定性的官方基准；直接搜索 r/StableDiffusion、r/comfyui 未找到一锤定音的对比帖，face-first 在指令编辑模型加持下也完全可行，应按手头模型灵活选择。
- 本报告所列具体参数（denoise、grow_mask_by、LoRA 步数等）多为社区实践经验值，需按你的底模、显存和题材实测微调，并非官方硬性规范。
- 显存：完整一致性工作流（PuLID-Flux/多视图/放大）对显存要求较高，建议 12GB+；8GB 可跑精简版或 GGUF 量化模型。