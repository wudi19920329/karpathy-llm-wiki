---
type: entity
aliases:
  - joycaption
  - llama-joycaption
  - JoyCaption Beta One
tags:
  - lora
  - tagging
  - vlm
  - tool
status: draft
created: 2026-07-02
updated: 2026-07-02
source:
  - sources/joycaption-readme.md
  - https://github.com/fpgaminer/joycaption
related:
  - "[[LoRA打标]]"
  - "[[角色LoRA数据集构建]]"
  - "[[Flux-LoRA-ai-toolkit训练实战]]"
  - "[[本机开发环境]]"
---

# JoyCaption

## 摘要 / Summary

**JoyCaption**(fpgaminer)是一个**为训练扩散模型而生**的图像打标 VLM:免费、开放权重、无审查(SFW/NSFW 等量覆盖),目标是打标质量对齐 GPT-4o 而无成本与内容限制。是 [[LoRA打标]] 工具选型表中"NSFW 原生支持"与"自然语言长 caption"两个场景的主力;mnemic 角色 LoRA 对照实验(见 [[Flux-LoRA-ai-toolkit训练实战#打标策略--captioning|Flux 训练实战 §打标策略]])中"JoyCaption 长自然语言 → likeness 最佳"即用它。

## 模型 / Model

| 项 | 值 |
|---|---|
| 当前版本 | **Beta One**(`fancyfeast/llama-joycaption-beta-one-hf-llava`) |
| 架构 | Llama 底座 + LLaVA 视觉接入 |
| 原生精度 | bfloat16,**约 17GB 显存** → 24GB 卡可跑([[本机开发环境|RTX 5090 D 24GB]] ✅) |
| 轻量化 | 8-bit / 4-bit 量化(ComfyUI 节点内置选项);社区另有 GGUF 量化(4-6GB,见 [[LoRA打标]] 选型表) |
| 审查 | 设计目标"Minimal Filtering";但底座 Llama 的安全机制偶发拒答(当通用 VLM 问答时),打标场景基本不受影响 |

## 运行方式 / How to run

三条路,**全部本地、全部不需要 API key**:

1. **ComfyUI 节点**(最省事,已有 ComfyUI 时首选):装节点即用,支持 8/4-bit 量化,与出图/精修同一环境。选型见下节。
2. **Python transformers 脚本**:批量扫目录出 `.txt`,最好自动化、可重复;bf16 需 ~17GB 显存。
3. **vLLM 本地服务**(吞吐最高):
   ```bash
   vllm serve fancyfeast/llama-joycaption-beta-one-hf-llava --max-model-len 4096 --enable-prefix-caching
   ```
   Windows 下官方建议走 Docker(`vllm/vllm-openai:latest` 镜像,README 有现成命令)。

> [!important] vLLM 不需要 API key
> vLLM 是**本地自托管**推理服务器,跑在自己 GPU 上,不联网、不计费。它提供 "OpenAI 兼容 API" 只是**接口协议形状**像 OpenAI,不代表连 OpenAI。官方启动命令(见上)**没有任何 key**;vLLM 的 `--api-key` 是**可选**参数,不设则端点不校验。用 OpenAI 客户端库调用时 `api_key` 字段填占位符(如 `"EMPTY"`)即可——那不是真凭证。与 [[本机开发环境]]"Claude Code Pro 订阅、无 API key"不冲突:打标全程用不到任何云端 key。

## ComfyUI 节点选型(2026-07 复核,首版判断有误已修正)

> [!warning] 首版调研的两个错误,均已核实修正
> 1. 首版主推 `1038lab/ComfyUI-JoyCaption`,但其 **Registry 版本停在 2025-10-27 的 v2.0.2,仓库提交到 2025-12-24 后再无新版发布**(~6 个月未更新、零 GitHub Release)——用户 2026-07 指出后经 `gh api` 核实为真。
> 2. 首版称"均不在 CNR registry,需 git clone"——**错误**,官方 `fpgaminer/joycaption_comfyui` 实际已上架 Registry(id `joycaption_comfyui`),可在 ComfyUI-Manager 里直接搜装。

| 包 | 最后更新 | Registry | 定位 |
|---|---|---|---|
| **fpgaminer/joycaption_comfyui**(JoyCaption 原作者官方) | **2026-02-25 打 tag v1.1.0**:模型加载拆成独立节点 `DownloadAndLoadJoyCaptionModel`(memory_mode: Default/8-bit/4-bit),`JoyCaption`/`JoyCaptionCustom` 只管推理 | ✅ 已上架(`joycaption_comfyui`),Manager 直接搜装 | **现主推**:当前维护、官方出品 |
| 1038lab/ComfyUI-JoyCaption | 提交 2025-12-24(Registry 卡在 2025-10-27 v2.0.2) | 已上架但落后仓库 | 曾经的一体化批量方案(`Image Batch Path`→`Caption Saver`→.txt,GGUF 12 档量化);现降级为次选,维护状态存疑 |
| aidenli/ComfyUI_NYJY(JoyCaption+JoyTag+翻译) | 2026-01-23,v1.22.0 | 已上架,145 stars | 候补;依赖含 `openai`/`gradio_client`(仅供其翻译功能用,不影响打标本身无需 API key) |
| without-ordinary/wo_joycaption_comfyui(fork,加 GGUF) | 2025-07-01,v1.0.4 | 已上架 | 更旧,不推荐 |
| o-l-l-i/ComfyUI-Olm-JoyCaption | 2025-08-27 | — | prompt 可视化编辑前端,复核阶段可选用 |

### 官方节点的代价与补法

读官方 `nodes.py` 源码确认:`JoyCaption`/`JoyCaptionCustom` **显式拒绝 batch>1**(`if image.shape[0] != 1: return ("", "Error: batch size greater than 1 is not supported.")`),且**不带**文件夹批量读取节点、也不带存 `.txt` 节点——这两块 1038lab 版反而都有。

**补法不需要新装东西**:`comfyui-easy-use` 包(常见于 ComfyUI 环境)自带:
- `easy loadImagesForLoop` — 从目录逐张读图循环喂给下游节点(功能对等 1038lab 的 `Image Batch Path`)
- `easy saveText` — 存文本到文件(`comfyui-custom-scripts` 的 `SaveText|pysssss` 同样可用)

> [!warning] `easy loadImagesForLoop`+`easy forLoopEnd` 的图内动态循环实测不可靠(2026-07-02)
> 组合管道 `easy loadImagesForLoop → JoyCaption → easy saveText`,`forLoopEnd.flow` 接回 Start 的 `flow` 输出、`initial_value1` 接 `saveText` 输出(让 `explore_dependencies` 把整条链收进循环体)——**连线完全符合 `comfyui-easy-use` 自己的设计**(读了 `py/nodes/image.py`/`logic.py` 的 `loadImagesForLoop`/`forLoopEnd`/`whileLoopEnd` 完整源码,机制是 `GraphBuilder` + 返回 `{"expand": ...}` 做动态子图递归)。但**真机实跑只处理了第 1 张就停**,模型/打标/存文件本身完全正常,只是没触发第二轮。排查了 `comfyui-lora-manager`、`comfyui-lumi-batcher` 两个介入 ComfyUI 执行链的第三方 hook,读代码确认均只读透传,不是元凶。判断是该节点的 `expand` 动态子图协议与当前 ComfyUI 版本不兼容,**继续深挖投入产出比低,已放弃图内循环**。
>
> **替代方案**:外部脚本逐张提交独立 prompt(`loadImagesForLoop` 每次 `start_index=i, limit=1`,不接 `forLoopEnd`,天然无需任何递归机制)。已验证脚本见项目 `ai-视觉/blue-seventeen-minutes/scripts/batch_caption.py`,在真实 16 张素材上跑通(14 生成 + 2 跳过 + 0 失败)。这是 ComfyUI 批处理更稳的通用模式:**遇到图内动态循环节点(forLoop/whileLoop 类)行为异常时,优先换成外部脚本逐个提交 `/prompt`,而不要在节点内部机制上反复调试**。

⚠️ 显存数字仍有出入:官方 README 称 bf16 ~17GB,1038lab 表格称 bf16 ~8GB——24GB 卡两说法下都够,同会话与出图共存时用 `DownloadAndLoadJoyCaptionModel` 的 8-bit/4-bit 模式。

> [!warning] canned extra_option 锁不住"身份特征不许描述"这类精确要求(2026-07-02)
> 打角色 LoRA 数据集时,黄金法则要求瞳色/发色等身份特征**完全不出现**在 caption 里(见 [[LoRA打标]]"省略想锁进触发词的")。`JJC_JoyCaption` 节点自带一条看似对口的 canned 选项:*"Do NOT include information about people/characters that cannot be changed (like ethnicity, gender, etc), but do still include changeable attributes (like hair style)."*——**实测不够硬**:16 张图的批量打标里,9 张仍写出了眼睛颜色,且**颜色互相矛盾**(同一角色,不同图分别写 brown / green,剧本设定是 amber)。这是最差情况——不是"没锁住",是"锁不住还写得不一致",训练时 LoRA 会被这些矛盾信号拉扯,瞳色/发色真的会漂移,不是理论风险。
>
> **原因推测**:canned 选项的措辞越抽象、举例越不对口(用 ethnicity/gender 举例,但实际要防的是瞳色/发色),VLM 越容易不照办或只部分照办。
>
> **修法**:改用 `JJC_JoyCaption_Custom` 节点(`system_prompt`/`user_query` 完全自定义),直接写"禁用词 + 重复强调 + 具体反例"的强指令,例如:
> ```
> STRICT RULE, follow exactly: never write any color word for the hair or the eyes anywhere
> in your answer. This includes 'black hair', 'dark hair', 'brown eyes', or any other hair/eye
> color word — these are forbidden words, banned, do not use them, even once, even in passing.
> ```
> 重测同一张图,瞳色/发色描述完全消失(只留发型/发饰等样式描述,样式本就该保留);16 张全量复测确认无一处颜色词泄漏。**通用结论:JoyCaption 的罐头 extra_option 适合宽泛的风格控制(光照/景别/构图这类),不适合精确的"某个具体特征绝对不能提"——后者应直接上 `JJC_JoyCaption_Custom` 写清楚"禁用词 + 重复 + 反例"**。项目实操细节(逐角色要额外锁哪些点)见 `ai-视觉/blue-seventeen-minutes/制作规划.md` §二。

## Caption 模式 / Modes

| 模式 | 输出 | 用途 |
|---|---|---|
| **Descriptive** | 长自然语言描述(formal/casual 两种语气) | Flux/T5 系训练 caption 主力 |
| **Straightforward** | 更简洁客观、无铺垫 | 同上,偏"只说画面里有什么" |
| Stable Diffusion Prompt | 自然语言 + booru tag 混合(~97% 稳定) | 直接当训练 prompt |
| Booru tag lists | Danbooru / e621 / Rule34 / 通用 booru(准确率偏低) | 二次元 tag 流(配 WD14 交叉验证更稳) |
| MidJourney | WIP,不稳定 | — |
| Art Critic / Product Listing / Social Media | 分析/商品/社媒文案 | 非训练用途 |

**Extra options**:可在指令里要求包含/排除光照、相机角度、水印、构图、景深、画幅、内容分级(SFW/suggestive/NSFW)、景别与机位高度等——即"**可指令化打标**",比 WD14 固定输出灵活得多。

## 关联 / Connections

- 打标黄金法则(标可变/省略想绑定的)与场景选型表 → [[LoRA打标]]
- 打标必须人工复核、触发词写法 → [[角色LoRA数据集构建#6-打标铁律-flux-用自然语言没描述的会被绑定|角色LoRA数据集构建 §6]]
- "JoyCaption 长自然语言 + 无触发词 → likeness 最佳"实验 → [[Flux-LoRA-ai-toolkit训练实战]]
- 本机可跑性(24GB / docker 29.5.3 可用)→ [[本机开发环境]]
- 剧本→漫剧全流程中打标所处的位置 → [[剧本到AI漫剧生产流水线]]

## 开放问题 / Open questions

- ~~ComfyUI 节点装哪个包~~ → 已答两轮(2026-07 复核后改判):官方 `fpgaminer/joycaption_comfyui`(Registry 可装)+ `comfyui-easy-use` 循环/存文本节点组管道;**本机装好后回填实测显存**(官方 17GB vs 1038lab 表 8GB 的出入待裁)。
- Beta One 之后的正式版(1.0)发布节奏;GGUF 量化档在打标质量上损失多少未见对照。
