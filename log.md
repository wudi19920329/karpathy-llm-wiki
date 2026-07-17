---
type: log
title: Log
updated: 2026-07-02
---

# Log — 吸收 / 查询时间线

> **只追加(append-only)**,新条目加到**末尾**(最新在最下)。每条用统一前缀
> `## [日期] 动作 | 涉及 — 备注`,便于 `grep "^## \[" log.md | tail -5` 取最近若干条。

## [2026-06-29] init | 知识库初始化(LLM Wiki schema)

## [2026-06-30] ingest | SAM3 文本分割 — 归档 sources/comfyui-sam3-docs.md;新建 [[SAM3-文本分割]]、综合 [[ComfyUI-文本分割方案对比]]、MOC [[ComfyUI-分割与精修]];对比页内含 GroundingDINO+SAM2 在 transformers 5.x 的脆性反例(折叠)

## [2026-06-30] ingest | 本机开发环境 — 归档 sources/local-machine-env.md;新建 [[本机开发环境]](type:reference);入 index 新分类 References;[[Windows-Python-中文编码]] 补反链消孤儿

## [2026-06-30] ingest | AI 图像/视频 8 篇(sources/20260619/) — 新建 6 页:[[角色LoRA数据集构建]]、[[Flux-LoRA-ai-toolkit训练实战]]、[[ComfyUI-角色多镜头一致性工作流]]、[[Flux提示词-景别词排序规范]]、[[AI视觉生成基础手册]]、[[扩散模型条件注入机制综述]];矛盾逐一确认后落 callout(配比张数 vs repeat 权重、lr 1e-3、alpha=rank vs /2、步数、正则图、触发词首位 vs 孤立、全身优先 vs 特写优先、tag/自然语言按底模分流、CFG 5-9 vs Flux guidance);[[ComfyUI-分割与精修]] MOC 补"角色生成"段接入

## [2026-06-30] meta | 流程优化 — 把"逐一确认矛盾点→逐一开干"固化进 wiki-ingest 技能(步骤 2/3)与 CLAUDE.md(Ingest 步骤 2.5);存 feedback 记忆 ingest-confirm-conflicts-per-subtopic(起因:本次漏查 B/C/D 子主题被用户点名)

## [2026-06-30] ingest | ComfyUI 搭工作流前查清单方案 — 原创综合(无外部信源):新建 [[ComfyUI-工作流前查清单方案]](synthesis),三类清单「谁最全」对比 + 搭工作流以 /object_info(get_node_info)为权威的固定流程;[[ComfyUI-分割与精修]] MOC 补反链;入 index 综合区。起因:用户要求该规则录 wiki 而非记忆文件


## [2026-06-30] ingest | Detailer-采样参数调优 — 新建 Detailer(SEGS)/FaceDetailer/DetailerForEach 共用采样参数调参页(denoise 优先/cfg=1.0 认底模/guide_size);挂入 ComfyUI-分割与精修 MOC,与角色多镜头一致性页互链,denoise 0.5 vs 0.3-0.45 以语境 callout 调和;归档 raw/assets/detailer-segs-node-panel.png
## [2026-06-30] ingest | 本机 Go 环境 — 探测 go version/env,扩档 sources/local-machine-env.md 加 Go 小节;[[本机开发环境]] 加 Go 表(go1.26.4/GOROOT/GOPATH/工具链)+ 两条 callout(CGO_ENABLED=0 且无 gcc→只能纯 Go 构建;GOPROXY 国际代理非 goproxy.cn)+ frontmatter tags 加 go + 开放问题加 cgo 工具链;index 摘要补 Go 1.26.4

## [2026-06-30] ingest | F2 批量改名工具 — 探测本机 f2.exe 版本(go version -m → ayoisaiah/f2/v2 v2.2.2,纯 Go),归档 sources/f2-help-v2.2.2.md(verbatim --help 快照);新建 [[F2-批量改名工具]](type:reference):心智模型(默认 dry-run/-x 落盘/-u 撤销)、核心标志表、替换变量、整目录顺序编号实战配方({%03d}+--sort natural,本机已验证);[[本机开发环境]] Go 工具行补 [[F2-批量改名工具]] wikilink + related + Connections(印证纯 Go 工具在 CGO 关闭机上 go install 即用);入 index References 区。单源无矛盾。起因:用户用 f2 改 ComfyUI 输出图后要求录 wiki

