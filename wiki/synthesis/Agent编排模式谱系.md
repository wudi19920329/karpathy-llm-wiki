---
type: synthesis
aliases: [agent编排, 多agent编排模式, agent orchestration patterns, 多agent选型指南]
tags: [agent, orchestration, multi-agent, context-engineering, llm]
status: stable
created: 2026-07-17
updated: 2026-07-17
source:
related: ["[[内容线Agent编排评估]]", "[[agent角色]]", "[[抖音科普视频生产流水线]]"]
---

# Agent 编排模式谱系

## 摘要 / Summary

多 agent 编排的设计空间由两条正交轴张成：**控制权在谁手里**（中心派发 / 移交 / 图 / 流水线 / 无中心）与**上下文怎么传**（全量 trace / 全稿产物 / 压缩摘要 / 结构化契约；经 prompt / 会话延续 / 共享 state / 文件系统）。本页按模式粒度梳理六大编排模式与上下文传递的四个维度，框架只作实例；含「先问要不要多 agent」的前提判断与跨项目选型指南。写作时（2026-07）对时效敏感主张做过定向核查。

> [!note] 使用场景
> 为新项目做 agent 架构选型时先读本页定坐标；本知识库首个实例对账见 [[内容线Agent编排评估]]。

## 前提：先问要不要多 agent

**单 agent + 好工具 + 好上下文工程是默认解**，多 agent 是有明确收益证据时才上的升级。两大阵营的立场与和解：

