# Chem-Skill — 化学实验室智能分析 Skill 套件

## 概览

本仓库包含 **3 个可独立或组合使用**的化学领域 AI Skill，运行于 Claude Code + oh-my-claudecode 环境下。每个 Skill 采用**渐进式加载、模块化设计、Schema 校验输出**的架构。

| Skill | 定位 | 适用场景 |
|-------|------|---------|
| [chem-auto-lab-skill](#1-chem-auto-lab-skill) | 化学实验数据自动处理 | 清洗数据、谱图解析、生成报告、实验推荐 |
| [domain-literature-experiment-extraction-ontology-skill](#2-domain-literature-experiment-extraction-ontology-skill) | 文献数据提取与知识图谱 | 从论文提取实验数据、构建本体、文献综述 |
| [literature-to-lab-bridge](#3-literature-to-lab-bridge) | 文献→实验室桥梁 | 搜索文献→提取数据→化学分析→带证据分级的推荐 |

### 架构关系

```
domain-literature-experiment-extraction-ontology-skill     chem-auto-lab-skill
         ┌──────────────────────────┐              ┌──────────────────────────┐
         │  Module 1: 文献搜索       │              │  Module 1: 数据清洗       │
         │  Module 2: 实验提取       │              │  Module 2: 谱图解析       │
         │  Module 3: 标准化        │              │  Module 3: 笔记结构化     │
         │  Module 4: 溯源/置信度    │              │  Module 4: 报告生成       │
         │  Module 5: 科学解释       │              │  Module 5: 实验推荐       │
         │  Module 6: 本体建模       │              │                          │
         │  Module 7: 文献综述       │              │                          │
         └──────────────────────────┘              └──────────────────────────┘
                      │                                          │
                      └────────────→ literature-to-lab-bridge ←──┘
                                      (Phase 0→1→1.5→2→3)
```

---

## 快速开始

### 环境要求

- **Python 3.10+** — 所有脚本使用 Python
- **依赖安装** (任一 Skill 的 requirements.txt):

```bash
pip install pandas numpy matplotlib scipy openpyxl seaborn jsonschema
```

### 目录结构

```
.claude/skills/
├── chem-auto-lab-skill/                      # Skill 1
│   ├── SKILL.md                              # 主描述文件（意图路由、模块选择）
│   ├── pipeline-execution.md                 # 全流水线编排细节
│   ├── scripts/                              # 可执行 Python 脚本
│   ├── references/                           # 各模块的详细参考文档
│   ├── schemas/                              # JSON Schema 校验文件
│   ├── assets/                               # 领域词库、单位转换表
│   └── templates/                            # 输出模板
├── domain-literature-experiment-extraction-ontology-skill/  # Skill 2
│   ├── SKILL.md
│   ├── pipeline-execution.md
│   ├── scripts/
│   ├── references/
│   ├── schemas/
│   └── assets/
└── literature-to-lab-bridge/                 # Skill 3
    ├── SKILL.md
    ├── pipeline-execution.md
    ├── scripts/
    └── schemas/
```

---

## 1. chem-auto-lab-skill — 化学实验数据自动处理

### 适用场景

用户提供化学实验室数据（Excel/CSV/TXT/仪器导出文件），需要：
- 数据清洗、标准化、异常值处理
- FTIR / Raman / UV-Vis / HPLC / NMR 谱图解析
- 实验记录结构化（从非结构化的文本笔记中提取参数）
- 生成分析报告和可视化图表
- 推荐下一步实验方案

### 模块架构

```
用户请求
   │
   ▼
┌─────────────────────────────────────────────────────────────┐
│                    意图分类                                   │
├──────────┬──────────┬───────────┬───────────┬──────────────┤
│ 数据清洗  │ 谱图解析  │ 笔记结构化 │ 报告生成   │ 实验推荐       │
│          │          │           │           │              │
│ Module 1 │ Module 2 │ Module 3 │ Module 4 │ Module 5     │
└──────────┴──────────┴───────────┴───────────┴──────────────┘
```

### 使用方式

#### 单个模块调用

所有脚本遵循一致的 CLI 接口：

```bash
# 数据清洗
python scripts/clean_data.py \
  --input data.xlsx \
  --output cleaned.json \
  --imputation median \
  --outlier iqr \
  --normalize zscore

# 谱图解析
python scripts/parse_spectrum.py \
  --input ftir_sample.csv \
  --output peaks.json \
  --type ftir \
  --baseline als \
  --smooth savgol

# 实验笔记结构化
python scripts/structure_notes.py \
  --input notebook.txt \
  --output structured.json

# 生成报告
python scripts/generate_report.py \
  --data merged_experiments.json \
  --output report.md \
  --format md

# 生成可视化
python scripts/visualize.py \
  --data merged_experiments.json \
  --output-dir figures/ \
  --type timeseries

# 实验推荐
python scripts/recommend.py \
  --experiments merged_experiments.json \
  --output recommendations.json \
  --n-recommendations 3 \
  --mode optimize
```

#### 全流程 Pipeline

```bash
python scripts/run_pipeline.py \
  --input-dir ./raw_data/ \
  --output-dir ./results/ \
  --mode full \
  --skill-path .claude/skills/chem-auto-lab-skill
```

支持多种 Pipeline 模式:

| Mode | 说明 | 执行模块 |
|------|------|---------|
| `full` | 完整端到端处理 | 1 → 2 → 3 → 4 → 5 |
| `clean-and-report` | 清洗+报告 | 1 → 4 |
| `analyze` | 清洗+谱图+报告 | 1 → 2 → 4 |
| `recommend-only` | 仅从已有数据生成推荐 | 5 |

#### 支持的输入格式

| 类别 | 格式 |
|------|------|
| 电子表格 | `.xlsx`, `.xls`, `.csv`, `.tsv` |
| 文本笔记 | `.txt`, `.md`, `.log` |
| 谱图数据 | `.csv` (xy), `.txt` (xy), `.jdx` (JCAMP-DX), `.spc` |
| JSON | 已结构化的实验数据 |

### 输出规范

- 所有输出为 **NDJSON** 或单个 JSON 对象
- JSON key 使用 **snake_case** 英文
- 日期使用 **ISO 8601** (`YYYY-MM-DDTHH:MM:SS`)
- 数值字段附带 `unit` 字段
- 所有输出包含 `metadata` 溯源信息

### 提问案例

以下是向 Claude Code 提问后，Skill 会自动触发并执行的示例：

> **案例 1：全流程分析**
> ```
> 帮我分析这个文件夹里的化学实验数据
> ```
> → Skill 自动识别为 Full Pipeline 模式，扫描文件夹中的所有文件类型，按 数据清洗 → 谱图解析(如有) → 笔记结构化(如有) → 报告生成 → 实验推荐 的顺序执行，输出 `report.md` + `recommendations.json` + 可视化图表。

> **案例 2：单个模块 — 数据清洗**
> ```
> 帮我把这个 Excel 数据清洗一下，有缺失值和异常值
> ```
> → 路由到 Module 1，执行 `clean_data.py`，自动处理缺失值（中位数填充）和异常值（IQR 法），输出标准化后的 JSON。

> **案例 3：单个模块 — 谱图解析**
> ```
> 帮我解析这张 FTIR 谱图，标出主要吸收峰
> ```
> → 路由到 Module 2，执行 `parse_spectrum.py --type ftir`，自动基线校正、平滑去噪、峰检测，输出峰位表 + 归属推断（如 3340 cm⁻¹ O-H 伸缩、2940 cm⁻¹ C-H 伸缩）。

> **案例 4：单个模块 — 实验推荐**
> ```
> 基于这批实验结果，推荐下一步应该做什么实验
> ```
> → 路由到 Module 5，执行 `recommend.py`，分析已有数据覆盖的参数空间，推荐探索未覆盖的条件范围，输出带优先级的实验方案。

---

## 2. domain-literature-experiment-extraction-ontology-skill — 文献数据提取与知识图谱

### 适用场景

需要系统地从科学文献中收集、提取和结构化实验数据，适用于：
- PVA/BOPET 光学膜、催化剂合成、电池材料、高分子改性、药物配方等领域
- 从论文中提取实验参数（材料配方、工艺条件、性能指标）
- 构建领域本体/知识图谱
- 生成文献综述和研究空白分析

### 模块架构

```
用户请求
   │
   ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                         意图分类                                          │
├────────────┬───────────┬──────────┬──────────┬─────────┬────────┬───────┤
│ 搜索/收集   │ 提取实验   │ 标准化   │ 溯源/    │ 解释/   │ 本体/  │ 综述/ │
│ 文献       │ 数据      │ 归一化   │ 置信度   │ 说明    │ 知识图谱│ 总结  │
│            │           │          │          │         │        │       │
│ Module 1   │ Module 2  │ Module 3 │ Module 4 │ Module 5│Module 6│Module7│
└────────────┴───────────┴──────────┴──────────┴─────────┴────────┴───────┘
```

### 使用方式

#### 全流程 Pipeline

```bash
python scripts/run_pipeline.py \
  --input-dir ./papers/ \
  --output-dir ./literature_output/ \
  --mode full \
  --skill-path .claude/skills/domain-literature-experiment-extraction-ontology-skill \
  --domain pva_bopet \
  --search-keywords "PVA optical film, light transmittance, haze, tensile strength"
```

#### Pipeline 模式

| Mode | 说明 | 执行模块 |
|------|------|---------|
| `full` | 完整端到端处理 | 1 → 2 → 3 → 4 → 5 → 6 → 7 |
| `extract-only` | 搜索+提取+标准化+溯源 | 1 → 2 → 3 → 4 |
| `knowledge-build` | 从已有数据构建本体+综述 | 6 → 7 |
| `explain` | 为已有数据生成科学解释 | 5 |
| `resume` | 从指定模块继续执行 | N → ... → 7 |

#### 支持的输入

| 类别 | 格式 | 说明 |
|------|------|------|
| 研究论文 | `.pdf` | 全文 PDF |
| 网页文章 | URL/HTML | 开放获取页面 |
| 补充材料 | `.pdf`, `.xlsx`, `.csv`, `.docx` | 表格和图表 |
| 专利 | `.pdf`, HTML | 实验例部分 |
| 预提取数据 | `.json`, `.csv` | 已结构化的实验记录 |

#### 领域配置（预置 PVA/BOPET 光学膜）

Skill 预置了丰富的领域知识：

| 类别 | 覆盖范围 |
|------|---------|
| **材料** | PVA (1799/1788/1792/0588), PET (光学级), TAC, COP, PMMA, PC |
| **添加剂** | 增塑剂(甘油/EG/PEG/山梨醇), 交联剂(硼酸/戊二醛/柠檬酸), 纳米填料(CNC/CNF/MMT/GO/CNT/SiO₂/TiO₂/ZnO) |
| **工艺** | 溶液浇铸、熔融挤出、单向/双向拉伸、涂布(棒/凹版/狭缝)、干燥(热风/红外)、热处理/退火 |
| **性能** | 透光率(%)、雾度(%)、拉伸强度(MPa)、断裂伸长率(%)、杨氏模量(GPa)、WVTR、OTR、Tg/Tm/Td(°C)、结晶度(%)、接触角(°) |

### 核心设计原则

1. **溯源至上** — 每个数据点携带来源文献、页码、原文片段
2. **置信度体系** — 0.0(无证据) ~ 1.0(直接引用)，从不编造数据
3. **高质量输出** — 部分完成+质量标注 优于 完整输出+编造数据

### 提问案例

> **案例 1：全流程文献数据提取**
> ```
> 帮我把PVA光学膜文献的实验数据全部提取出来
> ```
> → 触发 Full Pipeline (Module 1→2→3→4→5→6→7)。自动搜索文献 → 逐篇提取实验条件/性能参数 → 标准化单位（mil→μm, psi→MPa）→ 关联原文出处 → 构建材料-工艺-性能本体 → 生成文献综述，输出完整的 `experiments_normalized.json` + `ontology.json` + `literature_summary.md`。

> **案例 2：从已有论文文件夹提取**
> ```
> 我这里有十几篇 PVA 光学膜的 PDF 论文，帮我把实验数据都抽出来
> ```
> → 跳过 Module 1 在线搜索，直接读取本地 PDF，对每篇论文解析表格和正文中的实验参数，合并输出结构化数据。

> **案例 3：构建知识图谱**
> ```
> 根据已提取的 PVA 光学膜实验数据，构建一个材料-工艺-性能三要素的知识图谱
> ```
> → 路由到 Module 6，执行 `build_ontology.py`，自动识别实体和关系（材料→hasAdditive→增塑剂、工艺→conductedUnder→温度条件、材料→hasProperty→透光率），输出 JSON / OWL / Turtle 格式的本体文件。

> **案例 4：研究空白分析**
> ```
> 从这些文献里看看哪些实验条件还没有人研究过
> ```
> → 路由到 Module 7 的 gap analysis 部分，统计分析已有参数覆盖范围（如温度范围 60-120°C 已被覆盖，55°C 和 130°C 为空白区间），输出研究空白报告。

### 输出目录结构

```
<output_dir>/
├── pipeline_manifest.json
├── 01_literature/source_manifest.json       # 所有文献元数据
├── 02_extracted/experiments_raw.json        # 原始提取数据
├── 03_normalized/experiments_normalized.json # 标准化后数据
├── 03_normalized/experiments.csv            # CSV 导出
├── 03_normalized/experiments.xlsx           # Excel 导出
├── 04_provenance/provenance.json            # 溯源表
├── 05_explanations/explanations.md          # 科学解释
├── 06_ontology/ontology.json                # 本体 (也可输出 OWL/TTL)
├── 07_summary/literature_summary.md         # 文献综述
├── ambiguities.json                         # 未解决的歧义问题
└── run_summary.json                         # 最终统计
```

---

## 3. literature-to-lab-bridge — 文献到实验室的桥梁

### 适用场景

当用户需要**同时**完成文献搜索+数据提取+化学分析的全流程。典型的用户提问：
- "搜索 PVA 光学膜文献，提取实验数据，然后做化学分析"
- "从论文里收集催化剂合成数据，清洗后生成报告并推荐下一步实验"
- "帮我查找最近 5 年电池材料的文献，把数据提取出来分析趋势"

### 架构

```
Phase 0: 多轮迭代文献搜索 (4轮 → 逐步精准)
    │
    ▼
Phase 1: 文献提取+标准化 (调用 domain-literature skill)
    │
    ▼
Quality Gate: 数据量≥10条 && 置信度≥0.5 && 论文≥3篇
    │
    ▼
Phase 1.5: 研究空白验证 (交叉验证 + 打分)
    │
    ▼
Data Transformer: 文献 Schema → 实验室 Schema
    │
    ▼
Phase 2: 实验室分析 (调用 chem-auto-lab skill)
    │
    ▼
Phase 3: 证据分级推荐 (A/B/C/D/E + 可行性评分 → TOP_PLAN.md)
```

### 使用方式

#### 完整 Bridge Pipeline

Bridge pipeline 自动编排所有阶段：

```bash
python scripts/bridge_pipeline.py \
  --mode full \
  --domain pva_bopet \
  --search-keywords "PVA optical film, light transmittance, haze, CNC reinforcement" \
  --skill-path .claude/skills/literature-to-lab-bridge \
  --min-records 10 \
  --min-confidence 0.5 \
  --min-papers 3
```

#### 各阶段独立运行

关注文献部分的场景，跳过实验室分析：

```bash
python scripts/bridge_pipeline.py \
  --mode literature-only \
  --domain pva_bopet \
  --search-keywords "PVA optical film, light transmittance"
```

已有文献提取结果，只需实验室分析：

```bash
python scripts/bridge_pipeline.py \
  --mode lab-only \
  --domain pva_bopet \
  --phase1-output ./previous_run/phase1_output/
```

### 反编造保障机制

Bridge 的最大特色是 **反编造保障**（Anti-Fabrication Guarantees）：

| 阶段 | 保障机制 |
|------|---------|
| **Quality Gate** | 提取数据不足时不盲目执行，报告用户并征求意见 |
| **Phase 1.5** | 交叉验证"研究空白"的真实性，排除假空白 |
| **Phase 3** | 每个 claim 必须锚定文献证据，附引用来源 |

#### 证据分级体系

| 等级 | 定义 | 来源要求 |
|:----:|------|---------|
| **A** | 直接实验证据 | ≥3 篇论文一致数据 |
| **B** | 强间接证据 | ≥1 篇论文相关数据 |
| **C** | 弱间接/理论 | 机理明确但无直接数据 |
| **D** | 推断 | 化学直觉、相邻体系 |
| **F** | 猜测 | **排除，不进入最终输出** |

#### 可行性评分

```
可行性 = 设备得分(25%) + 复杂度得分(25%) + 时间得分(25%) + 成本得分(25%)
```

最终输出**仅一个最优方案** `TOP_PLAN.md`，选择标准：

```
max(证据等级评分 × 可行性评分 × 新颖度评分)
```

### 最终输出结构

```
<run_dir>/
├── bridge_manifest.json                    # 完整执行追踪
├── .bridge_events.jsonl                    # 事件日志
├── phase0_output/                          # 多轮搜索
│   ├── search_rounds.json
│   └── merged_corpus.json
├── phase1_output/                          # 文献提取结果
├── phase1.5_output/verified_gaps.json      # 验证后的研究空白
├── phase2_input/lab_experiments.json       # 转换后的数据
├── phase2_output/                          # 实验室分析结果
│   ├── report.md
│   ├── recommendations.json
│   └── figures/
├── phase3_output/
│   ├── evidence_graded_recommendations.json
│   └── TOP_PLAN.md                         # 最优实验方案
└── integrated_report.md                    # 综合报告
```

### 提问案例

> **案例 1：完整 Bridge 全流程**
> ```
> 帮我搜索 PVA 光学膜的文献，提取实验数据，分析其中的趋势，然后推荐下一步最值得做的实验
> ```
> → 触发 Bridge 全流程：Phase 0 执行 4 轮迭代搜索（关键词扩展+同义词+机理深挖+质量筛选）→ Phase 1 提取标准化 → Quality Gate 检查数据量 → Phase 1.5 验证研究空白真实性 → 数据转换 → Phase 2 实验室分析 → Phase 3 证据分级推荐，最终输出 `TOP_PLAN.md`（唯一最优方案，每个 claim 都有引用支持）。

> **案例 2：验证某个研究方向是否值得做**
> ```
> 我想研究纳米纤维素增强 PVA 光学膜的力学性能，帮我查查文献里这个方向有没有人做过，值不值得做，如果值得的话给一个具体实验方案
> ```
> → Phase 0 搜索 CNC/PVA 复合膜文献 → Phase 1 提取配比、拉伸强度、透光率数据 → Quality Gate 检查（分析是否已有足够文献基础）→ Phase 1.5 评估"CNC 增强 PVA"这个方向的新颖度和证据等级 → 如果判定为高价值空白，Phase 2 分析现有数据趋势 → Phase 3 给出包含具体配方（如 1-5 wt% CNC）、工艺参数（溶液浇铸、60°C 干燥）、性能预测的完整实验方案，所有数据附文献引用。

> **案例 3：Quality Gate 触发用户交互**
> ```
> 帮我搜一下 PVA 光学膜在 150°C 以上热处理的数据，然后做分析
> ```
> → 搜索后发现只有 2 篇论文、提取出 5 条数据 → Quality Gate 失败（`records:5<10, papers:2<3`）→ Bridge 不会盲目执行，而是报告"文献提取数据不足"，向用户建议：扩大搜索关键词、提供本地论文、降低阈值后继续、或放弃本次分析。

> **案例 4：已有文献数据，直接做实验室分析**
> ```
> 我这里有之前提取好的 PVA 光学膜实验数据（JSON 格式），帮我生成分析报告和推荐方案
> ```
> → 直接进入 Phase 2（跳过搜索和提取），执行数据清洗 → 报告生成 → 可视化 → 实验推荐。由于没有经过文献提取 Phase（无源头引用），推荐结果的证据等级会自动标注为 D（推断），提醒用户这是基于已有数据的分析而非文献支持。

---

## 通用设计理念

### 渐进式加载 (Progressive Loading)

每个 Skill 的参考文档只有在对应模块被调用时才读取，避免一次性加载全部内容：

| 时机 | 读取内容 |
|------|---------|
| Skill 触发 | `SKILL.md` — 意图路由、模块选择 |
| 模块执行 | 对应 `references/module-N-*.md` |
| Schema 校验 | 对应 `schemas/*.json` |
| 全流程 Pipeline | `pipeline-execution.md` |

### 模块独立性与可组合性

- 每个模块可**独立运行**，输出 Schema 校验的 JSON
- 模块间通过**文件**传递数据（非内存），保证可检查性和可恢复性
- Pipeline 支持**断点续跑**（resume 模式）

### 错误处理原则

1. **单模块失败** — 不阻塞其他模块，继续执行
2. **部分成功可接受** — 有质量标注的部分输出优于编造数据的完整输出
3. **明确报告** — 错误类型、位置、建议修复方式都清晰记录
4. **不编造** — 缺失字段明确设为 `null`，从不推测或制造数据

### Pipeline Event Log

每个 Pipeline 运行都会生成 `.pipeline_events.jsonl` 事件日志：

```jsonl
{"event": "pipeline_start", "mode": "full", "timestamp": "2026-05-24T10:00:00Z"}
{"event": "module_start", "module": 1, "timestamp": "2026-05-24T10:00:01Z"}
{"event": "module_complete", "module": 1, "errors": 0, "timestamp": "2026-05-24T10:00:05Z"}
{"event": "module_error", "module": 2, "error": "Format not recognized", "timestamp": "2026-05-24T10:00:06Z"}
{"event": "pipeline_complete", "status": "partial_success", "errors": 1, "timestamp": "2026-05-24T10:00:15Z"}
```

---

## 常见问题

### 如何选择正确的 Skill？

| 用户意图 | 推荐的 Skill |
|---------|-------------|
| "帮我清洗这个 Excel 数据，生成报告" | `chem-auto-lab-skill` |
| "分析这张 FTIR 谱图" | `chem-auto-lab-skill` (Module 2) |
| "从这些论文里提取实验数据" | `domain-literature-experiment-extraction-ontology-skill` |
| "帮我把这些论文建一个知识图谱" | `domain-literature-experiment-extraction-ontology-skill` (Modules 1→2→3→4→6) |
| "搜索文献+提取数据+化学分析+生成推荐方案" | `literature-to-lab-bridge` |
| "搜索PVA光学膜文献，提取实验数据，分析趋势，推荐下一步实验" | `literature-to-lab-bridge` |

### 如何调试特定模块？

使用 `--mode` 参数运行子集：

```bash
# 只运行 chem-auto-lab 的谱图解析+报告
python scripts/run_pipeline.py --input-dir ./data/ --output-dir ./out/ --mode analyze

# 只运行文献 Skill 的知识构建 (从已有数据)
python scripts/run_pipeline.py --input-dir ./prev_output/ --output-dir ./out/ --mode knowledge-build
```

### 如何从断点继续？

```bash
# 从 Module 3 继续
python scripts/run_pipeline.py \
  --input-dir ./prev_run/ \
  --output-dir ./new_run/ \
  --mode resume \
  --resume-from 3
```