# AI科技资讯 → 短视频/图文 → 半自动发布：2026年7月最优端到端方案

## TL;DR
- **推荐架构：Claude Code 作为编排大脑 + 本地 GPU 微服务群 + 半自动人工确认发布。** 截至2026年7月，没有任何单一开源项目把"AI资讯抓取→多模态理解→短视频/图文生成→双平台发布"整条链路做完，你的"Claude Code Max + 24GB GPU + 本地优先"组合就是当前最优形态——用 Claude Code 的 Skills/MCP/subagents/定时任务把各环节的开源工具"粘"起来。
- **第二段两条路线分别有明确首选：** 方案A（数字人口播）首选 **Wan2.2-S2V**（实测约25.65GB显存，24GB卡必须FP8量化+降分辨率）或部署更简单的 **HeyGem**（10–13GB显存）；方案B（素材混剪）首选 **MoneyPrinterTurbo**（95.8k stars）做"一键成片"，或 **pyJianYingDraft**（3.4k stars）走"剪映草稿自动生成+人工微调"路线。
- **发布段的现实：** 抖音有官方内容发布OpenAPI但需企业认证；小红书无官方发笔记API，只能靠 **xiaohongshu-mcp**（约14.5k stars，作者称同类项目"稳定运行一年多没有出现过封号")做浏览器自动化预填、人工点最终发布。封号风险无法完全排除，务必先实名、限速、用真实登录会话。

## Key Findings
- **信源聚合首选组合：** TrendRadar（35平台热榜+MCP）+ CloudFlare-AI-Insight-Daily（AI垂直日报）+ ai-daily-skill（Claude Code Skill）。
- **视频理解：** yt-dlp/lux 下载 → 本地 faster-whisper large-v3 转录（24GB绰绰有余）→ Qwen3-VL 做帧/图理解。
- **TTS首选 IndexTTS2**（B站开源，情感控制强）或 CosyVoice2；中文资讯播报两者都够用。
- **24GB显存的真正瓶颈只在两处：** 视频生成（Wan2.2 14B系列）和数字人（Wan2.2-S2V 14B），都需量化/降分辨率。文本、TTS、生图、转录都不是瓶颈。
- **发布环节最脆弱：** 逆向API签名方案风险最高，浏览器自动化复用真实会话是社区公认较低风险路径。

## Details

### 第一段：每日热点抓取与多模态理解

**信源聚合层（首选 + 备选）**
- **TrendRadar**（sansan0/TrendRadar，首选热榜聚合）：监控抖音、知乎、B站、微博、财联社等35个平台，关键词精准筛选 + AI分析简报 + MCP接口（可让Claude用自然语言对话分析趋势/情感），支持Docker，数据本地自持。2026年5月仍有活跃PR，维护健康。
- **CloudFlare-AI-Insight-Daily**（justlovemaki，首选AI垂类）：基于Cloudflare Workers聚合AI行业新闻、热门开源项目、前沿论文、科技大V言论，用Gemini摘要，自动发GitHub Pages。后端已迁移至 PrismFlowAgent（支持Docker）。优先支持 Folo 订阅源。
- **ai-daily-skill**（geekjourneyx，Claude Code Skill）：从 smol.ai 取资讯，内置Claude能力分类，生成结构化Markdown，并能一键出网页和小红书封面。可 `/plugin install` 直接装。
- **ai-news-skill / PulseAI**（frankzch）：免费AI Agent Skill，自然语言过滤，抓Reddit/TechCrunch/KOL，支持Claude Code/Codex/OpenClaw。
- **通用RSS：** RSSHub。微信公众号用 **wewe-rss**（cooderl，基于微信读书，较稳定但作者维护压力大、issue积压多，登录态偶有失效）或 **we-mp-rss**（rachelos，Web/API双采集模式）。注意：feedsub老牌方案 werss 已停运，RSSHub微信路由长期不稳定。
- **抓取（非发布）：** **MediaCrawler**（NanmiCoder，约45.1k stars）抓XHS/抖音/B站/微博/知乎等，README原文"项目默认使用 CDP 模式连接用户已有的 Chrome 浏览器…大幅降低平台风控检测风险"。可作为X/Twitter、B站、Hacker News、Reddit的抓取底座。