- **Anthropic（赞成派）**：多 agent 研究系统在广度型检索任务上比单 Claude Opus 4 agent 高 90.2%，核心收益是**上下文隔离**——每个 subagent 独占干净窗口深挖一个子方向，突破单窗口天花板。代价 ≈ 普通对话 **15 倍 token**。（[How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)，2025-06）
- **Cognition（反对派）**：「Don't Build Multi-Agents」——**行动携带隐含决策**，并行 subagent 各自做出冲突的隐含决策且互不知情（Flappy Bird 实验：一个 subagent 做出马里奥风背景，另一个做出不像游戏资产的鸟）；原则是 **share full agent traces，不要只传消息摘要**，最可靠的形态是单线程线性 agent。（[Don't Build Multi-Agents](https://cognition.com/blog/dont-build-multi-agents)，2025-06）
- **2026 收敛**：两派实为按任务形状分治——**读并行任务**（调研/检索/评测，子任务互不写同一产物）多 agent 收益大；**写耦合任务**（编码/创作单一产物）冲突成本高。Walden Yan 2026 年更新版原则：「多 agent 系统在**写操作保持单线程、其余 agent 贡献智能而非动作**时最有效」（single-writer + advisory 模式）。

判据一句话：**任务能塞进一个上下文窗口且不需要并行探索时，不要上多 agent。**

## 六大编排模式

| 模式                          | 控制权               | 上下文传递                    | 代表实现                                     | 适用              |
| --------------------------- | ----------------- | ------------------------ | ---------------------------------------- | --------------- |
| ① Orchestrator-workers 中心派发 | 常驻主脑分解/派发/汇总      | 派发 prompt 下行，压缩结果/落盘引用上行 | Anthropic Research、Claude Code subagents | 读并行探索、可分解任务     |
| ② Handoffs 控制权移交            | 无常驻中心，agent 间显式移交 | 会话历史随移交延续（可过滤）           | OpenAI Agents SDK（源自 Swarm）              | 对话路由（客服 triage） |
| ③ Graph/状态机                 | 显式有向图，条件边决定流转     | typed state 沿边流转         | LangGraph                                | 合规/可回放/长事务      |
| ④ Role-based crew / SOP 流水线 | 角色按工序接力           | 产物（artifact）为界面          | CrewAI、MetaGPT                           | 内容/软件生产线        |
| ⑤ Blackboard 共享工作区          | 去中心，介质协调          | 共享介质（文件系统/DB）读写          | Manus file-system-as-context             | 长任务外置记忆；常与①④复合  |
| ⑥ Event-driven / mesh       | 事件总线触发            | 消息/事件负载                  | 企业集成栈                                    | 大规模异构系统         |

### ① Orchestrator-workers（中心派发）

主脑 agent 分析任务→派发 worker（各自独立上下文窗口）→worker 不互相通信→主脑整合。**supervisor** 是其对话域变体（2026 年客服类产品的主导模式）。已知弱点：**主脑是上下文瓶颈**（要装下任务全貌+所有 worker 结果）；生产系统的缓解手段——worker 返回结构化压缩结果而非自由长文、rolling summary 控主脑历史、结果落盘只回传引用。Anthropic 早期教训：简单查询也 spawn 50 个 subagent——**派发深度必须与任务复杂度挂钩**。

### ② Handoffs（控制权移交）

「当前该谁说话」的问题：triage agent 判断意图后把**对话控制权**连同（可过滤的）历史移交给 specialist。轻量、上手最快；弱点是移交链本质线性、无原生条件分支与检查点，复杂工作流不如③。（[OpenAI: Orchestration and handoffs](https://developers.openai.com/api/docs/guides/agents/orchestration)）

### ③ Graph / 状态机

工作流本身建模为有向图：typed state、条件边、循环、**checkpointing 与 human-in-the-loop 是一等公民**（断点等人审再续跑）。「图即真相源」，适合合规、可回放、长事务场景；代价是工程重、改流程要改图。（[LangGraph](https://www.langchain.com/blog/context-engineering-for-agents)）

### ④ Role-based crew / SOP 流水线

把人类生产线的 SOP 搬进 agent：角色化（researcher→writer→reviewer）按工序接力，**中间产物文档化**为工序界面（MetaGPT 的核心主张）。适合有稳定工艺的内容/软件生产。隐含成本：**角色 prompt 是资产**，需要版本管理与 fork 对账（参 [[agent角色]]）。

### ⑤ Blackboard / 共享工作区

agent 不直接对话，经共享介质读写协作。现代形态是 **file-system-as-context**（Manus）：文件系统当"终极上下文"——无限、持久、agent 可直接操作；上下文窗口只放工作集，大对象落盘存引用。副产物：**产物即审计日志**。（[Context Engineering for AI Agents: Lessons from Building Manus](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus)，2025-07）

### ⑥ Event-driven / mesh

去中心事件总线，agent 订阅/发布事件。规模化企业集成才需要；可观测性与调试难度最高，小团队勿入。

## 上下文传递（正交于模式的第二轴）

### 传什么：保真度谱

```
全量 trace ——— 全稿产物 ——— 压缩摘要 ——— 结构化字段
(Cognition)    (SOP流水线)   (Anthropic研究)  (生产系统JSON)
← 写耦合任务往左（防隐含决策冲突）   读并行任务往右（防主脑窗口爆炸）→
```

摘要是**有损压缩**——砍掉的往往正是下游需要的隐含决策；广度检索例外（原始 trace 本来就冗余）。

### 怎么传：五通道

1. **派发 prompt 注入**——随任务下发，最直接；硬约束必须显式注入，勿假设 agent 会自己读到；
2. **会话历史延续**——handoffs/sessions 机制自动携带；
3. **共享 typed state**——graph 模式沿边流转；
4. **文件系统**——无限容量+持久+可审计，长任务标配；
5. **结构化契约**——上下行都约定 schema（下行硬参数清单、上行 JSON/定式文档），是防漂移的廉价校验点。

### 上下文经济学

- **KV-cache 命中率**是生产 agent 的第一成本指标（Manus：cached/uncached 差 **10 倍价**）→ prompt 前缀保持稳定（开头别放时间戳）、上下文 append-only；
- **Compaction**：接近窗口上限时摘要重开（Anthropic 实测 100 轮检索任务省 84% token）——但摘要即有损，关键决策先落盘再压缩；
- **Context rot / lost-in-the-middle**：长上下文中段信息衰减 → **recitation** 对策：把目标/清单反复重写到上下文尾部（Manus 的 todo.md 就是注意力操纵机制）。

### 隔离 vs 共享的根本张力

隔离防污染、给并行扩容（Anthropic 收益来源）；共享防隐含决策分歧（Cognition 事故来源）。工程折中 = **单写者 + 隔离读者**：并行的只读探索随便开，动产物的写路径永远单线程。

## 选型指南

| 任务形状 | 推荐模式 | 上下文策略 |
|---|---|---|
| 读并行探索（调研/评测/素材海选） | ① 中心派发 | worker 返回压缩/结构化结果，源材料落盘 |
| 单一产物多工序（内容/代码生产线） | ④ 流水线 + 单写者 + advisory 审校 | 全稿传递 + 产物落盘 + 硬参数契约 |
| 需硬审计/断点等人/可回放 | ③ 图状态机（或至少：落盘契约+人工门） | typed state / 文件即检查点 |
| 对话路由（多领域客服） | ② handoffs / supervisor | 会话延续 + 历史过滤 |
| 任务塞得进一个窗口 | **单 agent，别编排** | 上下文工程做好即可 |

通用律：

1. 模式可复合（中心派发 + 文件黑板 + 流水线是常见三合一）；
2. **上下文方式与模式正交但有亲和**——选完模式必须再显式选上下文策略，这一步跳过是多数多 agent 系统翻车的原因；
3. 派发深度与任务复杂度挂钩（分档/预算机制）；
4. 每个环节用**够用的最便宜模型**；
5. advisory（只出意见）比 executor（直接改）便宜且可拦截——存疑环节先用 advisory 试。

## 关联 / Connections

- 实例对账：[[内容线Agent编排评估]]——内容生产线体系在本谱系中的坐标与双判据评估
- 角色 prompt 资产管理：[[agent角色]]
- 单 agent 生产线先例：[[抖音科普视频生产流水线]]、[[剧本到AI漫剧生产流水线]]

## 来源 / Sources

- [Anthropic: How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)（2025-06）
- [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)（2025-09，compaction/结构化笔记/子agent隔离三技法）
- [Cognition: Don't Build Multi-Agents](https://cognition.com/blog/dont-build-multi-agents)（2025-06；2026 更新立场见 Walden Yan X 帖：single-writer + advisory）
- [Manus: Context Engineering for AI Agents](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus)（2025-07）
- [OpenAI: Orchestration and handoffs](https://developers.openai.com/api/docs/guides/agents/orchestration)、[Agents SDK](https://openai.github.io/openai-agents-python/)
- [LangChain: Context Engineering for Agents](https://www.langchain.com/blog/context-engineering-for-agents)
- 2026 业界综述（supervisor 主导客服域、主脑瓶颈缓解三手段）：[DevRev](https://devrev.ai/blog/ai-agent-orchestration)、[Beam](https://beam.ai/agentic-insights/multi-agent-orchestration-patterns-production)

## 开放问题 / Open questions

- 长横向任务（跨天/跨会话）的编排（durable execution）尚无收敛方案，graph 检查点 vs 文件黑板 vs 托管 session 三路并行演化中；
- KV-cache 经济学对**订阅制**（无按 token 计费）的适用度：成本不可见但延迟收益仍在，值得单独核算吗？（本机即订阅制，见 [[本机开发环境]]）
