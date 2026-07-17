---
type: synthesis
aliases: [内容线编排评估, orchestration评估, 内容工厂agent编排]
tags: [agent, orchestration, content-factory, ai-news, ai-tools, 评估]
status: stable
created: 2026-07-17
updated: 2026-07-17
source: D:\project\ai-llm\content-factory\docs\orchestration.md
related: ["[[Agent编排模式谱系]]", "[[agent角色]]"]
---

# 内容线 Agent 编排评估（双判据）

## 摘要 / Summary

对内容生产线编排体系（content-factory `docs/orchestration.md` + ai-news/ai-tools 域仓分工）做双判据评估：**谱系坐标**（对照 [[Agent编排模式谱系]]）+ **自家实证**（复盘数据/事故对账）。总判断：**主干与 2026 业界收敛方向高度一致**——体系坐标 = 中心派发（①）× SOP 流水线（④）× 文件黑板（⑤）三合一 + 人工门 HITL；其中三层改动权与 Cognition 2026 年的 single-writer + advisory 收敛结论几乎逐字重合，且本线固化时间（2026-07-14）不晚于业界共识成型。页尾挂复盘后再议候选，**非约束**。

> [!warning] 置信度注记
> 自家实证的样本量极小（全 agent 流程完整跑通 n=1：L2 首条 what-is-ai-vision；L1 复线复盘未跑）。所有「合理」判断是**方向一致性判断**，不是统计显著结论；以后每次复盘都应回来对账。评估基准日 2026-07-17。

## 被评体系速写

主会话只做编排 dispatch / 整合产出 / 把人工门；环节工作派发 10 个 agent（8 软链 + 2 中文 fork，见 [[agent角色]] 同类机制），机械活主会话跑技能。上下文四通道：全稿 prompt 传递、硬参数显式注入、`data/<池>/<id>/` 产物落盘链（agent 间不直接通信）、产物头部回述校验。三层改动权（生产出初稿/审校只出意见/裁决归主会话+用户）+ 三道人工门 + 快讯/深度分档 + 成本纪律（登记 spawn 次数）。

## 逐项对账

| 机制 | 谱系坐标 | 自家证据 | 判断 |
|---|---|---|---|
| 主会话中心派发，agent 间不通信 | 模式①正统；主脑瓶颈是已知弱点 | L2 首条 ≈9 次派发跑通，无 agent 间冲突 | ✅ 合理；瓶颈见候选1 |
| 全稿传递（勿只给摘要） | Cognition「share full traces」派；写耦合任务的正确端 | L1 试点「上下文碎裂是质量杀手」+ L2 复盘再证（审校质量依赖全文对照） | ✅ 合理；成本未量化见候选2 |
| 硬参数回述 + 头部复述校验 | recitation（Manus todo.md 同族）+ 结构化契约 | 竖屏节拍表事故 → 固化后未再犯 | ✅ 合理，且属自创的低成本校验点 |
| 产物落盘契约（产物即审计日志） | 模式⑤ file-system-as-context，业界收敛方向 | 「复盘之所以能做，全靠这套产物留痕」；回写纪律源自片尾改版脱节事故 | ✅ 合理 |
| 三层改动权（审校 advisory 不 executor） | **撞正 Cognition 2026 收敛**：writes single-threaded, agents contribute intelligence not actions | 17 条审校建议：14采纳/1折中/1拒绝/1半修，错误冲突率≈12%——advisory 拦截成本≈0，executor 会把错焊进成片 | ✅ 全体系最亮机制，实证与谱系双撑 |
| 三道人工门 | HITL checkpoint（模式③把它做成一等公民）；本体系用「落盘产物+停等」实现，无框架也成立 | QC 门拦下发布事故级问题（响度阻断、srt 货不对板）；章节错位实发事故补上「发布后回读」 | ✅ 合理 |
| 双审校不合并 | 多视角 advisory 并行（读并行，无写冲突） | 9 条建议仅 1 撞车，视角互补 | ✅ 合理 |
| 软链+fork 花名册（manifest 唯一真相源） | 模式④的角色 prompt 资产管理，业界无成熟对应物（prompt registry 尚未收敛） | fork 基线已登记（上游 134b4d0），对账成本随上游漂移增长 | ✅ 合理；fork 件增多时见候选4 |
| 快讯/深度分档 | 派发深度与复杂度挂钩（Anthropic「简单查询 spawn 50 subagent」的反面） | 分档本身未经复盘验证（L1 自有分工待验） | ⚠️ 方向对，待实证 |
| 成本纪律（登记 spawn 数） | 研究预算机制（agent 数/工具数/深度显式受控）的朴素版 | 首条基线 ≈9 次派发已登记 | ✅ 合理起点；见候选2/3 |

