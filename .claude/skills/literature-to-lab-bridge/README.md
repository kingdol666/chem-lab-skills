# Literature-to-Lab Bridge Skill — 文献到实验室的桥梁

> **执行模型**: LLM-native，端到端全自动执行（WebSearch → WebFetch → LLM 推理 → 结构化输出）
> **完整文档**: 参见 [SKILL.md](SKILL.md) 和 [pipeline-execution.md](pipeline-execution.md)

## 概述

将 **文献数据挖掘** (`domain-literature-experiment-extraction-ontology-skill`) 与 **实验室分析** (`chem-auto-lab-skill`) 串联成一条全自动化流水线。

**从搜索文献开始，到输出带证据分级的唯一最优课题方案结束。**

### 核心特色

- 🔄 **端到端全自动** — 输入关键词，输出可执行的课题方案
- 🛡️ **反编造保障** — Quality Gate + Gap 验证 + 证据分级，三层防护
- 📋 **唯一最优方案** — 不是列出 5 个模糊方向，而是输出 **一个** 具体的、可执行的实验方案（`TOP_PLAN.md`）
- 🔗 **每个 claim 锚定文献** — A/B/C/D/F 证据分级，F 级直接排除

---

## 适用场景

| 你想做什么 | 用这个 Skill？ |
|-----------|:------------:|
| "搜索文献，提取数据，分析趋势，推荐课题" | ✅ |
| "查查这个方向有人做过没，给个具体实验方案" | ✅ |
| "从论文提取数据，清洗后生成报告和推荐" | ✅ |
| 只搜索文献/提取数据 | ❌ → 用 `domain-literature` skill |
| 只分析已有的化学实验数据 | ❌ → 用 `chem-auto-lab` skill |

### 和另外两个 Skill 的关系

```
domain-literature-skill                    chem-auto-lab-skill
(文献搜索 + 实验提取)                        (数据清洗 + 报告 + 推荐)
           │                                       │
           └──────────────┬────────────────────────┘
                          │
                          ▼
               literature-to-lab-bridge
               (编排 + 验证 + 证据分级)
               → 输出 TOP_PLAN.md
```

---

## 执行流程

```
┌───────────────────────────────────────────────────────────────────┐
│  PHASE 0: 多轮迭代文献搜索                                        │
│  Round 1: 核心关键词 → 8-15 篇                                    │
│  Round 2: 同义词扩展 → 10-20 篇                                   │
│  Round 3: 机理深度 → 8-15 篇                                      │
│  Round 4: 质量筛选 → 去重 + 排序                                   │
│  Round 5: 查漏补缺 → 最终合并                                      │
└──────────────────────────────┬────────────────────────────────────┘
                               │ merged_corpus.json
                               ▼
┌───────────────────────────────────────────────────────────────────┐
│  PHASE 1: 文献提取与标准化                                        │
│  调用 domain-literature-experiment-extraction-ontology-skill      │
│  Module 1→2→3→4→7 → 输出 experiments_normalized.json              │
└──────────────────────────────┬────────────────────────────────────┘
                               │
                               ▼
┌───────────────────────────────────────────────────────────────────┐
│  QUALITY GATE: 数据量检查                                         │
│  ├── records_extracted >= 10 ?                                    │
│  ├── mean_confidence >= 0.5 ?                                     │
│  └── unique_papers >= 3 ?                                         │
│  FAIL → 报告用户，等待指令（不盲目执行）                            │
│  PASS → 自动继续                                                  │
└──────────────────────────────┬────────────────────────────────────┘
                               │
                               ▼
┌───────────────────────────────────────────────────────────────────┐
│  PHASE 1.5: 研究空白验证                                          │
│  逐个交叉验证研究空白的真实性                                       │
│  评分: 新颖度(0-10) + 证据等级(A-E) + 研究价值(0-10)                │
│  分类: high / medium / low / rejected                              │
│  → 排除假空白（已有文献覆盖但搜索遗漏的）                            │
└──────────────────────────────┬────────────────────────────────────┘
                               │ verified_gaps.json
                               ▼
┌───────────────────────────────────────────────────────────────────┐
│  DATA TRANSFORMER: 格式转换                                        │
│  文献 Schema → 实验室 Schema（字段映射、光谱检测、置信度筛选）      │
└──────────────────────────────┬────────────────────────────────────┘
                               │ lab_experiments.json
                               ▼
┌───────────────────────────────────────────────────────────────────┐
│  PHASE 2: 实验室分析                                              │
│  调用 chem-auto-lab-skill: Module 1→4→5 (+2 如有谱图数据)          │
│  数据清洗 → 报告生成 → 可视化 → 实验推荐                            │
└──────────────────────────────┬────────────────────────────────────┘
                               │ report.md + recommendations.json
                               ▼
┌───────────────────────────────────────────────────────────────────┐
│  PHASE 3: 证据分级推荐                                            │
│  每个 claim 按 A/B/C/D 分级 (F 级排除)                             │
│  可行性评分 = 设备×复杂度×时间×成本 (各25%权重)                     │
│  → 输出唯一最优方案 TOP_PLAN.md                                    │
└──────────────────────────────┬────────────────────────────────────┘
                               │
                               ▼
                     ┌─────────────────┐
                     │   TOP_PLAN.md   │
                     │   唯一最优方案   │
                     └─────────────────┘
```

