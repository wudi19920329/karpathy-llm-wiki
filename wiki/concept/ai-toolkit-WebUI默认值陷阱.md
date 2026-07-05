---
type: concept
aliases:
  - ai-toolkit UI 默认值
  - ai-toolkit 新建 job 默认值
  - m1r41训练配置实录
tags:
  - lora
  - flux
  - ai-toolkit
  - training
status: stable
created: 2026-07-04
updated: 2026-07-04
source:
related:
  - "[[Flux-LoRA-ai-toolkit训练实战]]"
  - "[[角色LoRA数据集构建]]"
  - "[[LoRA打标]]"
---

# ai-toolkit Web UI 默认值陷阱

## 摘要 / Summary

ai-toolkit **Web UI 新建 job 的默认值 ≠ [[Flux-LoRA-ai-toolkit训练实战|wiki 24GB 基线]]**——照默认直接开跑会踩坑。本页把 UI 默认值与基线的偏差、一个隐蔽的 **keep×steps** 删档陷阱、以及改 job 的 API 备忘，固化成「过 job 前检查清单」。首次实证：《蓝色十七分钟》主角 m1r41 首训体检（2026-07-04，详见项目 `高桥未来第一次训练发现的问题-20260704.md`）。

## 要点 / Key points

### UI 默认值 vs wiki 基线（开跑前逐项核对）

| 项 | UI 默认 | 基线 / 建议 | 说明 |
|---|---|---|---|
| `network.linear/alpha` | **32 / 32** | 角色 **16 / 16** | 32 留给焊死二次元画风 / 风格 LoRA；纯身份角色 16 更抗过拟合 |
| `train.steps` | **3000** | **1000–2000** + 早停 | 少样本 3000 过拟合风险高 |
| `sample.samples` | 通用占位词（red hair / chess / bomb…） | **必须含触发词** + 1 集内 + 1 集外泛化 | Flux 靠「看样图选档」，占位词无触发词＝盲训 |
| `sample.guidance_scale` | **4** | **3.5**（二次元更柔） | 仅影响样图，不影响训练 |
| `dataset.cache_latents_to_disk` | **false** | **true** | wiki「省显存必开」，加速 |
| `ema_config.use_ema` | **false** | **true**（小数据集） | LoRA 权重 EMA 开销小，提稳定性 |

> 已是基线、无需改的默认值：`lr 1e-4`、`adamw8bit`、`flowmatch`、`qfloat8` 量化、`resolution [512,768,1024]`、`save_every 250`、`train_text_encoder false`。`conv/conv_alpha` 对 Flux transformer 基本 inert、无害。

### ⚠️ keep × steps 删档陷阱

`save.max_step_saves_to_keep` 只保留**最近 N 个** checkpoint。UI 默认 `keep=4` + `save_every=250` → 只留最后 **1000 步**的档。若把 `steps` 设很大（如 3000），而 Flux 少样本最佳档常在 **500–1000 步**，**最佳档会被自动删光、无法回退挑档**（表现为「训完只剩几个过拟合档可选」）。

→ 规则：**`keep ≥ steps / save_every`**（保留全程做 XYZ 横比），或把 `steps` 压到与早停窗口匹配。

### 改 job 的 API 备忘

- **读单个 job**：`GET /api/jobs?id=<uuid>`（**不是** `/api/jobs/<uuid>`——那条没有 GET 路由，会 404 到 Next.js SPA 壳）。
- **改 job**：`POST /api/jobs`，body `{id, name, gpu_ids, job_config}`；服务端对 `job_config` 做 `JSON.stringify` 后覆盖存进 `aitk_db.db`（Prisma）。`id` 在场即 update、缺省即 create。job 需处于 `stopped`。

## 关联 / Connections

- 参数基线全表、lr/alpha/步数/正则图分歧 → [[Flux-LoRA-ai-toolkit训练实战]]
- 数据集张数 / 景别配比 / 背景范式 → [[角色LoRA数据集构建]]
- 打标黄金法则（标可变 / 不标想绑定）→ [[LoRA打标]]
- 首次实证：项目 `blue-seventeen-minutes` 的 `高桥未来第一次训练发现的问题-20260704.md`

## 开放问题 / Open questions

- ai-toolkit UI 版本升级后默认值是否变动，需再核（本页基于 2026-07 版）。
- m1r41 首训最佳 checkpoint 落在第几步（训完回填，验证「500–1000 步」经验）。