### 展开：为什么「全稿传递」与业界主流缓解手段相反却仍判合理

2026 生产系统的主流做法是 worker 返回**结构化压缩结果**、主脑用 rolling summary——那是**读并行**场景的解法（防主脑窗口爆炸）。内容线是**写耦合**场景：整条产物链（script→storyboard→srt→成片）共享同一组隐含创作决策，摘要砍掉的恰是这些决策——Cognition 的 Flappy Bird 事故就是这么来的。L1/L2 两次独立撞到同一教训（碎裂 = 质量杀手），方向判断成立；未解的是成本面（候选2）。

### 展开：三层改动权的谱系意义

业界从「Don't Build Multi-Agents」（2025-06）走到「single-writer + advisory 可行」（2026）用了约一年；本体系 2026-07-14 由 L2 首条复盘独立固化出同构规则（含机械修复例外：机器可复测的改动可直接干），且带着业界没有的量化注脚（12% 建议错误率 → advisory 的拦截价值）。这条是评估里唯一敢下「高度合理」的判断。

## 复盘后再议候选（非约束，等 L1 复线首条复盘裁决）

1. **主会话上下文膨胀无显式治理**——全稿传递+多产物整合都压在主会话窗口上，长会话触发 compaction 时裁决历史有被有损压缩的风险。已有部分缓解（裁决写进产物修订记录=落盘）；候选动作：把「主会话凭文件回读、不凭窗口记忆」升为明文纪律。
2. **全稿传递的 token 成本无量化口径**——复盘登记时把「spawn 次数」扩成「spawn 次数 + 派发 prompt 规模」，才能判断快讯档是否值得试「结构化摘要+原文落盘引用」的混合传递（风险对冲：碎裂教训在先，试点须带质量对照）。
3. **spawn 冷启动 vs 会话续用**——同条内容同 agent 多轮修订（如 Content Creator 裁决后返工）目前每轮重 spawn 重建上下文；候选：续用既有 agent 会话（保留其上下文）省冷启动，代价是引入上下文漂移风险。等修订轮次数据再议。
4. **fork 对账成本的天花板**——fork 件仅 2 个时手动 diff 对账可行；若 fork 件增多，候选「fork 即分叉、放弃上游对账」（承认 prompt 资产本地化）。
5. **审校意见文档加机器可读判决头**——review-*.md 已是定式文档，加 verdict 字段可让采纳率统计自动化。低优先级，纯锦上添花。

## 关联 / Connections

- 谱系与选型总图：[[Agent编排模式谱系]]
- 角色 prompt 资产先例：[[agent角色]]
- 被评对象正本：content-factory `docs/orchestration.md`（域仓映射见 ai-news / ai-tools 各自 CLAUDE.md）

## 来源 / Sources

- 自家：content-factory `docs/orchestration.md`（2026-07-14 L2 首条复盘固化）、ai-news `.claude/agents/README.md`、两仓 CLAUDE.md、复盘记忆（首次数据复盘 2026-07-15）
- 业界坐标引用一律见 [[Agent编排模式谱系]] 来源区（不重复列）

## 开放问题 / Open questions

- L1 复线首条复盘后回来对账：分档实证、修订轮次数据、token 口径——三项都直接喂给候选清单的裁决；
- n 增大后 12% 审校错误率是否稳定？若显著升高，advisory 模式的「拦截成本≈0」前提要复核（人工裁决本身有注意力成本）。