## [2026-07-01] ingest | 本机环境刷新(keep-current) — 重探本机;硬件+5 大运行时逐项与 2026-06-30 快照一致(无真矛盾,仅磁盘用量小幅陈旧已刷新)。用户拍板「不建新源、并成一节」:[[本机开发环境]] 新增「开发工具 / Dev tooling」段(git 2.54.0 / gh 2.95.0 / docker 29.5.3)+ 缺席工具链 note(rustc/cargo/gcc/clang/cmake 全缺→强化「仅纯 Go 可构建」);Python 段补 uv 托管解释器事实(仅 3.14.5,余为 download available;venv 为 per-project 无全局清单)答旧开放问题;CGO warning 扩 clang/cmake;frontmatter tags +git/docker、updated→07-01。sources/local-machine-env.md 保持 06-30 不动(git/gh/docker 未存档,页内注明补测来源)。index 摘要 + updated 同步

## [2026-07-01] refactor | pages→wiki 按 type 分文件夹 + type 枚举对齐原文 6 类 — git mv 15 页入 `wiki/{summary,entity,concept,comparison,overview,synthesis}/`(folder 名 == frontmatter type);type 迁移 source→summary·moc→overview·reference→entity(本机开发环境/F2)·synthesis→comparison(仅文本分割方案对比);ComfyUI-分割与精修 tag moc→overview。CLAUDE.md/templates 双语枚举注释(summary 摘要|entity 实体|concept 概念|comparison 对比|overview 概览|synthesis 综合)+ 目录树/路径 pages→wiki;三技能与 sources/README 路径 pages→wiki(写页按 type 落文件夹)。index 重排为 6 分类(+对比/Comparisons,信源/Sources→摘要/Summaries,导航/MOC→概览/Overview,删参考/References 并入实体/Entities)、frontmatter type moc→overview。起因:逐字比对 sources/karpathy-llm-wiki.md,对齐原文「The wiki」层命名与六类页

## [2026-07-02] ingest | LoRA打标的概念与应用 — 归档 sources/claude/LoRA打标的概念与应用.md;新建 [[LoRA打标]](concept,通用入门层:黄金法则/WD14 vs BLIP/格式示例/触发词基础);与 [[角色LoRA数据集构建]] §6、[[Flux-LoRA-ai-toolkit训练实战]] §打标策略互链(结论一致,无矛盾,新页作为两者的基础层反向引用)

## [2026-07-02] ingest | LoRA打标工具选型建议截图(2026) — 同一话题追问的后续截图,归档 raw/assets/lora-tagging-tool-selection-2026.png;并入 [[LoRA打标]] 新增"工具选型建议(按场景,2026)"小节(二次元/写实/NSFW/低显存/云端 API 五场景 × 推荐工具,新增 Qwen3-VL、Florence-2、GPT-4o/Claude API);与既有 WD14/BLIP 二元框架无矛盾,系细化;[[角色LoRA数据集构建]] §6 补反链

## [2026-07-02] ingest | 本机开发环境补充 Claude Code 订阅信息 — 用户告知(非命令行探测):Claude Code 走 Pro 订阅套餐,无独立 API key(非按 token 计费)。仿 2026-07-01 git/gh/docker 补测先例,未单独建源,直接并入 [[本机开发环境]] 「开发工具 / Dev tooling」表新增一行;frontmatter tags 加 claude-code、updated→07-02;index 摘要同步。单条事实,无矛盾。

## [2026-07-02] ingest+query | JoyCaption 实体页 + 剧本到AI漫剧生产流水线 — 起因:用户问"JoyCaption 的 vLLM 需要 API key 吗"+要《蓝色十七分钟》打标方案与整体规划。归档 sources/joycaption-readme.md(verbatim,vllm serve 命令无 --api-key,核验通过);新建 [[JoyCaption]](entity:模型/显存/三种本地运行/vLLM 无需 API key callout/caption 模式),补 [[LoRA打标]]·[[角色LoRA数据集构建]]·[[Flux-LoRA-ai-toolkit训练实战]] 三处待写链接;新建 [[剧本到AI漫剧生产流水线]](synthesis,通用六阶段串联六页);[[ComfyUI-角色多镜头一致性工作流]] 补反链。项目专属方案(角色打标表/六阶段实例/Flux.2→Flux.1 修正)写入 ai-视觉/blue-seventeen-minutes 项目,不入 wiki。发现并修正矛盾:项目提示词文件写给 Flux.2,与用户拍板的 Flux.1-dev 及库内"Flux.1 vs Flux.2 不要混"冲突