**视频信源理解**
- **字幕/转录：** yt-dlp（YouTube）+ lux（B站）下载音视频 → **faster-whisper large-v3** 本地转录。large-v3需显存>10G，24GB非常宽裕；可用CTranslate2的int8/float16量化提速，或用large-v3-turbo在质量几乎不降的前提下大幅加速。日常批量建议turbo，追准确率用large-v3。
- **视频帧/图片理解：** **Qwen3-VL**（8B GGUF量化在24GB上舒适运行；30B-A3B MoE的FP8也可单卡24GB，但注意MoE需完整加载专家权重）。原生OCR、中文场景理解、视频多帧理解都强。备选 MiniCPM-V。

**Claude Code编排**：定时任务（cron）触发抓取 → 去重（URL+标题相似度）→ 按热度/关键词排序 → subagent并发提炼每条核心 → 输出结构化每日选题库（JSON/Markdown），供第二段消费。

### 第二段：内容生成

#### 方案A：数字人 / AI口播资讯播报

**首选：Wan2.2-S2V（阿里通义，音频驱动数字人）**
- **显存：** 14B主模型在实时推理阶段单卡实测约 **25.65GB**（来自FSDP在推理时必须执行的"unshard"操作，为实测值非理论峰值），略超24GB，因此24GB卡必须走FP8量化 + 降分辨率。实测参数可参考：384×256分辨率+10片段峰值仅13.8GB（2分17秒出30秒预览）；688×368+100片段（约5分钟成片）显存19.2GB、耗时18分42秒（该实测为4090×4环境，单卡时长会更长）。
- **质量：** 口型/表情同步率高，MoE架构，适合口播资讯。ComfyUI原生支持（下载 wan2.2_s2v_14B_fp8_scaled + wav2vec音频编码器）。社区已有12G显存可用的一键包（降配）。
- **一条60秒资讯口播预期：** 单张24GB卡FP8量化下约需10–25分钟（取决于分辨率/片段数），画质"实用级"。

**备选：HeyGem（硅基智能开源，部署最简单、显存最友好）**
- **显存：** 16GB卡实测峰值10–13GB，24GB无压力。生成时长比约1:4到1:8（生成1分钟视频需4–8分钟，取决于显卡）。
- **质量：** 商用级口型同步，支持1张照片/1秒视频克隆形象与声音。Docker部署（完整约70GB，Lite版13.5GB，Lite更快但只能上传音频驱动不能文字驱动）。**核心生成模型未完全开源**（提供免费镜像）。
- **其他备选：** EchoMimicV3（1.3B，蚂蚁，多模态人体动画，一键包节约显存）、MuseTalk（256×256、30FPS+实时、唇形inpainting）、LatentSync（潜空间唇同步、身份保持强但高分辨率略糊）。SadTalker/Wav2Lip已较老，仅作兜底。

**TTS方案**
- **首选 IndexTTS2**（B站 index-tts）：支持情感向量或参考情感音频，中文资讯播报自然、情感可控，声音克隆逼真。多方对比测评中在情感表达上表现突出。
- **备选：** **CosyVoice2**（阿里，0.5B，可用自然语言控制语气，中英混合稳定）；**GPT-SoVITS**（微调后音色相似度最高，但要训练）；**Fish Speech**、**F5-TTS**（免微调零样本方便）。这几款在24GB上都轻松运行。

**完整流水线A：** 选题文案（Claude）→ IndexTTS2配音 → Wan2.2-S2V / HeyGem 口播视频 → faster-whisper自动生成SRT并烧录字幕（竖屏9:16）→ Qwen-Image/Flux生成封面图。

