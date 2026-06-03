# Chem-Auto-Lab Skill — 化学实验数据自动处理

## 概述

对用户提供的化学实验室数据（Excel/CSV/TXT/仪器导出文件）进行自动化处理，覆盖**数据清洗 → 谱图解析 → 笔记结构化 → 报告生成 → 实验推荐**全流程。

### 适用场景

- 化学实验数据清洗、标准化、异常值处理
- FTIR / Raman / UV-Vis / HPLC / NMR 谱图解析
- 非结构化的实验记录笔记结构化
- 生成分析报告和可视化图表
- 推荐下一步实验方案

---

## 执行流程

```
用户输入文件 (Excel/CSV/TXT/谱图文件)
    │
    ▼
┌─────────────────────────────────────────┐
│          文件自动分类                    │
├──────────┬──────────┬───────────────────┤
│ 电子表格  │ 谱图文件  │ 文本笔记          │
│ .xlsx    │ .jdx/.spc│ .txt/.md/.log    │
│ .csv/.tsv│ xy.csv   │                   │
└────┬─────┴────┬─────┴───────┬───────────┘
     │          │              │
     ▼          ▼              ▼
┌─────────┐ ┌─────────┐ ┌──────────────┐
│Module 1 │ │Module 2 │ │ Module 3     │
│数据清洗  │ │谱图解析  │ │ 笔记结构化    │
│clean_   │ │parse_   │ │ structure_   │
│data.py  │ │spectrum │ │ notes.py     │
│         │ │.py      │ │              │
└────┬────┘ └────┬────┘ └──────┬───────┘
     │           │              │
     └───────┬───┴──────────────┘
             ▼
     ┌─────────────┐
     │ 合并结构化数据 │
     └──────┬──────┘
            ▼
     ┌─────────────┐   ┌──────────────┐
     │ Module 4    │──▶│  可视化图表    │
     │ 报告生成     │   │ visualize.py │
     │generate_    │   └──────────────┘
     │report.py    │
     └──────┬──────┘
            ▼
     ┌─────────────┐
     │ Module 5    │
     │ 实验推荐     │
     │ recommend.py│
     └─────────────┘
```

### 模块独立 vs 全流程 Pipeline

```
单模块调用: 用户只做谱图解析 → 仅触发 Module 2
全流程调用: 用户"帮我分析整个文件夹" → Module 1→2→3→4→5 顺序执行
```

---

## 使用方式

### 环境准备

```bash
pip install pandas numpy matplotlib scipy openpyxl seaborn jsonschema
```

### 单模块命令

所有脚本遵循统一的 CLI 接口：

| 模块 | 命令 | 示例 |
|------|------|------|
| **Module 1** 数据清洗 | `python scripts/clean_data.py --input <文件> --output <输出>` | `--imputation median --outlier iqr --normalize zscore` |
| **Module 2** 谱图解析 | `python scripts/parse_spectrum.py --input <文件> --output <输出>` | `--type ftir --baseline als --smooth savgol` |
| **Module 3** 笔记结构化 | `python scripts/structure_notes.py --input <文件> --output <输出>` | `--context "PVA film preparation"` |
| **Module 4** 生成报告 | `python scripts/generate_report.py --data <JSON> --output <报告>` | `--format md --figures-dir ./figures/` |
| **Module 4** 可视化 | `python scripts/visualize.py --data <JSON> --output-dir <目录>` | `--type timeseries` |
| **Module 5** 实验推荐 | `python scripts/recommend.py --experiments <JSON> --output <输出>` | `--n-recommendations 3 --mode optimize` |

### 全流程 Pipeline

```bash
python scripts/run_pipeline.py \
  --input-dir ./raw_data/ \
  --output-dir ./results/ \
  --mode full \
  --skill-path .claude/skills/chem-auto-lab-skill
```

**支持的模式**:

| 模式 | 说明 | 执行模块 |
|------|------|:--------:|
| `full` | 完整端到端处理 | 1→2→3→4→5 |
| `clean-and-report` | 清洗+报告 | 1→4 |
| `analyze` | 清洗+谱图+报告 | 1→2→4 |
| `recommend-only` | 仅从已有数据生成推荐 | 5 |

### 输出目录结构

```
<output_dir>/
├── pipeline_manifest.json          # 执行记录
├── .pipeline_events.jsonl          # 事件日志
├── 01_cleaned/
│   ├── <basename>.json
│   └── merged_experiments.json     # 合并后的实验数据
├── 02_spectra/
│   └── <basename>.json             # 谱图解析结果
├── 03_structured/
│   └── merged_notes.json           # 结构化笔记
├── figures/
│   ├── 01_timeseries.png
│   ├── 02_correlation_heatmap.png
│   └── plot_manifest.json
├── report.md                       # 分析报告
└── recommendations.json            # 实验推荐
```

---

## 调用案例

### 案例 1：完整分析一个文件夹

> **用户提问**: "帮我分析这个文件夹里的化学实验数据"
>
> **Skill 响应**:
> 1. 扫描文件夹，自动分类文件类型（检测到 data.xlsx、ftir_sample.csv、notebook.txt）
> 2. **Module 1** — 清洗 Excel 数据（中位数填充缺失值，IQR 法检测异常值）
> 3. **Module 2** — 解析 FTIR 谱图，自动识别峰位（3340 cm⁻¹ O-H、2940 cm⁻¹ C-H、1735 cm⁻¹ C=O 等）
> 4. **Module 3** — 结构化实验笔记，提取材料配比、工艺参数
> 5. **Module 4** — 生成 `report.md` + 趋势图和相关性热力图
> 6. **Module 5** — 推荐 3 个实验方案并排序

### 案例 2：单独执行数据清洗

> **用户提问**: "帮我把这个 Excel 数据清洗一下，有缺失值和异常值"
>
> **Skill 响应**:
> - 执行 `clean_data.py`，自动选择中位数填充策略
> - IQR 法检测并标记异常值
> - Z-score 标准化数值列
> - 输出清洗后的 JSON，报告清洗统计量

### 案例 3：单独执行谱图解析

> **用户提问**: "帮我解析这张 FTIR 谱图，标出主要吸收峰"
>
> **Skill 响应**:
> - 执行 `parse_spectrum.py --type ftir`
> - ALS 基线校正 + Savitzky-Golay 平滑
> - 峰值检测 + 归属推断
> - 输出峰位表：波数、强度、归属、半峰宽

### 案例 4：基于已有数据推荐实验

> **用户提问**: "基于这批实验结果，推荐下一步应该做什么实验"
>
> **Skill 响应**:
> - 执行 `recommend.py --mode optimize`
> - 分析已有参数覆盖空间
> - 识别未探索的条件范围（如温度未覆盖 80°C 以上的区间）
> - 按优先级输出推荐实验方案

---

## 细节参考

| 场景 | 读取文件 |
|------|---------|
| 需要了解模块详细逻辑 | `references/module-N-*.md` |
| 需要全流程编排细节 | `pipeline-execution.md` |
| 需要 Schema 校验 | `schemas/*.json` |
| 需要查看脚本参数 | 执行 `python scripts/<script>.py --help` |