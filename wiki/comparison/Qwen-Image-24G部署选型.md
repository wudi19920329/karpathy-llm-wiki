---
type: comparison
aliases: [Qwen-Image部署选型, Qwen-Image量化对比, Qwen-Image-GGUF选型]
tags: [qwen-image, gguf, fp8, 量化, comfyui, 显存, 部署, 生图]
status: draft
created: 2026-07-07
updated: 2026-07-07
source:
related: ["[[本机开发环境]]"]
---

# Qwen-Image-24G部署选型

## 摘要 / Summary

Qwen-Image（20B MMDiT 文生图，本地开源中唯一中文文字渲染准确的模型）在 [[本机开发环境]]（RTX 5090 D v2 24GB / 64GB DDR5）上的格式选型结论：**bf16 原版不可行；GGUF Q6_K 是 24G 卡"全驻留"甜点位（默认）；Q4_K_M 为需余量时的降级档；Q8/fp8 仅作画质上限对照；Lightning LoRA 是提速档**。体积数字 2026-07-07 自 HuggingFace 仓库页逐一核实。

## 各格式体积与可行性

| 格式 | 仓库 | 体积 | 24G 显存 / 64G 内存可行性 |
|---|---|---|---|
| bf16 原版 | `Qwen/Qwen-Image` | transformer 44.8GB（9 分片）+ text encoder 16.6GB + VAE 0.25GB ≈ **61.6GB** | ❌ 显存装不下须逐层 offload；权重总量逼近 64GB 内存 → pagefile/OOM，单图分钟到十几分钟级，仅适合偶尔验证画质上限 |
| fp8_e4m3fn | `Comfy-Org/Qwen-Image_ComfyUI` | transformer 20.4GB | ⚠️ 权重可驻留但激活值余量极小，无法与其他模型共存 |
| GGUF Q8_0 | `QuantStack/Qwen-Image-GGUF` | 21.8GB | ⚠️ 权重 + 激活值会顶破 24G，触发部分 offload 掉速；画质最接近 bf16 |
| GGUF Q6_K | 同上 | **16.8GB** | ✅ **推荐默认**：全驻留后仍余 ~7GB 给激活值，1024²–1328² 出图从容——不掉速前提下画质最高的一档，且明显优于 Q4 |
| GGUF Q4_K_M | 同上 | 13.1GB | ✅ 降级档：需要更多余量（更高分辨率批量 / 与其他负载共存）时用 |
| GGUF Q2_K | 同上 | 7.06GB | 兜底档，画质损失明显 |

text encoder（Qwen2.5-VL-7B）三档均在 Comfy-Org 仓库 `split_files/text_encoders/`：bf16 16.6GB / **fp8_scaled 9.38GB（推荐）** / nvfp4 6.11GB（需 Blackwell 硬件）。VAE `qwen_image_vae.safetensors` 254MB。text encoder 编码完提示词即被 ComfyUI 卸载，不与主模型抢显存。

## Q8 vs fp8（画质/速度取舍）

- **画质：Q8 略胜。** Q8_0 = 8-bit 整数 + 每 32 个权重共享缩放因子（实际 ~8.5 bit/参数），出图与 bf16 基准几乎逐像素一致；fp8 e4m3 仅 3 位尾数，有轻微细节漂移。中文小字渲染场景 Q 系列更稳。
- **速度：fp8 更快，且在本机成立。** [[本机开发环境]] 的 RTX 5090 D v2 是 Blackwell（CC 12.0），有原生 fp8（乃至 fp4）计算单元；GGUF 推理需逐层实时反量化，单步慢约 10–20%。（Ampere 卡如 3090 无 fp8 单元，该优势不存在——本页结论依赖显卡代际。）
- **GGUF 与非 GGUF 的本质区别**：GGUF 是 llama.cpp 生态的量化容器格式（Ollama 跑 LLM 用的同一格式，但 Ollama 不能跑扩散模型），safetensors 是裸权重。ComfyUI 加载 `.gguf` 需 city96 的 **ComfyUI-GGUF** 自定义节点（"Unet Loader (GGUF)"）。

## Lightning（步数蒸馏，与量化正交）

`lightx2v/Qwen-Image-Lightning`：知识蒸馏 LoRA，把原版 40–50 步采样压到 **8 步（另有 4 步版）**，且 `cfg=1` 免掉负提示词分支——端到端约 10× 提速；代价是细节丰富度/多样性略降。**量化压体积，蒸馏压步数，两者可叠加。** GGUF 上挂 LoRA 有实时反量化开销，但相对 50→8 步的收益可忽略。文件名带 "Lightning" 的 merged GGUF 是 LoRA 预合并版：免挂载但行为锁死（不能切回高质量模式）。

## 推荐组合（下载合计 ~26.4GB）

| 文件 | 大小 | ComfyUI 位置 |
|---|---|---|
| `Qwen_Image-Q6_K.gguf` | 16.8GB | `models/diffusion_models/` |
| `qwen_2.5_vl_7b_fp8_scaled.safetensors` | 9.38GB | `models/text_encoders/` |
| `qwen_image_vae.safetensors` | 254MB | `models/vae/` |

路线：Q6_K + 50 步先建中文渲染质量基线 → 挂 Lightning 8 步提速 → 画质上限对照才用 Q8/fp8；显存吃紧或需共存时降 Q4_K_M（13.1GB）。

## 关联 / Connections

- 硬件约束来源 → [[本机开发环境]]
- 应用场景（AI 资讯流水线的中文信息卡 + 小红书图文配图）→ 项目文档 `ai-technology-news-pipeline/AI科技资讯流水线.md`（项目内实现细节以项目文档为准）
- 同生态：[[ComfyUI-工作流前查清单方案]]（搭工作流前核对模型/节点可用性）

## 开放问题 / Open questions

- Q6_K 中文小字渲染实测质量与实测显存峰值（跑过后回填，并同步 [[本机开发环境]] 的"5090 D v2 跑大模型实测"开放问题）。
- Lightning 8 步对中文文字渲染的影响实测。
- `QuantStack/Qwen-Image-Edit-2509-GGUF`（图生图改图版）是否值得纳入流水线。