---

## 使用方式

在 Claude Code 中通过自然语言触发，Skill 自动编排全流程：

### 典型触发

| 用户说 | 执行 |
|-------|------|
| "搜索PVA光学膜文献，提取数据，分析趋势，推荐下一步实验" | Full Pipeline |
| "帮我查查这个方向有人做过没，给个具体实验方案" | Full Pipeline |
| "从论文提取数据，清洗后生成报告和推荐" | Full Pipeline |

### Pipeline 模式

| 模式 | 说明 | 执行阶段 |
|------|------|:--------:|
| `full` | 完整流水线 | Phase 0→1→Gate→1.5→Transform→2→3 |
| `literature-only` | 只执行文献提取 | Phase 1 only |
| `lab-only` | 从已有数据执行分析 | Phase 2 (需 `phase2_input/`) |
| `resume` | 从失败阶段恢复 | 指定阶段开始 |
| `quality-gate` | 只检查数据质量 | Gate only |

---

## 证据分级体系

| 等级 | 定义 | 来源要求 | 展示方式 |
|:----:|------|---------|---------|
| **A** | 直接实验证据 | ≥3 篇论文有一致数据 | ✅ 正常显示 |
| **B** | 强间接证据 | ≥1 篇有相关数据 | ✅ 正常显示 |
| **C** | 弱间接/理论 | 机理已知但无直接数据 | ⚡ 标注"理论推断" |
| **D** | 纯推断 | 化学直觉、相邻体系 | ⚠️ 标注"无文献直接支持" |
| **F** | 纯猜测 | 无依据 | ❌ **排除，不出现在输出中** |

### 反编造保障

- **Quality Gate**: 数据不足时拒绝执行，报告用户等待指令
- **Phase 1.5**: 交叉验证每个"研究空白"的真实性，排除假空白
- **Phase 3**: 每个 claim 必须锚定至少一个文献来源，F 级 claim 永远不输出

### 可行性评分

```
可行性 = 设备得分(25%) + 复杂度得分(25%) + 时间得分(25%) + 成本得分(25%)
         standard(9)    simple(9)       fast(9)         low(9)
         specialized(6) moderate(6)     moderate(6)     moderate(6)
         advanced(3)    complex(3)      long(3)         high(3)
         custom(1)      very_complex(1) very_long(1)    very_high(1)

最终方案选择: max(证据等级评分 × 可行性评分 × 新颖度评分)
```

---

## 输出目录结构

