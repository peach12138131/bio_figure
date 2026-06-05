# data/1 绘图说明

本目录包含 4 个原始 Excel 数据文件，对应 4 组实验条件：

| 文件 | condition | substrate | 标题 |
|---|---|---|---|
| `6.3 fig1 b 10.xlsx` | B | 10 | B-10 |
| `6.3 fig1 b 18.xlsx` | B | 18 | B-18 |
| `6.3 fig1 p 10.xlsx` | P | 10 | P-10 |
| `6.3 fig1 p 18.xlsx` | P | 18 | P-18 |

最终产出 4 张散点图，位于 `process/1/result/`，每张图同时输出 4 种格式：
`fig1_{b10|b18|p10|p18}.{svg, pdf, png, tiff}`。

---

## 一、数据结构

每个 xlsx 文件内部含 **4 组数据块（block）**，每个 block 是 1 种 block 菌株与 D1/D6/D7 共培养的体系：

- block 顺序：`L43` → `L1091` → `L1253` → `L2182`
- 每个 block 含 5 个系列：`D1`, `D6`, `D7`, `block 菌株`(L43/L1091/L1253/L2182 之一), `Total`
- 时间点：`0h, 6h, 24h, 48h, 72h`
- 每个时间点 × 每个系列 = **3 个重复（replicate）**

每个文件共 4 × 5 × 5 × 3 = **300 行**，4 个文件共 **1200 行** 长表数据。

---

## 二、处理流程

### 1. 数据清洗：`process/1/1_dataprocess.py`

将 4 个 xlsx 文件解析为统一的长表 `process/1/result/processed.csv`，字段：

```
condition, substrate, block, strain, time_h, replicate, value
```

- `condition` ∈ {B, P}
- `substrate` ∈ {10, 18}
- `block` ∈ {L43, L1091, L1253, L2182}
- `strain` ∈ {D1, D6, D7, L43, L1091, L1253, L2182, Total}
- `time_h` ∈ {0, 6, 24, 48, 72}
- `replicate` ∈ {1, 2, 3}
- `value`：Log CFU/mL

### 2. 菌株名称：`process/1/strain_names.py`

集中维护菌株简写 ↔ 全名映射。图例显示为 `Abbrev(Full Name)` 格式。
若全名仍为占位（== 简写本身，如 `L43:'L43'`），`legend_label()` 会自动折叠为单一简写 `L43`，避免出现 `L43(L43)` 之类的冗余。
若需补全 D7 / L1091 / L1253 / L2182 的真实学名，仅改此文件，图例会自动恢复 `Abbrev(Full Name)` 形式。
配色不再来自这里，已在 `1_plt.py` / `analyze.py` 中按 Nature 风格重新规划（见下）。

### 3. 绘图：`process/1/1_plt.py`（Nature 风格）

每个文件输出一张图，规格：

#### 布局
- `subplots(4, 1, sharex=True, sharey=True)`，4 个子图紧密贴合（`hspace=0`），共享 X、Y 轴
- 子图顺序自上而下：`L43 → L1091 → L1253 → L2182`
- 画布尺寸 `7.2 × 8.0 inch`（≈ Nature 双栏宽 183 mm × 203 mm）

#### 标注
- **Panel letters**：子图左上外侧加 Nature 风格 `a / b / c / d`（10 pt 粗体）
- **子图内标题**：左上内侧 `D1-D6-D7-{block}`（7.5 pt 粗体，深灰）
- **总标题**：`B-10` / `B-18` / `P-10` / `P-18`（9 pt 粗体，居中偏左）
- **Y 轴**：`Log CFU/mL`，`fig.supylabel` 居中只显示一次
- **X 轴**：仅最下方子图显示 `Time` 与刻度标签 `0h, 6h, 24h, 48h, 72h`
- **底部脚注**：`n = 3 biological replicates per time point. Source data: processed.csv.`

