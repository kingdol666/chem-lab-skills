# Literature-to-Lab Bridge Skill — 文献到实验室的桥梁

## 概述

将 **文献数据提取** (`domain-literature-experiment-extraction-ontology-skill`) 与 **实验室分析** (`chem-auto-lab-skill`) 串联成一条自动化流水线。从搜索文献开始，提取实验数据，验证研究空白，进行实验室分析，最终输出**带证据分级的唯一最优实验方案**。

### 适用场景

- **需要同时搜索文献并做化学分析**："搜一下这个方向的文献，提取数据，分析趋势，推荐下一步实验"
- **验证研究方向**："查查文献里这个方向有没有人做过，值不值得做，给个具体实验方案"
- **已有文献数据，需要实验室分析**："这里有从论文提取的数据，帮我生成报告和推荐"

### 和另外两个 Skill 的关系

```
domain-literature-skill          chem-auto-lab-skill
(文献提取)                         (实验分析)
    │                                   │
    └──────────┬───────────┬────────────┘
               │           │
               ▼           ▼
        literature-to-lab-bridge
        (文献 → 实验室桥梁)
```

---

## 执行流程

```
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 0: 多轮迭代文献搜索                                       │
│  Round 1: 核心关键词 → 搜索 8-15 篇                              │
│  Round 2: 同义词扩展 → 搜索 10-20 篇                             │
│  Round 3: 机理深度 → 搜索 8-15 篇                                │
│  Round 4: 质量筛选 → 去重 + 排序                                 │
└────────────────────────────┬────────────────────────────────────┘
                             │ merged_corpus.json
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 1: 文献提取与标准化                                      │
│  (domain-literature-experiment-extraction-ontology-skill)       │
│  Module 1→2→3→4→7 → 输出 experiments_normalized.json             │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  QUALITY GATE (数据量检查)                                      │
│  ├── records_extracted >= 10 ?                                  │
│  ├── mean_confidence >= 0.5 ?                                   │
│  ├── unique_papers >= 3 ?                                       │
│  └── FAIL → 向用户报告，等待指令                                  │
└────────────────────────────┬────────────────────────────────────┘
                             │ PASS
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 1.5: 研究空白验证 (NEW)                                  │
│  逐个交叉验证文献中的研究空白 → 评分(新颖度+证据+价值) → 分类    │
│  high / medium / low / rejected                                  │
└────────────────────────────┬────────────────────────────────────┘
                             │ verified_gaps.json
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  DATA TRANSFORMER                                               │
│  transform_literature_to_lab.py → lab_experiments.json          │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 2: 实验室分析                                            │
│  (chem-auto-lab-skill) Module 1→4→5 (+ 2 如有谱图数据)          │
│  数据清洗 → 报告生成 → 可视化 → 实验推荐                          │
└────────────────────────────┬────────────────────────────────────┘
                             │ report.md + recommendations.json
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 3: 证据分级推荐 (NEW)                                    │
│  每个 claim 按证据分级 A/B/C/D                                  │
│  可行性评分 (设备×复杂度×时间×成本)                               │
│  → 输出唯一的 TOP_PLAN.md (最优方案)                             │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  INTEGRATED OUTPUT                                             │
│  bridge_manifest.json + verified_gaps.json + TOP_PLAN.md       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 使用方式

### 环境准备

```bash
pip install pandas numpy matplotlib scipy openpyxl seaborn jsonschema
```

### 全流程执行

```bash
python scripts/bridge_pipeline.py \
  --mode full \
  --domain "pva_bopet" \
  --search-keywords "PVA optical film, light transmittance, haze, tensile strength" \
  --min-records 10 \
  --min-confidence 0.5 \
  --min-papers 3
```

### Pipeline 模式

| 模式 | 说明 | 执行阶段 |
|------|------|:--------:|
| `full` | 完整流水线 | Phase 0→1→Gate→1.5→Transform→2→3→Finalize |
| `literature-only` | 只执行文献提取 | Phase 1 only |
| `lab-only` | 从已有数据执行分析 | Phase 2 only (需 `phase2_input/`) |
| `resume` | 从失败阶段恢复 | 指定阶段开始执行 |
| `quality-gate` | 只检查数据质量 | Gate only |
| `finalize` | 从已有输出生成 manifest | Finalize only |

### 关键脚本参数

| 脚本 | 关键参数 |
|------|---------|
| `bridge_pipeline.py` | `--mode`, `--domain`, `--search-keywords`, `--min-records`, `--min-confidence`, `--min-papers` |
| `transform_literature_to_lab.py` | `--input`, `--output`, `--domain`, `--include-spectral`, `--confidence-threshold` |
| `evidence_grader.py` | `--verified-gaps`, `--experiments`, `--literature-summary`, `--output`, `--max-recommendations` |
| `gap_verifier.py` | `--gaps`, `--experiments`, `--papers`, `--domain`, `--output` |

---

## 输出结构

```
<bridge-runs>/<bridge_id>/
│
├── bridge_manifest.json              # 完整执行追踪
├── .bridge_events.jsonl              # 事件日志
│
├── phase0_output/                    # 多轮文献搜索结果
│   ├── search_rounds.json
│   └── merged_corpus.json
│
├── phase1_output/                    # 文献提取(调用 literature skill)
│   ├── 01_literature/
│   ├── 02_extracted/
│   ├── 03_normalized/
│   │   ├── experiments_normalized.json
│   │   └── experiments.csv
│   ├── 04_provenance/
│   └── 07_summary/
│       └── literature_summary.json
│
├── phase1.5_output/                  # 研究空白验证结果
│   └── verified_gaps.json
│
├── phase2_input/                     # 转换后的实验数据
│   └── lab_experiments.json
│
├── phase2_output/                    # 实验室分析结果
│   ├── 01_cleaned/
│   ├── figures/
│   ├── report.md
│   └── recommendations.json
│
└── phase3_output/                    # 证据分级推荐
    ├── evidence_graded_recommendations.json
    └── TOP_PLAN.md                   # ⭐ 唯一最优方案
