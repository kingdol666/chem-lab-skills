# Domain Literature Experiment Extraction & Ontology Skill — 文献数据挖掘与知识图谱

> **执行模型**: LLM-native，通过 Claude Code 工具链（WebSearch + WebFetch + LLM 推理）自动执行
> **完整文档**: 参见 [SKILL.md](SKILL.md) 和 [pipeline-execution.md](pipeline-execution.md)

## 概述

系统性地从科学文献中收集、提取和结构化实验数据，构建领域本体和知识图谱。覆盖 **文献搜索 → 实验提取 → 数据标准化 → 溯源 → 科学解释 → 本体构建 → 文献综述** 全流程。

每个模块可独立运行，也可组合为 Full Pipeline（Module 1→2→3→4→5→6→7）端到端执行。

---

## 适用场景

| 你想做什么 | 触发的模块 |
|-----------|-----------|
| 搜索/收集特定领域的文献 | Module 1 |
| 从论文中提取实验参数（配比、温度、性能） | Module 2 |
| 标准化单位和命名 | Module 3 |
| 查看数据来源和置信度 | Module 4 |
| 生成科学解释 | Module 5 |
| 构建知识图谱/本体 | Module 6 |
| 文献综述、趋势分析、研究空白 | Module 7 |
| 上述全部 | Full Pipeline (1→2→3→4→5→6→7) |

适用于 **PVA/BOPET 光学膜**（预置领域知识）、催化剂合成、电池材料、高分子改性、药物配方等领域。

---

## 模块架构

```
用户输入 (PDF / 搜索关键词 / 预提取数据)
    │
    ▼
┌───────────────────────────────────────────────────────────────────┐
│  Module 1: 文献获取                                               │
│  WebSearch 多源搜索 → 去重(DOI/标题/作者) → source_manifest.json  │
│  或本地 PDF 解析 → 提取全文文本                                    │
└──────────────────────────────┬────────────────────────────────────┘
                               ▼
┌───────────────────────────────────────────────────────────────────┐
│  Module 2: 实验提取 (LLM 驱动)                                    │
│  表格解析 + 正文模式匹配 + 图表描述 → experiments_raw.json          │
│  每个数据点绑定 source_snippet + 初始置信度                        │
└──────────────────────────────┬────────────────────────────────────┘
                               ▼
┌───────────────────────────────────────────────────────────────────┐
│  Module 3: 数据标准化                                             │
│  单位转换 (mil→μm, psi→MPa) + 同义词映射 (glycerin→glycerol)      │
│  → experiments_normalized.json + CSV + Excel                      │
└──────────────────────────────┬────────────────────────────────────┘
                               ▼
┌───────────────────────────────────────────────────────────────────┐
│  Module 4: 溯源与置信度                                           │
│  每个字段 → 来源文献/页码/原文片段/提取方法                         │
│  置信度: 1.0(表格直接引用) ~ 0.1(从图数字化)                        │
└──────────────────────────────┬────────────────────────────────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
     ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
     │  Module 5    │ │  Module 6    │ │  Module 7    │
     │  科学解释     │ │  本体构建     │ │  文献综述     │
     │  参数物理意义 │ │  8类实体关系  │ │  趋势+空白   │
     └──────────────┘ └──────────────┘ └──────────────┘
```

---

## 使用方式

在 Claude Code 中通过自然语言触发，Skill 自动识别意图并路由到对应模块：

### 单模块触发

| 用户说（任何语言） | 路由到 |
|-------------------|--------|
| "搜索文献/找论文/检索" | → Module 1 |
| "提取数据/抽取实验/从论文中提取" | → Module 2 |
| "标准化/归一化/统一单位" | → Module 3 |
| "溯源/出处/置信度" | → Module 4 |
| "解释/为什么/机理分析" | → Module 5 |
| "本体/知识图谱/关系抽取" | → Module 6 |
| "综述/总结/趋势/研究空白" | → Module 7 |

### Full Pipeline 触发

| 用户说 | 执行 |
|-------|------|
| "帮我把PVA光学膜文献的实验数据全部提取出来" | Module 1→2→3→4→5→6→7 |

### Pipeline 模式

| 模式 | 说明 | 执行模块 | 输入要求 |
|------|------|:--------:|---------|
| `full` | 完整端到端 | 1→2→3→4→5→6→7 | 关键词或论文文件 |
| `extract-only` | 搜索+提取+标准化+溯源 | 1→2→3→4 | 同上 |
| `knowledge-build` | 构建本体+综述 | 6→7 | 已有 `experiments_normalized.json` |
| `explain` | 生成科学解释 | 5 | 已有 `experiments_normalized.json` |
| `resume` | 从指定模块继续 | N→...→7 | 需要前 N-1 模块的输出 |

---

## 预置领域知识（PVA/BOPET 光学膜）

本 Skill 预置了 PVA/BOPET 光学膜研究领域的完整词库：

| 类别 | 覆盖范围 |
|------|---------|
| **材料** | PVA (1799/1788/1792/0588), PET (光学级), TAC, COP, PMMA, PC |
| **添加剂** | 增塑剂(甘油/EG/PEG/山梨醇), 交联剂(硼酸/戊二醛/柠檬酸), 纳米填料(CNC/CNF/MMT/GO/CNT/SiO₂/TiO₂/ZnO) |
| **工艺** | 溶液浇铸、熔融挤出、单向/双向拉伸、涂布(棒/凹版/狭缝)、干燥(热风/红外)、退火 |
| **性能** | 透光率(%), 雾度(%), 拉伸强度(MPa), 断裂伸长率(%), 杨氏模量(GPa), WVTR, OTR, Tg/Tm/Td(°C), 结晶度(%), 接触角(°) |
| **仪器** | UV-Vis, haze meter, UTM, DSC, TGA, DMA, SEM, AFM, XRD, FTIR |
| **单位转换** | 见 `assets/unit_conversions.json` |