#### 数据编码
- **每个子图 5 个系列**散点：`D1`, `D6`, `D7`, `block 菌株`, `Total`（纯散点，无连线）
- **散点 dodge**：5 个系列在每个时间点用横向偏移 `-1.0 / -0.5 / 0 / +0.5 / +1.0` 区分，每点 3 个重复叠加
- **散点样式**：`s=24, alpha=0.95`，白色描边 `0.3 pt`（保留高密度叠加时的可读性）
- **X 轴**：按真实时间比例（0/6/24/48/72），`xlim=(-3, 75)`
- **Y 轴**：`ylim=(-0.4, 10)`、`yticks=[0,2,4,6,8,10]`、虚线网格（0.4 pt, alpha 0.35）

#### 配色（家族化语义编码）
| 系列 | 颜色 | 角色 |
|---|---|---|
| `D1` | `#0F4D92` 深蓝 | 基线菌株 1 |
| `D6` | `#3775BA` 中蓝 | 基线菌株 2 |
| `D7` | `#7884B4` 浅蓝灰 | 基线菌株 3 |
| `L43 / L1091 / L1253 / L2182` | `#B64342` 暗红 | 互换的 block 菌株（暖色突出） |
| `Total` | `#767676` 中性灰 | 汇总值 |

> 设计意图：D1/D6/D7 用**冷色家族**视觉成组（"三个不变的基线菌"），block 菌株用**红色对比**突出"被替换的那一个"，Total 用**中性灰**降权处理。

#### 图例
- 集中在图右侧居中，8 个 entry（`D1, D6, D7, L43, L1091, L1253, L2182, Total`）
- 格式 `Abbrev(Full Name)`，无边框

#### 排版规范
- 字体 `Arial`（自动回退 Helvetica / DejaVu Sans）
- 正文 7 pt、刻度 6.5 pt、图例 6.5 pt
- 坐标轴线宽 0.6 pt，刻度长 2.5 pt
- `svg.fonttype='none'` + `pdf.fonttype=42`：导出的 SVG/PDF 文字保持可编辑、可搜索（Illustrator/Inkscape 可直接修改）

#### 输出
- `.svg`（首选，可编辑文字）
- `.pdf`（可编辑文字，矢量）
- `.png` 600 dpi（投稿与预览）
- `.tiff` 600 dpi（部分期刊要求）

---

## 三、运行

```cmd
cd D:\TAOXU\code\after_20260214\biofigure_0605\process\1
D:\taoxu_softwares\miniconda\2\envs\open_manus\python.exe 1_dataprocess.py
D:\taoxu_softwares\miniconda\2\envs\open_manus\python.exe 1_plt.py
```

第一步只在 xlsx 改动后需要重跑；调整图形参数只需要重跑第二步。

---

## 四、统计分析：`process/1/analyze.py`

针对每个 (condition, substrate) 切片，分析 4 株李斯特菌（L43 / L1091 / L1253 / L2182）的生长趋势一致性。

### 1. 模型设定（按 `data/1/analyze.docx` 要求）

- **time 因子化**：`C(time_h)`，不假设线性时间响应
- **全模型**：`value ~ C(strain) * C(time_h)`
- **简模型**：`value ~ C(strain) + C(time_h)`（仅主效应）
- **随机效应**：`groups = strain_replicate`（生物学重复随机截距）
- **似然比检验（LRT）**：取 strain × time 交互项 p 值
  `p = chi2.sf(2*(llf_full − llf_red), df=(k_s−1)*(k_t−1))`
- **小样本不做残差正态性 / 方差齐性检验**（n=3）

### 2. 关键决策：剔除 0h 后再做 LMM

研究意图明确为"**不关注初始绝对数量，只关注生长趋势**"。但 P-10 切片 0h 各菌起点跨度达 0.86 log（同批 OD 读数，3 重复 std=0），LMM 会把"起点不同"识别为 strain × time 交互。

