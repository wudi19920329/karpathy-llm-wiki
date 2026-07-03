---
type: overview
aliases:
  - agent 角色
  - agent 提示词
  - agent prompts
tags:
  - agent
  - prompt
  - moc
status: stub
created: 2026-07-02
updated: 2026-07-02
source:
related:
  - "[[z-image-agent打标工程师]]"
  - "[[flux-agent打标工程师]]"
---

# agent 角色 🎭

## 摘要 / Summary

**agent 角色**是一批可复用的 **system prompt**——每一份把 LLM 固化成某个工作流步骤上**有纪律的专才**(而非泛泛聊天机器人),用硬约束(必守规则、输出结构、禁用项)锁定它的行为。本页是这些角色 prompt 的**导航枢纽(MOC)**;原始 prompt 存于 `sources/agent角色/`,每份对应一个角色页。

> 这与本库 [[CLAUDE|schema]] 的信条同源:让 LLM 成为**有纪律的维护者**。区别在于——这里收录的是**别处工作流里用的角色 prompt**(打标、分镜、审校……),把它们当作可复用资产沉淀、互链、对比。

## 角色清单 / Roles

| 角色 | 用途 | 页面 |
|---|---|---|
| LoRA 打标工程师(z-image) | 按「绑定/可变」二分,给角色 LoRA 数据集生成标准化**中文结构化**标签(z-image 底模) | [[z-image-agent打标工程师]] |
| LoRA 打标工程师(Flux) | 同一法则的按底模分流镜像:输出**英文自然语言 prose**、罕见 token 触发词、灯光必写、无负面词(Flux/T5 底模) | [[flux-agent打标工程师]] |

> 待写:后续角色(分镜 / 审校 / 提示词扩写 等)加入 `sources/agent角色/` 后,在此登记并各建一页。

## 关联 / Connections

- 打标角色所依据的打标方法论 → [[LoRA打标]]、[[角色LoRA数据集构建]]
- 来源目录:`sources/agent角色/`

## 开放问题 / Open questions

- 多个角色积累后,是否需要一层「角色 prompt 写法规范」(触发词/禁用词/输出结构的通用模式)抽象成概念页。