**适配其他领域**：替换 `assets/` 目录下的词库文件即可。

---

## 核心设计原则

### 1. 溯源至上（Provenance First）

每个提取的数据点携带：
- 来源文献（`source_id`）
- 页码/位置（`source_page`, `source_location`）
- 原文片段（`source_snippet`）
- 提取方法（`extraction_method`: table_parse / text_regex / llm_extraction）

### 2. 置信度体系

| 置信度 | 含义 |
|:------:|------|
| 1.0 | 表格中直接引用的数值 |
| 0.8-0.9 | 正文明确写出的数值+单位 |
| 0.6-0.7 | 需要推理才能提取 |
| 0.4-0.5 | 有歧义，记录了多种可能值 |
| 0.1-0.3 | 从图表数字化估算 |
| 0.0 | 缺失 |

### 3. 不编造（No Fabrication）

- 缺失字段显式为 `null`
- 不使用 0、空字符串、"N/A" 替代
- 单位无法转换时保留原值并标记
- 部分完成+质量标注 优于 完整输出+编造数据

---

## 输出目录结构

```
<output_dir>/
├── pipeline_manifest.json
├── .pipeline_events.jsonl
├── 01_literature/
│   ├── source_manifest.json           # 所有文献元数据
│   ├── dedup_report.json              # 去重报告
│   └── full_text/                     # 提取的全文文本
├── 02_extracted/
│   ├── experiments_raw.json           # 原始提取数据
│   └── extraction_log.json            # 提取统计
├── 03_normalized/
│   ├── experiments_normalized.json    # 标准化后数据（核心输出）
│   ├── experiments.csv                # CSV 导出
│   ├── experiments.xlsx               # Excel 导出
│   └── normalization_log.json         # 转换日志
├── 04_provenance/
│   ├── provenance.json                # 完整溯源表
│   └── confidence_distribution.json   # 置信度分布
├── 05_explanations/
│   ├── explanations.md                # 科学解释报告
│   └── explanations.json
├── 06_ontology/
│   ├── ontology.json                  # JSON 本体
│   ├── ontology.ttl                   # Turtle 格式
│   └── ontology.owl                   # OWL 格式
├── 07_summary/
│   ├── literature_summary.md          # 文献综述
│   └── literature_summary.json
├── ambiguities.json                   # 未解决歧义
└── run_summary.json                   # 最终统计
```

---

## 调用案例

### 案例 1：全流程文献数据提取

> **用户**: "帮我把PVA光学膜文献的实验数据全部提取出来"
>
> Module 1 搜索文献 → Module 2 逐篇提取实验条件/性能参数 → Module 3 标准化单位 → Module 4 绑定原文出处 → Module 5 解释物理意义 → Module 6 构建本体 → Module 7 生成综述和研究空白报告

### 案例 2：从本地 PDF 提取

> **用户**: "我这里有十几篇 PVA 光学膜的 PDF 论文，帮我把实验数据都抽出来"
>
> 跳过 Module 1 在线搜索 → 直接读取本地 PDF → 解析表格和正文中的实验参数 → 合并输出 `experiments_normalized.json` + CSV + Excel

### 案例 3：构建知识图谱

> **用户**: "根据已提取的数据，构建一个材料-工艺-性能的知识图谱"
>
> 路由到 Module 6 → 识别 8 类实体（Material, Additive, ProcessStep, Instrument, Condition, Measurement, Property, Result）→ 抽取关系（hasAdditive, processedBy, hasProperty）→ 输出 JSON / OWL / Turtle

### 案例 4：研究空白分析

> **用户**: "从这些文献里看看哪些实验条件还没有人研究过"
>
> 路由到 Module 7 → 统计已有参数覆盖范围（如温度 60-120°C 已覆盖，55°C 和 130°C 为空白）→ 输出研究空白报告

---

## 详细文档索引

| 文档 | 内容 |
|------|------|
| [SKILL.md](SKILL.md) | 主技能文件：意图路由、模块选择、领域配置、执行协议 |
| [pipeline-execution.md](pipeline-execution.md) | 全流水线编排：步骤、错误恢复、数据传递、断点续跑 |
| [references/module-1-literature-acquisition.md](references/module-1-literature-acquisition.md) | 搜索策略、去重规则、全文提取 |
| [references/module-2-experiment-extraction.md](references/module-2-experiment-extraction.md) | 提取模式、表格解析、字段映射 |
| [references/module-3-data-normalization.md](references/module-3-data-normalization.md) | 单位转换、同义词映射、命名规则 |
| [references/module-4-evidence-traceability.md](references/module-4-evidence-traceability.md) | 溯源 schema、置信度评分规则 |
| [references/module-5-explanation-generation.md](references/module-5-explanation-generation.md) | 科学解释模板、注意事项 |
| [references/module-6-ontology-modeling.md](references/module-6-ontology-modeling.md) | 类层次结构、关系抽取、OWL/JSON-LD 导出 |
| [references/module-7-literature-summary.md](references/module-7-literature-summary.md) | 趋势综合、空白分析、方向推荐 |
| [schemas/](schemas/) | 各模块输出的 JSON Schema |
| [assets/](assets/) | 领域词库、同义词映射、单位转换表 |
| [templates/](templates/) | 提取配置模板 |
