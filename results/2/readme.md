# data/2 绘图说明

本目录是 4 个 CSV 单培养（single-strain monoculture）CFU 数据，按 condition × substrate 切分：

| 文件 | condition | substrate | 含义 |
|---|---|---|---|
| `6.3 fig 1 single b 10.csv` | B | 10 | Biofilm, 10°C |
| `6.3 fig 1 single b 18.csv` | B | 18 | Biofilm, 18°C |
| `6.3 fig 1 single p 10.csv` | P | 10 | Planktonic, 10°C |
| `6.3 fig 1 single p 18.csv` | P | 18 | Planktonic, 18°C |

最终产出 **2 张热图**，位于 `process/2/result/`，每张图同时输出 4 种格式：
`fig2_{b|p}.{svg, pdf, png, tiff}`。

每张图含 2 个并排子图（**18°C 左 / 10°C 右**），布局与需求文档样例图一致。

---

## 一、数据结构

每个 CSV 第 1 行是列头，每个菌株全名重复 3 次表示 **3 个生物学重复**：

```
,, D1×3 重复, D6×3 重复, D7×3 重复, L43×3, L1091×3, L2182×3, L1253×3
```

第 1 列是时间标签（`0h`/`6h`/`24h`/`48h`/`72h`），第 2 列为空。

| 文件 | 时间点 | 行数（含表头）|
|---|---|---|
| B-10 / B-18 | 6h, 24h, 48h, 72h（**无 0h**） | 5 |
| P-10 / P-18 | 0h, 6h, 24h, 48h, 72h（**含 0h**） | 6 |

**7 株菌**（顺序与样例图行序一致）：

| 简写 | 全名 |
|---|---|
| D1 | Brochothrix thermosphacta |
| D6 | Pseudomonas fluorescens |
| D7 | Psychrobacter pulmonis |
| L43 | Listeria monocytogenes L43 |
| L1091 | Listeria monocytogenes L1091 |
| L2182 | Listeria monocytogenes L2182 |
| L1253 | Listeria monocytogenes L1253 |

> 与 data/1 不同：data/1 是 four-species biofilm 共培养（3 个 D + 1 个 L + Total），data/2 是 7 株菌**单培养**（无共培养、无 Total）。

---

## 二、处理流程

### 1. 数据清洗：`process/2/2_dataprocess.py`

将 4 个 CSV 解析为统一长表 `process/2/result/processed.csv`，字段：

```
condition, substrate, strain, time_h, replicate, value
```

- `condition` ∈ {B, P}
- `substrate` ∈ {10, 18}
- `strain` ∈ {D1, D6, D7, L43, L1091, L1253, L2182}
- `time_h` ∈ {0, 6, 24, 48, 72}（B 缺 0h）
- `replicate` ∈ {1, 2, 3}
- `value`：Log CFU/mL

**行数汇总**：378 行 = (B 7×4×3) ×2 + (P 7×5×3) ×2 = 168 + 210。

### 2. 菌株名称：`process/2/strain_names.py`

集中维护菌株简写 ↔ 全名映射。与 `process/1/strain_names.py` 同步（D7 全名 = `Psychrobacter pulmonis`，由本目录数据补全）。

### 3. 绘图：`process/2/2_plt.py`（Nature 风格热图）

#### 布局
- `subplots(1, 2)`，**18°C 左 / 10°C 右**
- 每子图：7 行（菌株）× 4 列（时间点）的 heatmap，`aspect='auto'`
- 画布尺寸 `11.2 × 5.0 inch`
- 单 colorbar：右侧共享（同 condition 的两个温度公用一把尺）

#### 单元值
- 3 个生物学重复**取均值**，保留 2 位小数（如 `7.36`）
- 颜色 = mean CFU（不是 delta），与样例图一致
- **跳过 0h**：P 数据虽含 0h，画图时只显示 6h–72h（与 B 对齐，便于 B/P 跨图对比）

#### 配色
- `cmap='Blues'`（序列色），浅 → 深 = CFU 低 → 高
- vmin / vmax **跨两个温度共享**，公平对比 18°C vs 10°C
- 单元格文字按 colormap 在该值的亮度自动黑/白（norm > 0.55 → 白字）
- 单元格之间用白色细线分隔（轻盈、视觉清爽）

#### 标注
- 大标题：`Biofilm (B)` / `Planktonic (P)`
- 子图标题：`18°C` / `10°C`
- X 轴：`6h / 24h / 48h / 72h`
- Y 轴：7 株菌简写（顺序：D1, D6, D7, L43, L1091, L2182, L1253）
- colorbar 标题：`Log CFU/mL`

#### 排版规范
- 字体 Arial，正文 13 pt、子图标题 15 pt、大标题 16 pt
- `svg.fonttype='none'` + `pdf.fonttype=42`：SVG/PDF 文字可编辑（Illustrator/Inkscape 直接改字）

#### 输出
- `.svg` / `.pdf`（矢量、文字可编辑）
- `.png` / `.tiff` 600 dpi（投稿与预览）

---

## 三、运行


第一步只在 CSV 改动后需要重跑；调整图形参数只需要重跑第二步。

---

## 四、备注

- `~$quirment.docx` 是 Word 打开 docx 时产生的临时锁文件，可忽略 / 关 Word 后删除。
- 修改菌株全名只需改 `process/2/strain_names.py`，无需重跑数据清洗。
- 如需把 0h 也画进 P 的热图：在 `2_plt.py` 的 `TIME_POINTS = [6, 24, 48, 72]` 前加 `0` 即可；但样例图未画 0h，此处保持一致。
- 如需把"增加 vs 减少"显式编码：把 `cmap='Blues'` 换成 `RdBu_r` 并将 `mat` 改为 `mat - mat[:, 0:1]`（相对 6h 的 delta），单元值同步改为差值。当前版本忠实于样例图，用绝对值 + 序列色。
