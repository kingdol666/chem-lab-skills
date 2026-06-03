# Domain Literature Experiment Extraction & Ontology Skill — 文献数据提取与知识图谱

## 概述

系统地从科学文献中收集、提取和结构化实验数据，构建领域本体和知识图谱，覆盖**文献搜索 → 实验提取 → 数据标准化 → 溯源 → 科学解释 → 本体构建 → 文献综述**全流程。

### 适用场景

- 从研究论文中提取实验参数（材料配方、工艺条件、性能指标）
- 构建领域知识图谱 / 本体（JSON / OWL / Turtle 格式）
- 多篇文献的交叉对比和趋势分析
- 研究空白识别和综述生成
- 适用于 PVA/BOPET 光学膜、催化剂、电池材料、高分子改性、药物配方等领域

---

## 执行流程

```
用户输入 (PDF论文 / 搜索关键词 / 预提取数据)
    │
    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                        Module 1: 文献获取                               │
│  在线搜索 (Semantic Scholar/PubMed) ◄──► 本地 PDF 解析                  │
│  去重 → 构建 source_manifest.json                                      │
└───────────────────────────────────┬────────────────────────────────────┘
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                        Module 2: 实验提取                               │
│  表格解析 ◄──► 文本模式匹配 ◄──► 图表描述提取                            │
│  每个数据点绑定 source_snippet + 初始置信度                              │
└───────────────────────────────────┬────────────────────────────────────┘
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                        Module 3: 数据标准化                             │
│  单位转换 (mil→μm, psi→MPa) + 同义词映射 (glycerin→glycerol)           │
│  输出 experiments_normalized.json + CSV + Excel                        │
└───────────────────────────────────┬────────────────────────────────────┘
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                        Module 4: 溯源与置信度                           │
│  每个字段关联原文位置、页码、提取方法                                    │
│  置信度评分: 1.0(直接引用表格) ~ 0.1(从图数字化)                        │
└───────────────────────────────────┬────────────────────────────────────┘
                                    ▼
               ┌────────────────────┼────────────────────┐
               ▼                    ▼                    ▼
     ┌──────────────────┐ ┌────────────────┐ ┌────────────────────┐
     │   Module 5       │ │   Module 6     │ │   Module 7         │
     │   科学解释        │ │   本体构建      │ │   文献综述          │
     │   解释每个参数的  │ │   提取 8 类实体 │ │   主题归纳、趋势    │
     │   物理意义和趋势  │ │   (材料/工艺/   │ │   分析、研究空白    │
     │                  │ │    性能等)      │ │   识别              │
     └──────────────────┘ └────────────────┘ └────────────────────┘
```

### 可组合的 Pipeline 模式

```
full (7个模块全跑)        extract-only (只提取+标准化)
1→2→3→4→5→6→7           1→2→3→4

knowledge-build (已有数据)  explain (只做解释)
6→7                       5
```

---

## 使用方式

### 环境准备

```bash
pip install pandas numpy openpyxl jsonschema
```

### 全流程 Pipeline

```bash
python scripts/run_pipeline.py \
  --input-dir ./papers/ \
  --output-dir ./literature_output/ \
  --mode full \
  --skill-path .claude/skills/domain-literature-experiment-extraction-ontology-skill \
  --domain pva_bopet \
  --search-keywords "PVA optical film, light transmittance, haze, tensile strength"
```

### Pipeline 模式

| 模式 | 说明 | 执行模块 | 输入要求 |
|------|------|:--------:|---------|
| `full` | 完整端到端处理 | 1→2→3→4→5→6→7 | 关键词或论文文件 |
| `extract-only` | 搜索+提取+标准化+溯源 | 1→2→3→4 | 同上 |
| `knowledge-build` | 从已有数据构建本体+综述 | 6→7 | 已有 `experiments_normalized.json` |
| `explain` | 为已有数据生成解释 | 5 | 已有 `experiments_normalized.json` |
| `resume` | 从指定模块继续 | N→...→7 | 需要前 N-1 个模块的输出 |

### 脚本命令速查

| 脚本 | 功能 | 关键参数 |
|------|------|---------|
| `run_pipeline.py` | 全流程编排 | `--input-dir`, `--output-dir`, `--mode`, `--domain`, `--search-keywords` |
| `normalize_data.py` | 单位转换+名称标准化 | `--input`, `--output`, `--vocabulary`, `--unit-map`, `--synonym-map` |
| `build_ontology.py` | 构建本体 | `--experiments`, `--output`, `--format` (json/owl/ttl/jsonld) |
| `generate_explanations.py` | 生成科学解释 | `--experiments`, `--output`, `--language` (zh/en) |
| `summarize_literature.py` | 文献综述 | `--experiments`, `--provenance`, `--output`, `--focus-areas` |
| `export_data.py` | 导出 CSV/Excel | `--input`, `--export-csv`, `--export-xlsx` |
| `validate_outputs.py` | Schema 校验 | `--data`, `--schema`, `--report-errors` |