```

---

## 调用案例

### 案例 1：完整 Bridge 全流程

> **用户提问**: "帮我搜索 PVA 光学膜的文献，提取实验数据，分析其中的趋势，然后推荐下一步最值得做的实验"
>
> **Skill 响应**:
> 1. **Phase 0** — 执行 4 轮迭代搜索（核心→同义词→机理→质量筛选），去重后得到论文列表
> 2. **Phase 1** — 提取+标准化每篇论文的实验数据，输出 `experiments_normalized.json`
> 3. **Quality Gate** — 检查（记录数≥10, 置信度≥0.5, 论文数≥3），通过后自动继续
> 4. **Phase 1.5** — 验证研究空白真实性，筛除虚假空白
> 5. **Data Transformer** — 将文献格式转换为实验分析格式
> 6. **Phase 2** — 数据清洗→报告生成→可视化→实验推荐
> 7. **Phase 3** — 证据分级（A/B/C/D），可行性评分，输出唯一最优方案 `TOP_PLAN.md`
> 8. 最终输出：`TOP_PLAN.md` 中每个 claim 都附有文献引用和证据分级

### 案例 2：验证研究方向

> **用户提问**: "我想研究纳米纤维素增强 PVA 光学膜的力学性能，帮查查文献里这个方向有没有人做过，值不值得做"
>
> **Skill 响应**:
> 1. Phase 0 搜索 CNC（纳米纤维素晶须）/PVA 复合膜相关文献
> 2. Phase 1 提取配比、拉伸强度、透光率等数据
> 3. Quality Gate 检查是否存在足够文献基础
> 4. Phase 1.5 评估"CNC 增强 PVA"的新颖度（7/10）和证据等级（B）
> 5. 判定为高价值空白 → Phase 2 分析现有数据趋势
> 6. Phase 3 输出具体实验方案：1-5 wt% CNC 配比、溶液浇铸法、60°C 干燥
> 7. 每个推荐的参数值都附有文献引用和证据分级

### 案例 3：Quality Gate 触发用户交互

> **用户提问**: "帮我搜一下 PVA 光学膜在 150°C 以上热处理的数据"
>
> **Skill 响应**:
> - 搜索后发现只有 2 篇论文、提取出 5 条数据
> - **Quality Gate 失败**：`records=5<10, papers=2<3`
> - 不会盲目执行，而是报告：
>   ```
>   文献提取数据不足 (5条 < 10条阈值)。
>   建议：①扩大搜索关键词；②提供更多本地论文；
>         ③降低阈值继续；④放弃本次分析。
>   ```
> - 等待用户指令后再决定下一步

### 案例 4：已有文献数据做实验室分析

> **用户提问**: "我这里有之前提取好的 PVA 光学膜实验数据（JSON 格式），帮我生成分析报告和推荐方案"
>
> **Skill 响应**:
> - 跳过 Phase 0/1，直接进入 Data Transformer + Phase 2
> - **证据降级**：由于没有经过文献 Phase（无源头引用），推荐结果自动标注为 D（推断）
> - 标注提示：`⚠️ 推断 — 无文献直接支持`
> - 报告内容基于已有数据生成，但不具备文献证据的支撑

---

## 证据分级体系

| 等级 | 定义 | 来源要求 | 展示方式 |
|:----:|------|---------|---------|
| **A** | 直接实验证据 | ≥3 篇论文有一致数据 | 正常显示 |
| **B** | 强间接证据 | ≥1 篇有相关数据 | 正常显示 |
| **C** | 弱间接/理论 | 机理已知但无直接数据 | 标注"理论推断" |
| **D** | 纯推断 | 化学直觉 | ⚠️ 标注"无文献直接支持" |
| **F** | 纯猜测 | 无依据 | **排除，不出现在输出中** |

### 反编造保障

- 每个 claim 必须锚定至少一个文献来源
- 无法锚定的 claim 最低标注为 D，明确告知"无文献直接支持"
- F 级 claim **永远不会**出现在输出中

---

## 细节参考

| 场景 | 读取文件 |
|------|---------|
| 需要理解 Bridge 架构 | `SKILL.md` |
| 需要全流程编排细节 | `pipeline-execution.md` |
| 需要了解文献提取模块 | `../domain-literature-experiment-extraction-ontology-skill/SKILL.md` |
| 需要了解实验分析模块 | `../chem-auto-lab-skill/SKILL.md` |
| 需要数据转换规则 | `python scripts/transform_literature_to_lab.py --help` |