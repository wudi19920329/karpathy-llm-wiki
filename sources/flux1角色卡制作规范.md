# 角色卡制作 & 出图规范（SOP）

> 适用：ComfyUI + FLUX.1-dev 漫剧制作 ｜ 维护：以后新增角色卡 / 出立绘 / 做 LoRA 数据集时照此执行。
> 配套文件：`角色卡.md`（具体角色）。

---

## 0. 核心顺序（不可颠倒）

```
剧本 → ① 角色卡 → ② 立绘（全身定妆，基准真值）→ ③ 特写/半身/多表情/多角度 →（可选）④ LoRA 数据集
```

- **先立绘后特写**：立绘一次性钉死发型/瞳色/服装/配饰/身材比例，是整套图的「基准真值」。先做特写只锁脸，回头补全身必飘。
- **同模型同 LoRA 铁律**：特写/衍生图必须用**画立绘的同一底模 + 同一组 LoRA**渲染。换模型（Kontext/Redux 等）= 必然换画风（脸变写实、高光/反光变多、线条变硬）。

---

## 1. 角色卡结构模板（每个角色都要有）

```markdown
### 角色名（罗马音）— 定位

| 项目 | 设定 |
|------|------|
| 年龄/身高 | |
| 发型 | |
| 五官 | |
| 服装 | |
| 配饰 | |
| 性格/情绪 | |
| **视觉锚点** | ← 识别角色的关键道具，一致性最重要的部分 |
| **专属配色** | |

**立绘提示词（全身强化版）：** ```...[STYLE]```
> 比例 832×1216。
```

- **视觉锚点**是最关键字段：选 2–4 个独一无二的道具/特征（例：未来=蓝缎带+银随身听；蓝=纸鹤发夹+白裙赤脚；司=灰挑染+怀表+大耳机），**每张图都必须保持**。

---

## 2. 通用画风后缀 `[STYLE]`（所有角色共用，保画风统一）

```
anime film still, in the style of Makoto Shinkai and Naoko Yamada, Kyoto Animation aesthetic,
delicate detailed eyes, subtle facial expression, soft cinematic lighting, watercolor atmosphere,
film grain, highly detailed, masterpiece, best quality
```
> 换项目就换这一段，角色提示词正文不动 → 全员画风一致。

---

## 3. 立绘提示词写法（全身强化版公式）

```
[拉远框架] + [角色主体描述] + [服装配饰 + 鞋/脚] + [表情/姿势] + [拉远锚点] + [背景+光] + [STYLE]
```

**① 拉远框架（开头）：**
`full body shot, full-length view from head to toe, entire body visible,`

**② 拉远锚点（结尾，出全身的命门）：**
`standing upright on the ground, <shoes/bare feet> visible, small figure centered in frame, wide framing,`
> 「**脚/鞋 visible**」和「**head to toe**」「**small figure in frame**」是最强的三个拉远信号。

**③ 背景 + 光：**
`plain soft <颜色> background, gentle even soft lighting,`

**④ 删除拉近词：** 不要写 `portrait / close-up / detailed face` 等，会把镜头往脸上拽。

---

## 4. 出图参数（FLUX.1-dev）

| 参数 | 推荐值 |
|------|--------|
| 模型 | `flux1-dev.safetensors` + `loras/Makoto_Shinkai_style.safetensors`(强度 0.7–1.0) |
| VAE / CLIP | `vae/flux1/ae.safetensors` ＋ `clip_l` + `t5xxl_fp16`(或 fp8 省显存) |
| 比例（全身） | **832×1216** 或 768×1344（竖图，**别用方图**，方图必裁半身） |
| CFG | **1.0**（Flux 固定，**无需负面提示词**） |
| Flux guidance | 3.0–3.5 |
| 采样器/调度 | euler + simple/beta，**steps 20–30** |

---

## 5. 全身出不来怎么办（决策顺序）

1. **先改提示词 + 比例**（首选，零画风风险）：竖图 832×1216 + 第 3 节的拉远框架/锚点 + 删拉近词 → 90% 搞定。
2. **还不稳 → 加 OpenPose ControlNet**：
   - 模型 `controlnet/flux1/flux1-dev-controlnet-openpose.safetensors`
   - **strength 0.4–0.6 / end_percent ≈ 0.5**（前半控姿势、后半松手交还底模+LoRA）→ 结构锁住、画风基本不动。
   - ⚠️ 用 **OpenPose**，别用 Depth/Canny（会把结构纹理带进来，线条变硬 = 画风漂移）；strength 别拉满 1.0。