#### 方案B：素材混剪 + 配音 + 字幕

**首选：MoneyPrinterTurbo**（harry0703，**95.8k stars / Fork 13.9k**，v1.3.0 于2026-06-10发布，新增Coverr素材源、Groq LLM、无配音生成模式）
- 流程：主题/关键词 → LLM写脚本旁白 → 按脚本切词从Pexels/Pixabay拉B-roll → Edge-TTS（默认免费）或本地Whisper做配音+字幕 → MoviePy/FFmpeg合成9:16或16:9的MP4。MIT许可，推荐Docker部署，有FastAPI(:8080)可供Claude Code调用。
- **显存：** 本体轻量（脚本可用云API或本地Ollama），显存主要花在配套的本地TTS/生图上。
- **诚实提醒：** 它自动化的是"拼装"不是"判断"——脚本质量、素材匹配度决定成片质量，本地小模型写的脚本明显弱于前沿模型，资讯类建议用Claude写脚本再喂给它。

**备选：NarratoAI**（linyqh，9.8k stars，v0.8.3 于2026-06-14发布，2026-06-10有0.8.1大版本更新）：影视/资讯解说型，LLM文案+自动剪辑+配音+字幕一站式，支持Qwen2-VL/TwelveLabs Pegasus视频理解、IndexTTS-1.5克隆。适合"已有长视频→解说混剪"。**注意其许可为学习研究用途、非商用。**

**剪映草稿方案：pyJianYingDraft**（GuanYixuan，3.4k stars，v0.2.6 于2026-03-16发布，活跃维护）
- 用Python生成剪映草稿JSON，自动铺素材/字幕/转场/入场动画/音频淡入 → 人工在剪映里微调 → 导出。Windows支持脚本自动导出，Linux/Mac支持生成草稿但需在Windows剪映导出。
- **版本限制（重要）：** 剪映6+加密 draft_content.json，模板模式仅支持≤5.9；批量自动导出仅≤剪映6（7+隐藏了自动化控制）。这是"人工微调"路线里最稳的一环，但要锁定剪映版本。
- **FunClip**（modelscope）：FunASR识别 + LLM智能裁剪，适合从长视频按语义提取高光片段，本地Gradio UI。

**素材来源与生图/生视频**
- **免费素材：** Pexels/Pixabay API（MoneyPrinterTurbo原生集成）。
- **AI生图：** **Qwen-Image**（20B，中文文字渲染/信息图/海报最强，GGUF Q4_K_M约13.1GB，或DiffSynth-Studio低显存模式最低4GB可推理）；**Flux.1 dev**（FP8/GGUF在24GB流畅，写实强但中文弱）；Z-Image（小显存备选）。**中文信息图/带字海报首选Qwen-Image。**
- **AI生视频片段：** Wan2.2 T2V 14B（24GB需GGUF Q4/FP8）或 5B版（官方称TI2V-5B最低8G显存可生成、I2V-A14B最低12G，均需开启共享显存）。

**完整流水线B：** 文案 → Claude生成分镜脚本 → 素材匹配(Pexels)/AI生图/生视频片段 → IndexTTS2/CosyVoice2配音 → MoneyPrinterTurbo自动合成 或 pyJianYingDraft生成剪映草稿人工微调 → 字幕烧录 → 封面。

#### 小红书图文方案
- **文案：** Claude生成爆款标题（悬念/数字/对比）、emoji分点排版、3–5个话题标签策略。
- **配图：** **MD2Card**（md2card.com，Markdown一键转知识卡片，15+主题、长文自动拆分、有MCP Server，可被Claude Code直接调）；开源自建可用 markdown-to-image / 流光卡片 / 小红书封面生成器（html2canvas类，本地渲染）；信息图/数据卡用本地Qwen-Image生成。资讯类图文3:4竖版卡片是主力形态。

### 第三段：半自动发布