因此 LMM 仅使用 **6h / 24h / 48h / 72h** 四个时间点（df = 3 × (k_s−1)）；**生长曲线图仍保留 0h** 的散点，供直观对照。

### 3. 两两 pairwise LMM

对每个切片，对 4 株菌做 C(4,2)=6 次 pairwise LMM，组成 4×4 对称 p 值矩阵（对角线 `—`）。

### 4. 输出

每个切片产出：

- `result/stats_{tag}.csv` —— 每 (strain, time_h) 的 `mean / sem / n`
- `result/pvals_{tag}.csv` —— 4×4 pairwise LMM 交互项 p 值矩阵
- `result/growth_{tag}.{svg,pdf,png,tiff}` —— 4 株菌 mean ± SEM 生长曲线
- `result/heatmap_{tag}.{svg,pdf,png,tiff}` —— p 值热图

`tag ∈ {b10, b18, p10, p18}`。

### 5. 视觉规范（Nature 风格）

**通用**：
- Arial 字体；SVG / PDF 文字可编辑（`svg.fonttype='none'`, `pdf.fonttype=42`）
- 标题左对齐 chip 风格（`fontweight='bold'`, `loc='left'`）

**4 株菌配色（Paul Tol bright，色盲安全）**：
| 系列 | 颜色 | marker |
|---|---|---|
| `L43`   | `#4477AA` 蓝   | `o` |
| `L1091` | `#EE6677` 粉红 | `s` |
| `L1253` | `#228833` 绿   | `^` |
| `L2182` | `#CCBB44` 黄褐 | `D` |

颜色 + 形状双重编码，黑白打印仍可区分。

**生长曲线**：
- `errorbar` 误差棒（细线：linewidth 1.8 / elinewidth 0.9 / capsize 2.5）
- 图例右侧外置、无边框
- 右下角注释 `n = 3 biological replicates`（斜体小灰字）
- 仅左/下脊线，y 轴虚线网格

**Heatmap**：
- `Reds_r` + `TwoSlopeNorm(vcenter=0.05)`：p<0.05 显眼深红，p>0.05 渐淡至白
- 单元格白色细线分隔，无黑色边框
- 单元格文字按背景亮度自动黑/白
- p<0.05 加粗，并标显著性星号：`*` (p<0.05) / `**` (p<0.01) / `***` (p<0.001)
- colorbar 标注 `0.05` 阈值线

### 6. 结果备忘（按真实数据）

| 切片 | p 总体 | 解读 |
|---|---|---|
| **B-10** | 0.02 – 0.25（多数 ≥ 0.05） | 4 株趋势基本一致 |
| **B-18** | L43↔L1091 ≈ 0.29；其余 ≈ 0 | L1253 / L2182 与其余显著不同 |
| **P-10** | L43↔L1253 = 0.78；其余 0–0.09 | L1091 与其余 3 株显著不同 |
| **P-18** | L43↔L1253 = 0.28；其余 ≈ 0 | L1091 / L2182 与其余显著不同 |

部分切片 p ≈ 0 是真实的：4 株李斯特菌在不同基质 / 温度下生长动力学确有分化（峰值时机不同、平台期高度不同），且 n=3 实验重复一致性极高（σ ≈ 0.05–0.20），即使 0.3 log 的差异也可跨过显著阈值。

---

## 五、备注

- 运行 `1_plt.py` 时 stderr 会出现大量 `CreateFile() Error: 5`，是 matplotlib 在 Windows 上检测系统字体目录时的权限提示，不影响输出。
- 修改菌株全名只需改 `process/1/strain_names.py`，无需重跑数据清洗。
- 如需调整子图间距、dodge 宽度、Y 轴范围、配色，仅改 `1_plt.py` 顶部常量（`COLOR`, `SERIES_OFFSETS`, `TIME_POINTS`）与 `subplots_adjust` 即可。
- 若期刊要求改字号或单栏宽度，调整 `mpl.rcParams.update({...})` 与 `figsize`。