3. **做数据集时别用固定 pose**：同一骨架控所有图会得到一堆同姿势、降低多样性、易过拟合。数据集靠提示词让姿势自然变化。

---

## 6. 背景 & 光影规则（尤其做 LoRA 数据集）

**铁律：LoRA 学的是「数据集里所有图的最大公约数」。** 每张都一样的东西会被烤进角色；变化的才会被分离。

| 维度 | 做法 |
|------|------|
| 背景 | ❌ 别全用纯白/纯黑（会烤进角色、之后放场景互相打架）；❌ 别全复杂场景（稀释身份）。✅ **简单 + 多色 + 虚化**：浅灰/米白/淡蓝/暖橙轮换，少量(~20–30%)简单环境 |
| 背景打标 | ✅ **每张都给背景打标**（`simple white background` / `blurred seaside background`…）——打标=背景被剥离；不打标=默认烤进角色 |
| 光影 | ✅ 主体用**均匀柔和中性光**，让发色/肤色/服装固有色被学准；允许轻微角度变化助泛化 |
| 染色光 | ⚠️ **避免强烈染色光**（整脸罩蓝/橙）。本项目「蓝色世界」的群青染色若拿来训身份，会把蓝烤进固有色 → 现实场景里也发蓝。染色效果单独做风格 LoRA 或在 caption 写明 `ethereal blue glowing world` |
| 灰度图 | ❌ 彩色角色别用黑白图，会丢掉配色身份 |

---

## 7. 角色一致性手段

| 手段 | 用途 | 本项目模型 |
|------|------|-----------|
| 同模型+同LoRA | 画风/身份基础保证 | flux1-dev + Makoto_Shinkai_style |
| **PuLID** | 锁人脸 ID，多镜头同一张脸 | `pulid/flux1/pulid_flux_v0.9.1` + `EVA02_CLIP_L` + InsightFace |
| **InfiniteYou** | 身份生成（更强一致性） | `infinite_you/aes_stage2/...` + InsightFace |
| 低 denoise i2i | 从立绘裁切出像素级一致的特写 | denoise 0.25–0.45 |

---

## 8. LoRA 数据集配比（单角色）

| 维度 | 建议 |
|------|------|
| 张数 | 20–40 张精图（Flux 上质量 > 数量） |
| 景别 | 全身/半身 ~40%、**脸部特写 ~40%**、其余角度 ~20% |
| 角度 | 正面为主 + 少量 3/4 侧、侧面 |
| 表情 | **要多样**（身份稳即可，表情/嘴型本就该变，别修） |
| 背景/光 | 见第 6 节 |

---

## 9. 排错速查

| 症状 | 主因 | 解法 |
|------|------|------|
| 出不了全身 / 裁半身 | 方图 / 缺拉远锚点 / 带了拉近词 | 竖图 832×1216 + `head to toe / 脚visible / small figure in frame`，删 portrait |
| 画风漂移（变写实、高光多、线条硬） | 用了别的模型(Kontext/Redux) 或 ControlNet 强度过高/用了Depth/Canny | 回到同模型+同LoRA；ControlNet 改 OpenPose、strength≤0.6、end_percent≈0.5 |
| 特写糊 | 裁小脸放大=纯插值 | 源头出大图（首选）或 FaceDetailer，别死磕裁剪放大 |
| 脸不像同一人 | denoise 过高 / 没挂立绘的 LoRA | 降 denoise、确认 LoRA 链与强度一致 |
| 嘴型/表情变了 | denoise 0.4 重画高频 + 表情词在指挥 | 数据集不用管；要一致则 denoise 0.25–0.32 且删表情词 |
| LoRA 出图老带固定背景/光 | 数据集背景/光太单一、没打标 | 背景多色+虚化+逐张打标，光用均匀柔光 |

---

## 10. 本项目可用模型速查

- **出图**：`flux1-dev` + LoRA `Makoto_Shinkai_style`（画风）/`AntiBlur`（抗糊）
- **改图/重绘**：`flux1-fill-dev`（局部）、`flux1-kontext-dev`（指令编辑）
- **控制**：ControlNet `Union-Pro-2.0`（多合一）/`openpose`/`depth`
- **一致性**：PuLID flux v0.9.1、InfiniteYou
- **后处理**：FaceDetailer(`face_yolov8m.pt`)、GFPGAN/CodeFormer(人脸修复)、birefnet(抠图)
- **视频**：Wan2.2 I2V、LTX 2.3（立绘出图后转动态）
> 详见 `模型配置清单.md`（如已生成）。
