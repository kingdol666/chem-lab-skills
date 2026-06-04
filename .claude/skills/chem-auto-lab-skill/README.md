# Chem-Auto-Lab Skill — 化学实验数据自动处理

> **执行模型**: LLM-native，通过 Claude Code 工具链自动执行
> **完整文档**: 参见 [SKILL.md](SKILL.md) 和 [pipeline-execution.md](pipeline-execution.md)

## 概述

对用户提供的化学实验室数据进行自动化处理，覆盖 **数据清洗 → 谱图解析 → 笔记结构化 → 报告生成 → 实验推荐** 全流程。

每个模块可独立运行，也可组合为 Full Pipeline 端到端执行。

---

## 适用场景

| 输入 | 你想做什么 | 触发的模块 |
|------|-----------|-----------|
| Excel/CSV 实验数据 | 清洗缺失值、标准化、去除异常 | Module 1 |
| FTIR/Raman/UV-Vis/HPLC/NMR 数据文件 | 峰检测、基线校正、归属推断 | Module 2 |
| 非结构化实验笔记 `.txt` | 提取材料配比、工艺参数 | Module 3 |
| 已清洗的实验数据 | 生成分析报告和图表 | Module 4 |
| 已有实验结果 | 推荐下一步实验方案 | Module 5 |
| 整个文件夹 | 全流程分析 | Full Pipeline (1→2→3→4→5) |

---

## 模块架构

```
用户输入文件
    │
    ▼
┌─────────────────────────────────────────────────────┐
│              文件自动分类                              │
│  .xlsx/.csv/.tsv → spreadsheet                      │
│  .jdx/.spc/xy.csv → spectrum                        │
│  .txt/.md/.log   → lab_notes                        │
└────────┬───────────────┬───────────────┬────────────┘
         │               │               │
         ▼               ▼               ▼
   ┌──────────┐   ┌──────────┐   ┌──────────────┐
   │ Module 1 │   │ Module 2 │   │ Module 3     │
   │ 数据清洗  │   │ 谱图解析  │   │ 笔记结构化    │
   └────┬─────┘   └────┬─────┘   └──────┬───────┘
        │               │               │
        └───────────┬───┴───────────────┘
                    ▼
            ┌──────────────┐
            │   合并数据    │
            └──────┬───────┘
                   ▼
         ┌──────────────┐    ┌──────────────┐
         │  Module 4    │───▶│  可视化图表    │
         │  报告生成     │    └──────────────┘
         └──────┬───────┘
                ▼
         ┌──────────────┐
         │  Module 5    │
         │  实验推荐     │
         └──────────────┘
```

---

## 使用方式

在 Claude Code 中通过自然语言触发，Skill 自动识别意图并路由到对应模块：

### 单模块触发

| 用户说 | 路由到 |
|-------|--------|
| "清洗/标准化/缺失值/异常值" | → Module 1 |
| "FTIR/Raman/UV-Vis/谱图/峰检测/波数" | → Module 2 |
| "实验记录/笔记/结构化" | → Module 3 |
| "报告/周报/趋势分析/统计摘要" | → Module 4 |
| "推荐/建议/下一步/参数优化" | → Module 5 |

### Full Pipeline 触发

| 用户说 | 执行 |
|-------|------|
| "帮我分析这个文件夹" / "全部" / "完整分析" | Module 1→2→3→4→5 |

### Pipeline 模式

| 模式 | 说明 | 执行模块 |
|------|------|:--------:|
| `full` | 完整端到端处理 | 1→2→3→4→5 |
| `clean-and-report` | 清洗+报告 | 1→4 |
| `analyze` | 清洗+谱图+报告 | 1→2→4 |
| `recommend-only` | 仅从已有数据生成推荐 | 5 |

---

## 支持的输入格式

| 类别 | 格式 | 检测规则 |
|------|------|---------|
| 电子表格 | `.xlsx`, `.xls`, `.csv`, `.tsv` | 按扩展名 |
| 谱图数据 | `.csv`(xy), `.txt`(xy), `.jdx`(JCAMP-DX), `.spc` | 按扩展名 + 列名模式匹配 |
| 文本笔记 | `.txt`, `.md`, `.log` | 按扩展名 |
| 已结构化数据 | `.json` | 直接加载 |

---

## 输出规范

所有模块输出遵循统一规范：

- **格式**: NDJSON 或单 JSON 对象
- **字段名**: `snake_case` 英文
- **日期**: ISO 8601 (`YYYY-MM-DDTHH:MM:SS`)
- **数值**: 附带 `unit` 字段
- **溯源**: 包含 `metadata` 块（脚本版本、时间戳、输入文件）
- **缺失值**: 显式 `null`，不用空字符串或 0

### 输出目录结构

```
<output_dir>/
├── pipeline_manifest.json          # 执行记录
├── .pipeline_events.jsonl          # 事件日志
├── 01_cleaned/
│   ├── <basename>.json             # 各文件清洗结果
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

### 案例 1：全流程分析

> **用户**: "帮我分析这个文件夹里的化学实验数据"
>
> Skill 自动识别为 Full Pipeline → 扫描文件夹 → 文件分类 → 数据清洗(Module 1) → 谱图解析(Module 2, 如有) → 笔记结构化(Module 3, 如有) → 报告+可视化(Module 4) → 实验推荐(Module 5)

### 案例 2：单独谱图解析

> **用户**: "帮我解析这张 FTIR 谱图，标出主要吸收峰"
>
> 路由到 Module 2 → 自动识别 FTIR 类型 → 基线校正 + 平滑去噪 → 峰检测 → 归属推断（如 3340 cm⁻¹ O-H 伸缩、2940 cm⁻¹ C-H 伸缩）

### 案例 3：实验推荐

> **用户**: "基于这批实验结果，推荐下一步应该做什么实验"
>
> 路由到 Module 5 → 分析参数空间覆盖度 → 识别未探索条件 → 按优先级输出推荐方案

---

## 详细文档索引

| 文档 | 内容 |
|------|------|
| [SKILL.md](SKILL.md) | 主技能文件：意图路由、模块选择、执行协议 |
| [pipeline-execution.md](pipeline-execution.md) | 全流水线编排：步骤、错误恢复、事件日志 |
| [references/module-1-data-cleaning.md](references/module-1-data-cleaning.md) | 数据清洗规则和配置 |
| [references/module-2-spectroscopy.md](references/module-2-spectroscopy.md) | 谱图解析规则和峰检测算法 |
| [references/module-3-log-structuring.md](references/module-3-log-structuring.md) | 笔记结构化的 few-shot 示例 |
| [references/module-4-report-generation.md](references/module-4-report-generation.md) | 报告结构和图表选择规则 |
| [references/module-5-experiment-recommend.md](references/module-5-experiment-recommend.md) | 推荐类型和推理规则 |
| [schemas/](schemas/) | 各模块输出的 JSON Schema |
