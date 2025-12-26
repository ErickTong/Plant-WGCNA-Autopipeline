
# Plant WGCNA Ultimate Pipeline (v5 Enhanced)

本仓库提供一个面向植物转录组/多组学数据的 **WGCNA 自动化分析 Pipeline**

---

## 📥 输入参数与输入文件说明

本 pipeline 需要两类输入：**表达矩阵**（datExpr）与 **表型/分组矩阵**（datTraits）。  
建议所有文件使用 **UTF-8 编码**，并以 `tsv/csv` 格式保存。

---

### 1）表达矩阵（Expression matrix）

✅ 支持两种常见格式：

#### A. 样本在列（genes × samples）
- 行：基因（GeneID）
- 列：样本（SampleID）
- 示例：

| GeneID | S1 | S2 | S3 |
|-------|----|----|----|
| GeneA | 10 | 12 | 8  |
| GeneB | 3  | 4  | 2  |

#### B. 样本在行（samples × genes）
- 行：样本（SampleID）
- 列：基因（GeneID）
- 示例：

| SampleID | GeneA | GeneB |
|---------|-------|-------|
| S1      | 10    | 3     |
| S2      | 12    | 4     |
| S3      | 8     | 2     |

📌 要求：
- 基因/样本 ID 必须唯一
- 不建议含缺失值；如存在缺失值，建议先填补或过滤
- 推荐提前做表达量过滤（如 TPM > 1 in ≥ 50% samples）

---

### 2）表型矩阵（Traits / phenotype matrix）

- 行：样本（SampleID）
- 列：性状/分组信息（Trait1, Trait2, ...）
- 示例：

| SampleID | Treatment | Height | Sugar |
|---------|-----------|--------|-------|
| S1      | CK        | 10.2   | 3.1   |
| S2      | CK        | 11.0   | 3.5   |
| S3      | Salt      | 8.9    | 2.7   |

📌 要求：
- `SampleID` 必须与表达矩阵中的样本名一致
- 性状可以是连续数值（推荐）或分类变量  
  - 分类变量建议转换成数值或 dummy variables（如 CK=0, Salt=1）

---

### 3）核心参数（常用可调）

> 若你的脚本中使用 config 或在脚本顶部定义参数，请根据实际变量名修改。

| 参数 | 说明 | 建议 |
|------|------|------|
| `power` | soft-threshold (β) | 使用 pickSoftThreshold 选取，使 signed R² ≥ 0.85 |
| `networkType` | 网络类型 | `signed`（常用）或 `unsigned` |
| `minModuleSize` | 最小模块基因数 | 植物常用 30–50 |
| `mergeCutHeight` | 模块合并阈值 | 0.25（常用） |
| `corType` | 相关性计算方法 | `pearson`（默认）或 `bicor`（鲁棒） |
| `TOMType` | TOM 类型 | 通常与 networkType 一致 |
| `maxBlockSize` | blockwiseModules 分块大小 | 大基因集建议 5000–15000 |

---

### 4）输出文件前缀（prefix）

脚本通常会生成一批以项目名为前缀的结果文件，例如：
- `HHSS_WGCNA_01_SampleClustering.pdf`
- `HHSS_WGCNA_01_ModuleTrait_Cor.tsv`

建议你为每次分析设置唯一的 `prefix`，方便版本管理和复现。

---

## 🧪 最简运行示例

```r
# 示例：设置输入文件路径与核心参数
expr_file   <- "data/expr.tsv"
trait_file  <- "data/traits.tsv"

power <- 12
networkType <- "signed"
minModuleSize <- 30
mergeCutHeight <- 0.25

source("scripts/00WGCNA_Ultimate_Pipeline_v5_plus_export_plus_hubGS.R")
```
---


## 📤 输出文件介绍（Outputs）

脚本运行结束后，会在输出目录生成一组 **PDF 图形文件 + TSV 结果表格**，文件名前缀为你设置的 `{prefix}`（项目名/分析批次标识）。

---

### 1）质量控制与诊断图（PDF）

#### ✅ 1.1 样本聚类树
- `{prefix}_SampleClustering.pdf`

用途：用于检测样本是否存在明显离群（outlier），建议在建网前检查。

---

#### ✅ 1.2 Soft-threshold Power 评估图
- `{prefix}_SoftThreshold_Power.pdf`

