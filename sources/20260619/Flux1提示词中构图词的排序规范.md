# Flux1 提示词中"面部特写 / 半身照 / 全身立绘"构图词的排序规范

## TL;DR
- **位置：构图（景别）词应紧跟主体、放在 prompt 的前段（开头主体之后），而非末尾。** Black Forest Labs 官方《Prompting Fundamentals》明确"FLUX 对越靠前的词权重越大"，并在《Building a Good Prompt》中演示"先写清主体、再写景别"以避免镜头被无意拉远。
- **不可同时出现：同一张图的 prompt 中三种景别词只能用一个。** 特写、半身、全身互相矛盾；PixelPrompt 实测指南指出，相互冲突的取景指令会让 Flux 在两个概念间"取平均"，产出构图混乱、人物比例失真的结果。
- **推荐写法：`景别词+主体 → 动作/姿态 → 外貌 → 风格 → 场景 → 光照 → 相机/技术参数`**，用自然语言（英文整句）书写，景别词紧贴主体之后，把细碎相机参数放到后段。

## Key Findings

**1）构图词放在哪里？——前段，主体之后，不要放末尾。**
Flux 的注意力机制对靠前的 token 赋予更高权重。Black Forest Labs 官方《Prompting Fundamentals》（docs.bfl.ml）逐字写道："FLUX pays more attention to words and concepts mentioned earlier in your prompt. Structure your prompts strategically by front-loading the most important elements. Lead with the subject: Put the main thing first."（FLUX 更关注提示词中靠前提及的词与概念，应把最重要的元素前置，主体写在最前。）

官方《Building a Good Prompt》在 Framing（取景）章节给出可直接照搬的规则，逐字为："Framing controls how the subject is positioned in the image. Prompt order matters here too. If FLUX keeps pulling too far back, make the subject clear first and move environmental details later in the sentence."（取景决定主体在画面中的位置；语序同样重要；若 FLUX 总把镜头拉得太远，就先写清主体、把环境细节放到句子后面。）

官方两个示例对比鲜明：
- 易失控（场景被放大）：`Person standing inside a forest fire, strong determined attitude, close-up shot, realistic`
- 更可控：`Person with a strong determined expression, forest fire in the background, close-up shot, realistic`

可见景别词（close-up shot）紧跟主体与动作描述，环境放其后。fal.ai《Flux 2 Prompt Guide》同样强调："Flux 2 weighs earlier information more heavily. Place critical requirements at the beginning… subject first, details second. Don't hide your primary requirement at the end of a long prompt."（Flux 对靠前信息赋予更高权重，关键要求放开头、细节放其后，不要把主要诉求埋在长 prompt 末尾。）

社区实测亦印证：MyAIForce《How to Control Image Layout and Background Sharpness》发现 "one keyword that made a huge difference is 'closeup shot of.' It's a powerful tool for bringing your subject into focus"，并总结 "reducing the background detail or bringing the subject to the beginning of the prompt will shift the focus"（减少背景描写、或把主体提到 prompt 开头，就能把焦点转移到人物）。

**2）三种景别能否同时出现？——不能。**
特写（面部）、半身（胸/腰以上）、全身（头到脚）是互斥的取景范围，无法在一张静态图里同时成立。PixelPrompt Blog《How to Write Flux Prompts That Actually Work》（imageprompt.cloud）逐字指出："Contradictory instructions — 'wide angle extreme close-up' or 'bright dark atmosphere' cause the model to average between concepts, producing mediocre results."（互相矛盾的指令——例如"广角+极端特写"或"明亮+黑暗氛围"——会让模型在两个概念间取平均，产出平庸结果。）因此一个 prompt 只选一个景别词。若需要同一角色的多景别图，应分多次生成（或在视频/分镜流程中用多帧分别描述）。

**3）推荐写法与顺序。**
Black Forest Labs 官方 skills 仓库（github.com/black-forest-labs/skills）的 SKILL.md 给出的结构公式为：`[Subject] + [Action/Pose] + [Style/Medium] + [Context/Setting] + [Lighting] + [Camera/Technical]`；其 README 给出简化变体 `[Subject] + [Action] + [Style] + [Context] + [Lighting] + [Technical]`。注意公式中"相机/技术"是最后一项——这指的是镜头型号、光圈、胶片等细碎参数；而决定构图大小的"景别词"应按官方 Framing 指南紧贴主体写、不要拖到句尾，以锁定构图。

## Details

### 三种景别的对应英文词（Flux 推荐用自然语言整句）
Flux 偏好自然语言而非 SD 式 tag 堆砌。知乎资深作者总结："Flux 极其偏爱自然语言，而不是 SD 系模型的 booru style tag……你必须要用比较冗长的自然语言。"常用景别词：
- **面部特写**：`close-up shot of [the woman's] face`、`intimate close-up`、`extreme close-up`（仅五官/眼睛）。
- **半身照**：`upper-body portrait`、`medium shot`（腰/胸以上）、`cowboy shot`（约大腿以上）。
- **全身立绘**：`full body shot`、`standing full body shot`、`full-body portrait`、`complete figure shot`。

