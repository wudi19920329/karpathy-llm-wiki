---
type: synthesis
aliases: [ComfyUI清单查询方案, ComfyUI模型LoRA插件清单, 搭工作流前查清单]
tags: [comfyui, workflow, mcp, inventory]
status: stable
created: 2026-06-30
updated: 2026-06-30
source:
related: ["[[ComfyUI-角色多镜头一致性工作流]]", "[[ComfyUI-分割与精修]]"]
---

# ComfyUI 搭工作流前查清单方案

## 摘要 / Summary

搭建任何 [[ComfyUI-分割与精修|ComfyUI]] 工作流**之前**,先查环境清单(有哪些模型 / LoRA / 自定义节点),再动手写节点——不要凭空写节点类型或模型名。核心判断:**列「磁盘上有什么文件」≠ 列「ComfyUI 实际认得、能用什么」**,两者经常不一致(放错目录 / 格式不支持 / 节点没加载时就会差),搭工作流要的是**后者**。

> [!rule] 固定工作规则(用户拍板)
> 我给用户搭 ComfyUI 工作流时,先走「查清单」流程再写节点。① 与磁盘对不上(下了却没加载)当场告知,不默默用磁盘文件名硬塞进工作流。

## 要点 / Key points

### 三类清单「谁最全」要分开看

没有单一命令「全都最全」,真相源按类别分开:

| 你要的信息 | 最全的来源 | 为什么 |
|---|---|---|
| **模型 / LoRA(能不能用)** | HTTP API `GET /object_info` | 返回的是 ComfyUI **实际加载成功、能在节点里选**的枚举值(`lora_name`、`ckpt_name`、`vae_name`…)。放错目录 / 格式不支持的文件**不会**出现——这正是「可用性」真相 |
| **模型 / LoRA(磁盘有哪些文件)** | `comfy model list` / 直接看 `models/` 目录 | 列文件本身,含没被识别、放错位置的。查「下了但选不到」 |
| **插件 / 自定义节点** | ComfyUI-Manager 的 `cm-cli.py show installed` | 最全:包名、版本、git 来源、enabled/disabled、是否有更新。`comfy node show` 是它的封装 |

### 搭工作流的硬约束 = 有效性

节点 `class_type` 必须存在;loader 节点里的 `lora_name` / `ckpt_name` / `vae_name` 必须**精确匹配** ComfyUI 暴露的字符串,否则工作流报错跑不了。所以**以 `/object_info` 为权威**(决定能不能用、字符串怎么写)。

### 我搭工作流时执行的查清单流程

| 步骤 | 用哪个(MCP / 官方等价) | 拿什么 |
|---|---|---|
| ① 节点是否可用 + 可选值真相 | `get_node_info`(= `/object_info`) | 节点存在性、`lora_name`/`ckpt_name`/`vae_name` 等**精确枚举**——写进工作流的字符串源头 |
| ② 选哪个模型 / LoRA | `list_local_models`(= `comfy model list`) | 路径、大小等磁盘元数据,便于按用途挑 |
| ③ 需要的自定义节点在不在 | `list_installed_nodes` / `list_packs`(= `cm-cli show installed`) | 缺则先提示安装,不硬写不存在的节点 |

**原则:以 ① 为权威**;① 与 ② 对不上(下了却没加载)当场告知用户,不默默拿磁盘文件名硬塞。

### 官方推荐命令速查

- **comfy-cli**(官方 CLI,`pip install comfy-cli`):`comfy model list`(模型 / LoRA)、`comfy node show installed`(插件,代理给 ComfyUI-Manager)。
- **ComfyUI-Manager cm-cli**(插件管理源头):`python cm-cli.py show installed` / `simple-show all`。
- **HTTP API**(GUI 与上述工具的底层):`GET /object_info`(节点定义 + 模型枚举)、`GET /embeddings`、`GET /system_stats`。

## 关联 / Connections

- 相关:[[ComfyUI-角色多镜头一致性工作流]]、[[ComfyUI-分割与精修]]、[[SAM3-文本分割]]
- 来源:本次对话原创综合(无外部信源)

## 开放问题 / Open questions

- `list_local_models` 与 `/object_info` 的差集自动化对比(查「下了却用不了」)是否值得做成一步固定校验?