**平台官方API现状（2026年7月）**
- **抖音开放平台：** **有**官方内容发布OpenAPI（"发布内容至抖音"直接发视频/图片，另有H5分享流程）。要求：**企业认证**（仅支持企业营业执照注册；蓝V企业认证费为600元/首年，之后每年续费120元），"视频发布及管理"权限对全部审核通过应用默认开通；但"发布携带头条文章""时效新闻标签"等能力仅**定向开放给媒体行业**第三方。限制：视频≤4GB/≤15分钟、图片≤100MB，审核逻辑与端内一致，**带logo/水印大概率导致降权/下架/封号**。→ 有企业资质的话，抖音走官方API最稳妥。
- **小红书：** **无**官方"发笔记"API。开放平台面向电商/小程序/商品工具；蒲公英/聚光/私信是广告与数据API（且高门槛，需大额投放准入），非内容发布API。因此所有第三方工具都靠逆向创作平台(creator.xiaohongshu.com)的web端点或浏览器自动化。

**推荐半自动发布工具（首选 + 备选）**
- **首选 xiaohongshu-mcp**（xpzouying，**约14.5k stars / Fork 2.2k**，349 commits，活跃，含Openclaw集成）：Go + 浏览器自动化的MCP，复用cookie驱动真实web UI发布图文/视频，支持标签、定时发布（1小时–14天）、原创声明、可见范围、多账号。作者风险说明原文：**"原来的项目稳定运行一年多，没有出现过封号的情况，只有出现过 Cookies 过期需要重新登录…我是使用 Claude Code 接入，稳定自动化运营数周后，验证没有问题后开源。"** 关键约束：同一账号不允许多网页端同时登录（会互相踢下线）；作者实操每天发帖量约50篇；新号/未实名会触发实名提醒（非封号，建议先实名）；严禁引流/纯搬运。配套 **x-mcp** 浏览器插件版（Chrome商店），零环境部署、操作可见、复用常用浏览器登录态，对非技术用户更友好。
- **备选 social-auto-upload**（dreammis，12.4k stars，247 commits，README近况说明2026-03-24标注重构期）：Python+Playwright，覆盖抖音/小红书/视频号/快手/B站/百家号/TikTok，CLI支持`--schedule`定时发布，正迁移到 **patchright** 隐身驱动以降低检测。适合抖音无企业资质时的网页自动化预填。
- **其他小红书MCP备选：** RedNote-MCP（iFuryst，Node+Playwright）、xhs-mcp（ShunL12324，多账号+SQLite）、xhs-toolkit（aki66938，含数据分析）。

**封号风险评估（决策依据）**
- **逆向API签名（x-s/x-t，如ReaJason/xhs，约2.1k stars但2025年中后不活跃）：风险最高**——签名算法一轮换即失效、脆弱，无"无封号"记录背书。不推荐作为主力。
- **浏览器自动化复用真实登录会话（cookies/CDP）：社区公认风险更低**——MediaCrawler的CDP模式明确称"大幅降低平台风控检测风险"，social-auto-upload用patchright提升隐蔽性，xiaohongshu-mcp有1年多无封号个案。
- **"预填web上传页 + 人工点最终发布"：** 没找到直接量化证据证明"人工点最后一下"本身比全自动更能防封号；但**已被证实的降风险手段**是：①始终在真实登录的浏览器会话内操作（CDP/cookies复用）②遵守限速（小红书约≤50篇/天）③避免水印/引流/纯搬运④新号先实名。因此"人工确认"的真正价值是**内容质量把关 + 合规兜底**，风控收益主要来自前述四点。

## Recommendations