Next Diffusion《Mastering FLUX.1 Prompts for High-Quality Realistic Portraits》明确把景别词列为相机角度并放在整句开头紧贴主体，其原文列举："Examples of camera angles: Full body shot, Medium shot, Close-up shot, Eye-level shot, Intimate close-up shot"，示例如 `Intimate close-up shot of a seductive woman with ombre…hair`、`Standing full body shot of a graceful East Asian woman …`、`Upper-body portrait of a … woman …`。这印证了"景别词置于主体前/主体紧邻位置"的写法在 Flux 上稳定有效。

### 为什么不要放在末尾
官方与社区一致认为，把核心要素（含主体与你想锁定的构图）埋在长 prompt 末尾，会被 Flux 降低优先级——fal.ai 指南直言 "If you bury the subject at the end of a long description, FLUX may deprioritize it. This is the most common structural mistake we've seen new users make."（把主体埋在长描述末尾会被降权，这是新手最常见的结构性错误。）因此景别词应在前段确立，而细碎的相机参数（镜头、光圈、胶片型号，如 `shot on Fujifilm X-T5, 35mm f/1.4`）可放后段做风格修饰。

### 全身图被"切腿"的常见问题
社区反映 Flux/SD 生成全身图时常截断腿脚。解决思路：在前段明确 `full body shot` 并配合站姿、鞋子等"脚部存在"的描述，同时减少会让镜头拉近的细节描写；必要时把画幅比例调为竖图（如 768×1344），给全身留出空间。

### 与 Flux Kontext（图像编辑）的区别
若使用 Flux Kontext 改变现有图的景别，原则不同：官方建议明确"改什么、保留什么"，例如中文社区示例"保持原始画面风格，镜头拉近，给左下角睡着的猫一个特写镜头"。这属于编辑指令而非从零构图，不在本文"立绘构图排序"范畴。

## Recommendations

**第一步（默认模板，适用绝大多数人物图）：**
```
[景别词 + 主体] , [动作/姿态] , [外貌细节] , [风格] , [场景/背景] , [光照] , [相机/镜头参数]
```
示例（全身立绘）：
`Full body shot of a young East Asian woman with long black hair, standing confidently with hands on hips, wearing a modern white dress, anime illustration style, in an outdoor sunny garden, soft natural light, shot on 35mm lens`

示例（面部特写）：
`Close-up shot of a young East Asian woman's face, calm expression, fair skin and dark eyes, photorealistic, blurred warm bokeh background, soft window light, shot on 85mm f/1.8`

**第二步（如果镜头总被拉远 / 主体太小）：** 按官方 Framing 规则进一步把"主体+景别词"前置，删减背景字数，或给景别词更具体的措辞（`standing full body shot`、`head-to-toe full body shot`）。

**第三步（需要多景别）：** 不要在一个 prompt 里堆 `close-up` + `full body`；固定 seed 与主体描述，分别生成特写、半身、全身三张，以保证人物一致性。

**判断阈值（何时改写法）：**
- 仍被切腿 → 换竖图比例并强化"脚/鞋/站立"描述。
- 背景过糊但想要清晰环境 → 增加背景细节字数或挂 anti-blur LoRA。
- 景别完全不生效 → 检查是否误用 SD 式权重语法 `(word:1.5)`／`::2`，这些在 Flux 中无效；并确认 prompt 用英文整句而非 tag 堆砌。

## Caveats
- Black Forest Labs 关于"语序靠前权重更大""主体前置""无负向提示词"的官方表述见于 docs.bfl.ml 的《Prompting Fundamentals》《Building a Good Prompt》及 FLUX.2 指南；FLUX.1 与 FLUX.2 文本编码器不同（FLUX.1 为 T5+CLIP 双编码，FLUX.2 为单一 VLM），但上述核心原则官方对两代均适用。"景别词紧贴主体、放前段"的最强证据来自《Building a Good Prompt》的 Framing 示例。
- "三种景别不能共存"是基于摄影常识与社区实测（矛盾指令导致取平均，PixelPrompt）的结论；官方文档未逐字写出"禁止同时使用 close-up 与 full body"。
- 多数景别示例与具体措辞来自第三方教程（Next Diffusion、MyAIForce、fal.ai、PixelPrompt 等）而非官方逐字规范；它们彼此印证，但实际效果会随模型版本、guidance 值（FLUX.1-dev 常用 3.5，调低至 2 皮肤更自然）、LoRA 而变化，建议固定 seed 做对照实验。
- 官方 skills 仓库的 `rules/t2i-prompting.md` 原文（GitHub 原始文件）因机器人限制未能逐字获取；本文结构公式取自该仓库 SKILL.md 与 README，框架性原则取自 docs.bfl.ml 对应指南。