---

## 五、统计分析（`process/1/analyze.py`）

### 目的
判定 4 株李斯特菌（**L43 / L1091 / L1253 / L2182**）在各自的 four-species biofilm 体系下的**整体生长趋势是否一致**。
仅关注趋势形状，不关注绝对值（4 株初始量本就不同）。

### 输入
- `process/1/result/processed.csv`（已有，无需重跑数据清洗）
- 过滤：`strain ∈ {L43, L1091, L1253, L2182}`
- 按 `(condition, substrate)` 切成 4 组：`B-10 / B-18 / P-10 / P-18`

### 输出（位于 `process/1/result/`）
每组 4 个产物：
| 文件 | 含义 |
|---|---|
| `stats_{tag}.csv`   | 每 (strain, time_h) 的 mean / sem / n=3 |
| `pvals_{tag}.csv`   | 4×4 对称 p 值矩阵（对角空） |
| `growth_{tag}.{svg,pdf,png,tiff}` | 4 株菌 mean ± SEM 折线图 |
| `heatmap_{tag}.{svg,pdf,png,tiff}` | 4×4 p 值 heatmap |

其中 `tag ∈ {b10, b18, p10, p18}`。

### 方法 —— 两两 pairwise LMM
对 4 株菌做 C(4,2)=6 次两两比较，每对独立拟合 LMM，取 strain×time 交互项 p 值填入 heatmap。

**模型设定**（按用户要求）：
- `time_h` 作**因子变量** `C(time_h)`，**不**作连续变量
- 全模型：`value ~ C(strain) * C(time_h)`，随机截距 `groups = strain×replicate`（uid）
- 简模型：`value ~ C(strain) + C(time_h)`
- 似然比检验（LRT）：`LR = 2*(llf_full - llf_red)`，自由度 `df = (k_strain-1)*(k_time-1) = 1×4 = 4`
- `p = chi2.sf(LR, df)`
- **不**检验残差正态性 / 方差齐性（数据量小）
- **只**输出交互项 p（菌株主效应、时间主效应 p 不输出）

**优化器**：`fit(reml=False, method='lbfgs')`，失败回退 `powell`。

### 生长曲线图
- 4 条 errorbar 折线，颜色：L43 蓝 / L1091 红 / L1253 绿 / L2182 橙
- X 轴按真实时间间隔（0/6/24/48/72），`xlim=(-3, 75)`
- 标题为 `B-10` / `B-18` / `P-10` / `P-18`（**不写 "mean+sem"**）
- 图例格式 `Abbrev(Full Name)`

### Heatmap
- `imshow` + `cmap='RdYlGn'`，`vmin=0, vmax=1`
- 单元注释保留 3 位小数，对角线 `—`
- colorbar 标题：`LMM strain×time interaction p`

### 运行

```cmd
cd D:\TAOXU\code\after_20260214\biofigure_0605\process\1
D:\taoxu_softwares\miniconda\2\envs\open_manus\python.exe analyze.py
```

仅依赖已有 `processed.csv`，无需重跑数据清洗。

### 依赖
- `statsmodels`（`mixedlm`）
- `scipy.stats.chi2`
- `numpy`, `pandas`, `matplotlib`

若 `statsmodels` 未装：
```cmd
D:\taoxu_softwares\miniconda\2\envs\open_manus\python.exe -m pip install statsmodels --only-binary :all: -i https://pypi.tuna.tsinghua.edu.cn/simple
```

### 备注
- `CreateFile() Error: 5` 与 1_plt.py 同源（Windows 字体探测权限），不影响输出。
- 小样本 LMM 可能出现 ConvergenceWarning，已在脚本顶部抑制。
- 多重比较未做 Bonferroni / Holm 校正（本轮按原始 p 输出，6 对比较若需校正可在 `pvals_*.csv` 上自行乘以 6 并 cap 到 1）。