包含两张经典评估图：
1) **Scale independence**：signed R² vs power（用于选择 power）
2) **Mean connectivity**：平均连接度 vs power  
图中会标记最终使用的 `power`，并通常以 signed R² ≥ 0.85 为参考标准。

---

### 2）输入数据与清洗后的矩阵导出（TSV）

#### ✅ 2.1 建网实际使用的表达矩阵
- `{prefix}_datExpr_cleaned.tsv`

说明：经过过滤/缺失处理后的表达矩阵（最终用于 WGCNA 建网的 datExpr）。

---

#### ✅ 2.2 参与分析的表型矩阵
- `{prefix}_datTraits_used.tsv`

说明：用于模块-性状关联分析的 traits 表型矩阵（样本为行）。

---

### 3）模块信息与模块特征向量（TSV）

#### ✅ 3.1 基因模块归属表
- `{prefix}_Gene_Module_Assignment.tsv`

字段说明：
- `GeneID`：基因 ID
- `ModuleColor`：该基因所属模块颜色（如 blue、turquoise）

用途：用于后续按模块提取基因集、功能富集或绘制模块网络。

---

#### ✅ 3.2 模块特征向量（Module Eigengenes）
- `{prefix}_ModuleEigengenes.tsv`

说明：每个样本在每个模块上的 eigengene（ME）值，用于下游关联分析与可视化。

---

#### ✅ 3.3 每个模块基因数统计
- `{prefix}_Module_GeneCounts.tsv`

说明：统计每个模块包含的基因数量（方便判断模块规模、筛选模块）。

---

### 4）模块-表型关联分析结果（TSV）

#### ✅ 4.1 模块-性状相关矩阵
- `{prefix}_ModuleTrait_Cor.tsv`

说明：模块 ME 与性状（traits）的相关系数矩阵。

---

#### ✅ 4.2 模块-性状 P 值矩阵
- `{prefix}_ModuleTrait_Pvalue.tsv`

说明：对应模块-性状相关的显著性 P 值。

---

#### ✅ 4.3 模块-性状 long format（便于 ggplot）
- `{prefix}_ModuleTrait_Cor_Pvalue.long.tsv`

字段通常包括：
- `Module`
- `Trait`
- `Correlation`
- `Pvalue`

用途：便于使用 ggplot2 做热图/气泡图/点图。

---

### 5）基因层级指标导出：kME / GS（TSV）

#### ✅ 5.1 模块成员度 kME 表（Gene × Module）
- `{prefix}_Gene_kME_Table.tsv`

说明：kME（Module Membership）衡量基因在模块中的代表性程度（类似“中心性”）。  
常用于筛选 hub genes。

通常字段包含：
- `kME<module>`
- `p_kME<module>`

---

#### ✅ 5.2 全性状 GS 矩阵（Gene × Trait）
- `{prefix}_GeneTrait_GS_allTraits.tsv`
- `{prefix}_GeneTrait_GS_allTraits_Pvalue.tsv`

说明：GS（Gene Significance）衡量基因表达与各性状的相关程度。  
可用于：
- 与 kME 联合筛选候选基因  
- 优先锁定与目标性状最相关的模块/基因

---

### 6）Hub genes 筛选结果（TSV）

#### ✅ 6.1 每模块 Top Hub genes（按 |kME| 排序）
- `{prefix}_HubGenes_Top30_by_kME.tsv`

说明：对每个模块按 `abs(kME)` 由高到低排序，输出 Top 30 的 hub genes（默认 Top30，可在脚本中调整）。

常用于：
- 后续功能验证（qPCR、突变体）
- 网络可视化（Cytoscape）
- 富集分析（GO/KEGG）

---

## 🔍 推荐分析流程（建议）
1. 查看 `{prefix}_SampleClustering.pdf` 确认无离群样本  
2. 查看 `{prefix}_SoftThreshold_Power.pdf` 确认 power 选择合理  
3. 结合 `{prefix}_ModuleTrait_Cor.tsv` 找显著模块  
4. 在显著模块中结合 kME 与 GS 筛选候选 hub genes  
5. 导出候选基因进行富集分析或实验验证  

---

## 🚀 使用方式

直接运行脚本即可（建议在 R >= 4.2，WGCNA 最新稳定版环境）。

```r
source("scripts/00WGCNA_Ultimate_Pipeline_v5_plus_export_plus_hubGS.R")
```

## 🧬 应用场景

* 植物转录组模块挖掘
* 表型/代谢物/环境因子关联分析
* Hub gene 筛选用于后续功能验证