## [2026-07-02] query | JoyCaption ComfyUI 节点选型(2026-07) — 调研社区 4+ 封装(web + registry + 本机已装清单):批量打标首选 1038lab/ComfyUI-JoyCaption v2.0.2(Image Batch Path→Caption Saver 存 .txt、GGUF 12 档、Clear After Run),官方 fpgaminer 版无批处理;均不在 CNR registry 需 git 装。回填 [[JoyCaption]] 新增「ComfyUI 节点选型」节 + 解开放问题;标注显存出入(官方 17GB vs 1038lab 表 8GB)待实测。本机 ComfyUI 已装 41 包无 JoyCaption。项目 制作规划.md/TASKS.md 同步具体包名与节点链

## [2026-07-02] query | 修正 JoyCaption ComfyUI 节点推荐 — 起因:用户指出 1038lab/ComfyUI-JoyCaption"最近不更新了"。gh api 核实:该包最后提交 2025-12-24、Registry 版本停在 2025-10-27 v2.0.2 未再发版、零 GitHub Release,确属陈旧。改查发现官方 fpgaminer/joycaption_comfyui 于 2026-02-25 打 tag v1.1.0(模型加载拆独立节点)且已上架 ComfyUI Registry(可 Manager 直装,纠正首版"均不在 registry"的错误判断)。读官方 nodes.py 源码确认其节点仅支持单图、无批量/存文本节点,搭配本机已装的 comfyui-easy-use(easy loadImagesForLoop + easy saveText)补齐管道,功能对等 1038lab 方案但主体维护更活跃。回填 [[JoyCaption]] 节点选型表(改判 + 4 候选对比 + 组合管道说明);项目 制作规划.md/TASKS.md 同步节点链

## [2026-07-02] query | 搭建 JoyCaption ComfyUI 批量打标工作流 — 用户确认已装官方 fpgaminer/joycaption_comfyui(v1.1.0)。用 get_node_info 拉取精确节点 schema(JJC_DownloadAndLoadJoyCaptionModel/JJC_JoyCaption/easy loadImagesForLoop/easy forLoopEnd/easy saveText/原生 StringConcatenate),排除了 easy joyCaption2/3API(BizyAIR 云端节点,带 apikey_override,非本地方案,已避开)。搭建管道:loadImagesForLoop→JoyCaption(Descriptive/long,extra options含光照/景别/构图/SFW分级,不用真名 extra option)→StringConcatenate(前置触发词)→saveText(.txt存图旁同名)。validate_workflow 结构校验通过,save_workflow 存入 ComfyUI 库为 joycaption_batch_dataset_tagging.json。未做真实端到端执行(项目尚无数据集图片,且首次运行要下载 ~17GB 模型,未经用户确认不擅自触发)。回填 制作规划.md §二(工作流节点链说明)+ 新增数据集目录约定 datasets\<角色>\;TASKS.md S3 勾选前两项、去重复行

## [2026-07-02] query | JoyCaption 批量打标工作流调试:easy-use forLoop 循环不生效 — 用户实跑发现只处理了 1 张图。get_logs 确认模型加载/单张打标/存文件均正常,循环未触发第二轮。读 comfyui-easy-use 的 loadImagesForLoop/forLoopEnd/whileLoopEnd(py/nodes/image.py、logic.py)完整源码核实连线符合其 GraphBuilder+expand 动态子图设计,排查 lora-manager/lumi-batcher 两个第三方 execution hook 均确认透传非元凶,判断是该节点与本机 ComfyUI 版本的 expand 协议不兼容,放弃图内循环。改用外部脚本 ai-视觉/blue-seventeen-minutes/scripts/batch_caption.py 逐张提交独立 prompt(loadImagesForLoop start_index=i/limit=1,不接 forLoopEnd,无需递归),在「高桥未来」16 张真实素材验证通过(14 生成+2 跳过+0 失败,caption 含触发词/自然语言/光照/景别/构图/SFW 分级均正确)。回填 [[JoyCaption]] 新增循环不可靠的 callout + 通用建议(ComfyUI 图内循环节点异常时换外部脚本逐个提交 /prompt);制作规划.md/TASKS.md 同步脚本用法 + 订正数据集实际目录(ComfyUI output/BlueSeventeenMinutes/资产库/<角色>,非之前占位的 datasets/<角色>)