```
<bridge-runs>/<bridge_id>/
├── bridge_manifest.json              # 完整执行追踪（所有 phase 状态+统计）
├── .bridge_events.jsonl              # 事件日志
│
├── phase0_output/                    # 多轮文献搜索
│   └── search_rounds.json            # 每轮搜索查询+结果数+去重统计
│
├── phase1_output/                    # 文献提取（调用 literature skill）
│   ├── 01_literature/
│   │   └── source_manifest.json      # 文献元数据（标题、作者、DOI、摘要）
│   ├── 02_extracted/
│   │   └── experiments_raw.json      # 原始提取数据
│   ├── 03_normalized/
│   │   ├── experiments_normalized.json  # 标准化后数据（核心）
│   │   └── experiments.csv
│   ├── 04_provenance/
│   │   └── provenance.json           # 溯源表
│   ├── 07_summary/
│   │   └── literature_summary.json   # 趋势+空白分析
│   └── run_summary.json              # Phase 1 统计
│
├── phase1.5_output/                  # 研究空白验证
│   └── verified_gaps.json            # 验证后的空白+评分
│
├── phase2_input/                     # 转换后的数据
│   └── lab_experiments.json
│
├── phase2_output/                    # 实验室分析（调用 chem-auto-lab skill）
│   ├── 01_cleaned/
│   ├── figures/
│   ├── report.md
│   └── recommendations.json
│
└── phase3_output/                    # 证据分级推荐
    ├── evidence_graded_recommendations.json   # 候选方案（Top 3）
    └── TOP_PLAN.md                            # ⭐ 唯一最优方案
```

### TOP_PLAN.md 内容结构

| 章节 | 内容 |
|------|------|
| Executive Summary | 一句话课题描述 |
| 证据锚定表 | 每个 claim 的 A/B/C/D 等级 + 文献引用 |
| 实验方案 | 材料清单、设备需求、实验矩阵、操作步骤 |
| 预期结果 | 各性能指标预期范围 + 置信度 |
| 风险评估 | 主要风险 + 缓解策略 |
| 成本与时间 | 总预算、周期估算 |

---

## 调用案例

### 案例 1：完整 Bridge 全流程

> **用户**: "帮我搜索 PVA 光学膜的文献，提取实验数据，分析趋势，推荐下一步最值得做的实验"
>
> Phase 0 执行 5 轮迭代搜索 → Phase 1 提取标准化 → Quality Gate 检查数据量 → Phase 1.5 验证空白真实性 → Phase 2 实验室分析 → Phase 3 证据分级 → 输出 `TOP_PLAN.md`（每个 claim 附文献引用）

### 案例 2：验证研究方向

> **用户**: "我想研究纳米纤维素增强 PVA 光学膜，查查有没有人做过，给个具体方案"
>
> Phase 0 搜索 CNC/PVA 文献 → Phase 1 提取配比/强度/透光率 → Phase 1.5 评估新颖度(7/10)+证据等级(B) → 判定高价值空白 → Phase 3 输出具体实验方案（1-5 wt% CNC，溶液浇铸，60°C 干燥）

### 案例 3：Quality Gate 触发用户交互

> **用户**: "帮我搜一下 PVA 光学膜在 150°C 以上热处理的数据"
>
> 搜索后发现 2 篇论文、5 条数据 → **Quality Gate 失败**（`records:5<10, papers:2<3`）→ 报告"文献提取数据不足"，建议：扩大关键词 / 提供更多论文 / 降低阈值 / 放弃。等待用户指令。

---

## 详细文档索引

| 文档 | 内容 |
|------|------|
| [SKILL.md](SKILL.md) | 主技能文件：Bridge 架构、Phase 编排、数据转换规则 |
| [pipeline-execution.md](pipeline-execution.md) | 全流水线细节：初始化、质量门、错误恢复、事件日志 |
| [../domain-literature-experiment-extraction-ontology-skill/SKILL.md](../domain-literature-experiment-extraction-ontology-skill/SKILL.md) | Phase 1 使用的文献提取 Skill |
| [../chem-auto-lab-skill/SKILL.md](../chem-auto-lab-skill/SKILL.md) | Phase 2 使用的实验室分析 Skill |
| [schemas/](schemas/) | Bridge 输出的 JSON Schema |