**从零搭建路线图（先跑通哪段、再扩展哪段）**
1. **第一阶段（约1周）——信源→选题库（最低成本、最快见效，先做这段）：** Docker部署TrendRadar + CloudFlare-AI-Insight-Daily，Claude Code装ai-daily-skill；配faster-whisper large-v3做YouTube/B站转录，Qwen3-VL做图/帧理解。产出每日结构化选题库。这一段几乎不吃显存、当天可见效果。
2. **第二阶段（1–2周）——先小红书图文 + 方案B混剪（风险最低、验证内容）：** 图文用 MD2Card + Qwen-Image；混剪用 MoneyPrinterTurbo 跑通"选题→成片"。此阶段封号风险最小、最快验证你的内容选题是否受欢迎。
3. **第三阶段（2–3周）——接入方案A数字人（追求人设与差异化）：** 先用 HeyGem（显存友好、Docker一键、部署最简单）验证口播形态，再升级 Wan2.2-S2V（FP8+降分辨率）追求质量。TTS先上 IndexTTS2 克隆一个固定播报音色。
4. **第四阶段（约1周）——半自动发布闭环：** 小红书用 xiaohongshu-mcp（先实名、限速≤50/天、初期用headed模式肉眼观察）；抖音有企业资质走官方发布API，否则用 social-auto-upload 预填网页+人工点发布。用 Claude Code 定时任务串起"选题→生成→预填→等你确认"的每日节奏。

**24GB显存瓶颈与降级手册**
- **Wan2.2 T2V/I2V 14B（最大瓶颈）：** 用GGUF Q4_K_M/Q5或FP8-scaled，分辨率降到832×480、81帧起步；显存不够再用5B版（8–12G可用，需开共享显存）。
- **Wan2.2-S2V 14B：** 25.65GB略超24GB，**必须**FP8 + 降分辨率（384×256起）或直接用12G一键包/5B路线。
- **Qwen-Image 20B：** 用GGUF Q4_K_M（约13.1GB）或DiffSynth低显存模式（最低4GB）；FP8需内存充足。
- **调度纪律：** 数字人、TTS、生图、转录**不要同时常驻显存**——让Claude Code按序调用微服务、每步用完即卸载，24GB就能跑完整条链。

**触发调整的阈值（何时改方案）**
- 小红书出现实名/风控/限流提醒 → 立即降发布频率、切headed模式、排查违禁词、暂停当天发布。
- Wan2.2-S2V/T2V频繁OOM → 降一档量化(Q4)、降分辨率/帧数/片段数，或切5B。
- 某开源项目停更超3个月或issue大面积无响应 → 切换到备选（本报告每环节都给了备选）。
- 单条视频耗时>30分钟影响日更节奏 → 数字人退回HeyGem、混剪退回纯Pexels素材+Edge-TTS。

## Caveats
- **数字人核心模型开源程度：** HeyGem核心生成模型未完全开源（仅免费镜像）；厂商宣传的"4K""秒级""100%口型"等多为营销话术，落地以本报告给出的显存/耗时实测数为准。
- **小红书政策风险：** 无官方发布API，所有第三方方案都有政策与封号风险，无法完全排除；xiaohongshu-mcp作者"无封号"是**个案经验，非保证**。矩阵化、高频、搬运会显著抬高风险。
- **Wan2.2-S2V的25.65GB为实测值**，24GB单卡必须量化，有画质/速度代价；多卡实测数据（如18分钟出5分钟片）在单卡上会更慢。
- **剪映版本锁定：** 剪映7+隐藏自动化控制并加密草稿，pyJianYingDraft的模板模式/自动导出功能受版本限制，需锁定较低版本。
- **抖音官方API门槛：** 需企业营业执照 + 蓝V认证（600元/首年、续费120元/年），媒体行业能力还需额外申请；个人号无法用官方发布API，只能走网页自动化。
- **时效性：** 所有star数、版本号、维护状态以2026年7月为准（MoneyPrinterTurbo 95.8k、MediaCrawler ~45.1k、xiaohongshu-mcp ~14.5k、pyJianYingDraft 3.4k、NarratoAI 9.8k、social-auto-upload 12.4k）。AI领域迭代极快，落地前请复核各仓库最新提交与issue活跃度。