## [2026-07-02] query | JoyCaption canned extra_option 锁不住瞳色/发色,实测发现+修复 — 用户看 016.txt 发现眼睛被描述,担心训练 LoRA 后眼睛会变样。核实:剧本 canon 未来是琥珀色瞳,但已生成的 16 个标注里 9 个提到眼睛颜色且互相矛盾(brown/green)。原用 JJC_JoyCaption 节点的 canned extra_option("Do NOT include info about people/characters that cannot be changed...")测试证实不够硬,同一张图重测依然写"soft brown"。改用 JJC_JoyCaption_Custom 节点自定义 system_prompt/user_query,写入"禁用词+重复强调+具体反例"的强指令,重测后瞳色/发色描述完全消失;16 张全量重跑复测(--overwrite)确认无一处颜色词泄漏,只保留鞋子颜色(brown loafers,剧本设定本身如此,非泄漏)与发型样式描述。回填 [[JoyCaption]] 新增 canned-option 不可靠的 callout + 通用结论(精确锁定用 custom prompt,不用罐头选项);[[LoRA打标]] 关联条目补一句;制作规划.md §二 记录修复过程 + 提醒蓝/司需按各自锁定表定制 --user-query;TASKS.md 同步

## [2026-07-02] ingest | z-image LoRA 打标 agent prompt — 信源 sources/agent角色/z-image-lora通用agent打标提示词.md(用户已放入,未再归档)。用户拍板:立「agent 角色」枢纽 + 专角色概念页,基调「带基线+标注」。新建 [[agent角色]](overview,MOC,后续多角色留位)、[[z-image-agent打标工程师]](concept,source 指向该 prompt;页名后按用户改为 z-image- 前缀,留 agent打标工程师 别名);逐子主题矛盾扫描 A 强化(黄金法则硬操作化,连 [[JoyCaption]] 需强指令实测)/B 表面(中文打标=按底模分语言分支,z-image 原生中文,不撞 Flux)/C 无实证(固定字段顺序)/D 一致(触发词领头)/E 真张力(示例真名 diaochan vs 罕见 token 基线,已标注)/F 存疑(固定画质词会被烘焙),B/E/F 落 callout。交叉更新 [[LoRA打标]](新增「打标风格按底模分流(含语言)」表 + z-image 中文分支 + 关联链)、[[角色LoRA数据集构建]] §6(黄金法则/触发词两处补链 + 真名警示)。index 概览/概念各加一页。待写 [[Z-Image]]

## [2026-07-02] ingest | Flux 版打标 agent(生成新源 + ingest) — 用户要求参考 z-image 版、按 FLUX 规范生成新 prompt 并建并列角色页。用 flux-best-practices 技能(core-principles/t2i-prompting/negative-prompt-alternatives)+ 库内 Flux 结论,生成 sources/agent角色/flux通用agent打标提示词.md(中文指令、英文 NL 输出);新建 [[flux-agent打标工程师]](concept)。相对 z-image 版按底模分流改造:纯英文自然语言 prose(非中文短 tag)、软结构 front-load(非固定字段序)、罕见 token 触发词(修 z-image 真名 E 分歧)、不加恒定画质样板(修 F 隐患)、显式禁负面词;黄金法则/身份特征硬禁相同。与库内 Flux 页全面对齐无冲突;Flux.1/2 与打标风格无关(共用 T5)已 callout。交叉更新 [[agent角色]] 枢纽(加 Flux 行)、[[z-image-agent打标工程师]](补姊妹链)、[[LoRA打标]](Flux 行补角色页链)、index 概念区。附带修正:文件夹 agent提示词→agent角色 改名导致的 6 处失效 source 路径(z-image 页 frontmatter / index×2 / overview×3 / log)一并订正

## [2026-07-03] query | Flux.1 LoRA 训练尺寸 — 用户问「flux1 lora训练尺寸」。检索命中 [[Flux-LoRA-ai-toolkit训练实战]](resolution [512,768,1024] 多分辨率自动分桶,24GB 基线)+ [[角色LoRA数据集构建]] §预处理(Flux/SDXL 素材按 1024、原图≥1024 不可放大、边 64 倍数或开 ARB 分桶)。两页已充分覆盖,直接检索作答,无需回填新页;附带提醒 Flux.1 vs Flux.2 参数不互通。