### 输入格式

| 类别 | 格式 | 说明 |
|------|------|------|
| 研究论文 | `.pdf` | 全文 PDF 提取 |
| 网页文章 | URL/HTML | 开放获取页面 |
| 补充材料 | `.pdf`, `.xlsx`, `.csv`, `.docx` | 表格和图表 |
| 专利 | `.pdf`, HTML | 实验例部分 |
| 预提取数据 | `.json`, `.csv` | 已有结构化数据 |
| 搜索关键词 | 字符串 | 会自动执行在线搜索 |

### 输出目录结构

```
<output_dir>/
├── pipeline_manifest.json
├── .pipeline_events.jsonl
├── 01_literature/
│   └── source_manifest.json           # 文献元数据
├── 02_extracted/
│   └── experiments_raw.json           # 原始提取数据
├── 03_normalized/
│   ├── experiments_normalized.json    # 标准化后数据
│   ├── experiments.csv                # CSV 导出
│   └── experiments.xlsx              # Excel 导出
├── 04_provenance/
│   ├── provenance.json                # 溯源表
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

> **用户提问**: "帮我把PVA光学膜文献的实验数据全部提取出来"
>
> **Skill 响应**:
> 1. **Module 1** — 在线搜索 PVA 光学膜相关文献，自动去重，构建 source manifest
> 2. **Module 2** — 逐篇解析论文中的表格和正文，提取实验条件（配比、温度、时间）和性能数据（透光率、雾度、拉伸强度）
> 3. **Module 3** — 标准化单位（mil→μm, psi→MPa）、同义词映射（glycerin→glycerol）
> 4. **Module 4** — 每个数据点绑定原文出处、页码、原文片段，计算置信度
> 5. **Module 5** — 解释每个参数的含义和趋势
> 6. **Module 6** — 构建材料-工艺-性能本体（8类实体、关系抽取）
> 7. **Module 7** — 生成文献综述，包含趋势分析、研究空白识别

### 案例 2：从本地 PDF 提取

> **用户提问**: "我这里有十几篇 PVA 光学膜的 PDF 论文，帮我把实验数据都抽出来"
>
> **Skill 响应**:
> - 跳过 Module 1 在线搜索，直接扫描本地 PDF 文件夹
> - 对每篇 PDF 提取文本内容（扫描版尝试 OCR）
> - 解析表格中的数据行（材料配比、工艺参数、性能指标）
> - 输出结构化 JSON + CSV + Excel

### 案例 3：构建知识图谱

> **用户提问**: "根据已提取的 PVA 光学膜实验数据，构建一个材料-工艺-性能三要素的知识图谱"
>
> **Skill 响应**:
> - 执行 `build_ontology.py --format json,ttl,owl`
> - 自动提取 8 类实体：Material、Additive、ProcessStep、Instrument、Condition、Measurement、Property、Result
> - 识别关系：`hasAdditive`、`processedBy`、`hasProperty`、`conductedUnder`
> - 输出 3 种格式：JSON（内部表示）、Turtle、OWL（标准本体语言）

### 案例 4：文献综述和研究空白

> **用户提问**: "从这些文献里看看哪些实验条件还没有人研究过"
>
> **Skill 响应**:
> - 执行 Module 7，统计分析所有提取数据的参数覆盖范围
> - 识别空白：如温度 60-120°C 已被覆盖，但 55°C 和 130°C 未有文献报道
> - 输出研究空白报告，附带新颖度评分

---

## 核心设计原则

1. **溯源至上** — 每个数据点携带来源文献、页码、原文片段，可追溯验证
2. **置信度体系** — 0.0(无证据) ~ 1.0(直接引用表格)，从不编造数据
3. **不编造** — 缺失字段明确为 `null`，从不推测或制造数据
4. **部分成功 > 完整但错误** — 有质量标注的部分输出优于编造数据的完整输出

---

## 细节参考

| 场景 | 读取文件 |
|------|---------|
| 需要了解模块详细逻辑 | `references/module-N-*.md` |
| 需要全流程编排细节 | `pipeline-execution.md` |
| 需要 Schema 校验 | `schemas/*.json` |
| 需要领域词库 | `assets/pva_bopet_vocabulary.json` |
| 需要提取配置模板 | `templates/extraction_config_template.json` |
| 需要查看脚本参数 | 执行 `python scripts/<script>.py --help` |