## [2026-07-04] query | m1r41 LoRA ai-toolkit 参数体检 + 8 项优化落地 — 用户在 ai-toolkit Web UI 配好主角高桥未来 m1r41 的 Flux.1-dev 角色 LoRA job(未开跑),要对照 wiki 基线找需优化参数并直接改 job。经 REST API(GET/POST /api/jobs)读回 job_config 发现 UI 新建 job 默认值偏离 [[Flux-LoRA-ai-toolkit训练实战]] 基线,直接改 8 项并 API 回读校验全部生效:rank 32→16(用户拍板纯身份路线,画风交独立风格 LoRA)、steps 3000→2000、max_step_saves_to_keep 4→8(避免删掉 500–1000 步最佳档)、dataset.cache_latents_to_disk→true、use_ema→true、sample.samples 通用占位词→含 m1r41 的集内锚+集外场景泛化+换装(兼 S8 便服)3 条、guidance 4→3.5、sample 尺寸 1024²→832×1216。数据集发现记进项目 高桥未来第一次训练发现的问题-20260704.md(⚠️16 张全戴随身听违反角色卡「有/无两版」铁律→S8 靠 prompt/inpaint 摘、全白背景+单套装、打标纪律正确)。新建 concept 页 [[ai-toolkit-WebUI默认值陷阱]](UI 默认≠基线检查清单 + keep×steps 删档陷阱 + 改 job 的 API 备忘),交叉更新 [[Flux-LoRA-ai-toolkit训练实战]] 关联区加链,index 概念区登记。

## [2026-07-07] query | 本机开发环境（资讯流水线 Qwen-Image 选型语境） — 命中 [[本机开发环境]]：RTX 5090 D v2 24GB（Blackwell CC 12.0）修正了对话中按 3090/4090 假设的 fp8 硬件加速条件判断；64GB 内存证实 bf16 原版（权重 ~61.6GB）不可行。回填新页 [[Qwen-Image-24G部署选型]]（comparison：bf16/fp8/Q8/Q6_K/Q4_K_M/Lightning 选型，用户拍板推荐档从 Q4_K_M 改为 Q6_K 全驻留甜点位，体积 2026-07-07 自 HF 核实），交叉链入 [[本机开发环境]] 关联区，index 对比区登记。

## [2026-07-09] query | 本机开发环境(抖音科普账号方案语境) + 回填两页 — 用户要对标「小白debug」的抖音 AI 科普号实现方案。命中 [[本机开发环境]](5090D 24GB/Node v24/Claude Max 无 API key→LLM 环节走会话)、[[剧本到AI漫剧生产流水线]](看板娘 LoRA 资产链直接复用)。逐帧分析用户录屏(《保姆级Claude Code速学教程》4:19 段)得成片形态结论。回填 [[对标账号-小白debug拆解]](entity,stable,两帧截图入 raw/assets/)与 [[抖音科普视频生产流水线]](synthesis,draft,逐句TTS免对齐/Remotion数据驱动/封面叠字三个关键机制;待 E001 验证转 stable);index 实体/综合区各登记。项目实例落 D:\project\ai-vision\ai-technology-sharing(Remotion 引擎/SOP/选题库/TTS 脚本),不入 wiki。发现:GPT-SoVITS Blackwell 卡须用 -nvidia50 整合包,已记入综合页。

## [2026-07-09] query | 动画库对比+抖音横竖屏 — 两源(sources/claude/ 剪藏,2026-07-09)有源无页,回填 [[动画库选型对比]](comparison,stable)+[[抖音横竖屏选择]](concept,stable);发现连接:小白debug横屏长视频=「深度知识讲解」例外分支非矛盾、Remotion 公司许可费补入 [[抖音科普视频生产流水线]] 开放问题(与 Flux.1-dev 非商用同属变现前合规项)并加关联链;index 对比/概念区各登记一页

## [2026-07-17] query+回填 | [[Agent编排模式谱系]] + [[内容线Agent编排评估]] — 起因:ai-news 仓 grilling 定案(模式粒度/双判据/拆两页/候选非约束)。定向核查 Anthropic 多agent系统·Cognition Don't-Build-Multi-Agents(含 2026 single-writer+advisory 收敛)·Manus 上下文工程·OpenAI handoffs·LangGraph 后落两页(均 synthesis,stable);关键发现:内容线三层改动权与 Cognition 2026 收敛结论同构且固化不晚于业界。content-factory docs/orchestration.md 挂指针指向评估页;index 综合区登记两页
