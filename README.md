# Chem-Skill — 化学实验室智能分析 AI Skills 套件

> **运行环境**: Claude Code + oh-my-claudecode
> **执行模型**: LLM-native 流水线（通过 Claude Code 内置工具全自动执行）

## 概览

本仓库包含 **3 个可独立或组合使用**的化学领域 AI Skill，安装于 Claude Code 的 oh-my-claudecode 框架下。每个 Skill 采用 **渐进式加载（Progressive Loading）、模块化设计、Schema 校验输出**的架构。

输入领域关键词 → 自动搜索文献 → 提取实验数据 → 分析验证 → 输出可靠课题方案，**端到端全自动化执行**。

| Skill | 定位 | 一句话描述 |
|-------|------|-----------|
| [chem-auto-lab-skill](#1-chem-auto-lab-skill) | 化学实验数据处理 | 清洗数据、谱图解析、生成报告、推荐下一步实验 |
| [domain-literature-experiment-extraction-ontology-skill](#2-domain-literature-experiment-extraction-ontology-skill) | 文献数据挖掘 | 从论文提取实验参数、构建知识图谱、文献综述 |
| [literature-to-lab-bridge](#3-literature-to-lab-bridge) | 文献→实验室桥梁 | 搜索文献→提取数据→化学分析→输出带证据分级的课题方案 |

### 架构关系

```
┌─────────────────────────────────────────────────────────────────────────┐
│               domain-literature-experiment-extraction-ontology-skill      │
│                              (文献数据挖掘)                               │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌────────┐ │
│  │ 文献搜索 │→│ 实验提取 │→│ 标准化   │→│ 溯源     │→│ 本体构建 │→│ 综述    ││
│  │ Module1 │ │ Module2 │ │ Module3 │ │ Module4 │ │ Module6 │ │ Module7│ │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘ └────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      literature-to-lab-bridge                             │
│                       (文献→实验室桥梁)                                    │
│  Phase 0 → Phase 1 → Phase 1.5 → Transform → Phase 2 → Phase 3          │
│  多轮搜索    文献提取    空白验证     格式转换     实验室分析    证据分级  │
│                              │                                            │
└──────────────────────────────┼────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                       chem-auto-lab-skill                                 │
│                         (实验室数据处理)                                  │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │
│  │ 数据清洗  │→│ 谱图解析  │→│ 笔记结构化 │→│ 报告生成  │→│ 实验推荐  ││
│  │ Module1  │ │ Module2  │ │ Module3  │ │ Module4  │ │ Module5  │      │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘      │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 项目结构

```
chem-skill/
├── .claude/
│   └── skills/
│       ├── chem-auto-lab-skill/                          # Skill 1：实验数据自动处理
│       │   ├── SKILL.md                                  # 主技能文件（意图路由、模块选择）
│       │   ├── pipeline-execution.md                     # 全流水线编排细节
│       │   ├── references/                               # 各模块详细参考文档
│       │   ├── schemas/                                  # JSON Schema 校验文件
│       │   ├── scripts/                                  # Python 辅助脚本
│       │   └── README.md
│       ├── domain-literature-experiment-extraction-ontology-skill/  # Skill 2：文献挖掘
│       │   ├── SKILL.md
│       │   ├── pipeline-execution.md
│       │   ├── references/
│       │   ├── schemas/
│       │   ├── scripts/
│       │   ├── assets/                                   # 领域词库、单位转换表
│       │   └── templates/                                # 提取配置模板
│       └── literature-to-lab-bridge/                     # Skill 3：文献→实验室桥梁
│           ├── SKILL.md
│           ├── pipeline-execution.md
│           ├── scripts/
│           ├── schemas/
│           └── README.md
│
├── bridge-runs/                                          # 实际运行输出目录
│   └── bridge_20260602_pva_optical_film/                 # PVA光学膜课题搜索示例
│       ├── bridge_manifest.json                          # 完整执行追踪
│       ├── phase0_output/search_rounds.json              # 多轮搜索记录
│       ├── phase1_output/                                # 文献提取结果（10篇论文, 35条实验）
│       └── phase3_output/TOP_PLAN.md                     # ⭐ 最终课题方案
│
└── README.md
```

---

## 执行模型：LLM-native 流水线

本项目的 **核心执行机制** 不依赖 Python 脚本，而是通过 **Claude Code 内置工具** 全自动执行：

```
用户输入 "搜索 PVA 光学膜的文献，提取数据，推荐下一步实验"
    │
    ▼
┌──────────────────────────────────────────────────────────┐
│  1. Skill 自动触发（基于关键词匹配）                        │
│     → 加载 literature-to-lab-bridge SKILL.md              │
│     → 路由到 Full Pipeline 模式                           │
├──────────────────────────────────────────────────────────┤
│  2. Phase 0：多轮文献搜索                                 │
│     4-5 轮 WebSearch，逐步精准                            │
├──────────────────────────────────────────────────────────┤
│  3. Phase 1：文献提取（LLM 驱动）                          │
│     WebFetch 获取全文 → LLM 提取实验数据                      │
├──────────────────────────────────────────────────────────┤
│  4. Phase 1.5：空白验证（LLM 驱动）                        │
│     交叉引用文献 → 评分：新颖度+证据+价值                      │
├──────────────────────────────────────────────────────────┤
│  5. Phase 3：证据分级推荐（LLM 驱动）                      │
│     每个 claim 锚定文献 → 输出 TOP_PLAN.md                 │
└──────────────────────────────────────────────────────────┘
```

每个 Phase 的输出是 **schema-validated JSON**，存储在 `bridge-runs/<run_id>/` 目录下，可检查、可恢复。

---

## 快速开始

### 环境要求

- **Claude Code** — AI 编程助手
- **oh-my-claudecode**（推荐）— 多 Agent 编排框架
- 无需安装 Python 依赖（脚本为辅助工具，非核心执行路径）

### 安装

完整的安装步骤（支持 Claude Code、oh-my-claudecode、OpenClaw、Hermes）见 ➡️ **[INSTALL.md](INSTALL.md)**

一键安装（Claude Code / oh-my-claudecode）：

```bash
# 在项目目录下执行
git clone https://github.com/kingdol666/chem-lab-skills.git /tmp/chem-skills
cp -r /tmp/chem-skills/.claude/skills/* .claude/skills/
rm -rf /tmp/chem-skills
```

### 使用方式

在 Claude Code 中通过自然语言触发：

| 任务 | 输入示例 | 触发的 Skill |
|------|---------|-------------|
| 数据分析 | "帮我分析这个文件夹里的化学实验数据" | `chem-auto-lab-skill` |
| 谱图解析 | "帮我解析这张 FTIR 谱图" | `chem-auto-lab-skill` |
| 文献提取 | "从这些论文里提取实验数据" | `domain-literature-experiment-extraction-ontology-skill` |
| 文献→实验室 | "搜索 PVA 光学膜的文献，提取数据，分析趋势，推荐下一步实验" | `literature-to-lab-bridge` |

或通过 `/oh-my-claudecode:<skill-name>` 直接调用。

### 实际运行输出示例

本项目已包含一次完整的 Bridge Pipeline 运行结果：

```
bridge-runs/bridge_20260602_pva_optical_film/
├── bridge_manifest.json                # 运行报告
├── phase0_output/search_rounds.json    # 5轮搜索，22篇论文
├── phase1_output/                      # 10篇论文，35条实验数据
├── phase1.5_output/verified_gaps.json  # 5个gap，验证通过4个
├── phase3_output/TOP_PLAN.md           # ⭐ 最优实验方案
└── phase3_output/evidence_graded_recommendations.json
```

**最终输出** ⭐ `TOP_PLAN.md` 是一份可直接用作课题方案的完整文档：

| 内容 | 说明 |
|------|------|
| Executive Summary | 一句话课题描述 |
| 证据锚定表 | 所有 claim 注明了 A/B/C/D 等级和文献来源 |
| 具体实验方案 | 材料清单、设备需求、7组实验矩阵、操作步骤 |
| 预期结果 | 各性能指标的预期范围、置信度 |
| 风险评估 | 5个主要风险的缓解策略 |
| 成本估算 | 总计约 ¥16,000，8周完成 |

---

## 三大 Skill 详解

### 1. chem-auto-lab-skill — 化学实验数据自动处理

**适用场景**：用户提供化学实验室数据文件，需要清洗、解析、报告、推荐。

**5 个独立模块**：

| 模块 | 功能 | 典型输入 |
|------|------|---------|
| Module 1 | 数据清洗、缺失值填充、异常值检测 | `.xlsx`, `.csv` |
| Module 2 | FTIR/Raman/UV-Vis/NMR/HPLC 谱图解析 | `.csv`, `.jdx` |
| Module 3 | 非结构化实验笔记 → 结构化参数 | `.txt`, `.md` |
| Module 4 | 分析报告 + 可视化图表 | 合并后的 JSON 数据 |
| Module 5 | 参数空间分析 → 实验推荐 | 清洗后的数据 |

详见 [chem-auto-lab-skill/README.md](.claude/skills/chem-auto-lab-skill/README.md)。

### 2. domain-literature-experiment-extraction-ontology-skill — 文献数据挖掘

**适用场景**：从科学论文中系统性地提取实验参数、构建知识图谱、识别研究空白。

**7 个模块 + 完整 Pipeline**：

| 模块 | 功能 | 关键输出 |
|------|------|---------|
| Module 1 | 多源文献搜索（WebSearch）+ 去重 | `source_manifest.json` |
| Module 2 | LLM 驱动提取表格/正文实验参数 | `experiments_raw.json` |
| Module 3 | 单位归一化（mil→μm）+ 同义词映射 | `experiments_normalized.json` |
| Module 4 | 每个数据点绑定来源/页码/原文 | `provenance.json` |
| Module 5 | 科学原理解释 | `explanations.md` |
| Module 6 | 材料-工艺-性能本体构建 | `ontology.json` / `.owl` |
| Module 7 | 趋势分析 + 研究空白识别 | `literature_summary.md` |

**预置领域知识**：PVA/BOPET 光学膜（材料、添加剂、工艺、性能、仪器、单位转换）。可适配其他领域（替换 `assets/` 目录下的词库即可）。

详见 [domain-literature-experiment-extraction-ontology-skill/README.md](.claude/skills/domain-literature-experiment-extraction-ontology-skill/README.md)。

### 3. literature-to-lab-bridge — 文献→实验室桥梁

**适用场景**：同时需要文献搜索+数据提取+化学分析的端到端任务，输出带证据分级的课题方案。

**执行流程**：

```
Phase 0: 多轮文献搜索 (4-5轮，逐步精准)         搜索+去重+质量筛选
    │
    ▼
Phase 1: 文献提取+标准化                        实验数据提取+归一化
    │
    ▼
Quality Gate: 数据量检查                        记录≥10且置信度≥0.5且论文≥3
    │  FAIL → 报告用户，等待指令
    │  PASS → 自动继续
    ▼
Phase 1.5: 研究空白验证                         交叉验证，排除假空白
    │
    ▼
Data Transformer: 格式转换                      文献Schema → 实验室Schema
    │
    ▼
Phase 2: 实验室分析                             数据清洗→报告→推荐
    │
    ▼
Phase 3: 证据分级推荐                           每个claim锚定文献→选出最优方案
    │
    ▼
输出: TOP_PLAN.md                               唯一最优课题方案
```

**反编造保障机制**（Anti-Fabrication Guarantees）：

- Quality Gate：数据不足时不盲目执行
- Phase 1.5：交叉验证研究空白真实性
- Phase 3：每个 claim 必须锚定文献证据，F 级排除

详见 [literature-to-lab-bridge/README.md](.claude/skills/literature-to-lab-bridge/README.md)。

---

## 核心设计原则

### 1. 渐进式加载（Progressive Loading）

每个 Skill 的参考文档只在对应模块被调用时读取，避免一次性加载全部内容：

| 时机 | 读取内容 |
|------|---------|
| Skill 被自然语言触发 | `SKILL.md` — 意图路由 |
| 特定模块执行 | 对应 `references/module-N-*.md` |
| Schema 校验 | 对应 `schemas/*.json` |
| 全流程 Pipeline | `pipeline-execution.md` |

### 2. 模块独立性与可组合性

- 每个模块可**独立运行**，输出 schema-validated JSON
- 模块间通过**文件**传递数据，保证可检查性和断点续跑
- Pipeline 支持 **resume 模式**

### 3. 不编造（No Fabrication）

- 缺失字段显式设为 `null` — 从不使用 0、空字符串、"N/A" 替代
- 置信度 0.0（无证据）至 1.0（直接引用表格）
- 部分完成+质量标注 优于 完整输出+编造数据

### 4. Schema 驱动输出

- 所有输出使用 JSON Schema 校验
- JSON key 使用 `snake_case` 英文
- 数值字段包含 `value`、`unit`、`unit_normalized`
- 所有输出带 `metadata` 溯源信息（skill 版本、时间戳、输入源）

---

## 参考文献

每个 Skill 的 `references/` 目录下有详细的模块参考文档：

| 文件 | 内容 |
|------|------|
| `references/module-1-*.md` | 各模块详细逻辑和规则 |
| `schemas/*.json` | 输出格式校验标准 |
| `assets/*.json` | 领域词库、同义词映射、单位转换 |
| `templates/*.json` | 提取配置模板 |

---

## 示例运行

本仓库 `bridge-runs/` 目录下包含一次完整的 Bridge Pipeline 运行结果：

- **输入**：搜索 PVA 光学膜文献
- **过程**：5 轮搜索 → 22 篇论文 → 10 篇可提取 → 35 条实验数据
- **输出**：`TOP_PLAN.md` — 反应性染料掺杂 PVA 纳米复合光学偏振膜课题方案

详细输出参见 [bridge-runs/bridge_20260602_pva_optical_film/](bridge-runs/bridge_20260602_pva_optical_film/)。

---

## 常见问题

### Q: 这些 Skill 和普通 Python 脚本有什么区别？

**A**: 核心区别在于执行模型。Python 脚本是**辅助工具**（处理批量文件转换等确定性任务），而**主流程**由 LLM 通过 Claude Code 内置工具（WebSearch、WebFetch、LLM 推理）全自动执行。SKILL.md 定义的是"LLM 如何理解和执行任务"的指令集，而非传统程序代码。

### Q: 如何适配其他研究领域？

对应 `domain-literature-experiment-extraction-ontology-skill` 的 `assets/` 目录：
- 替换 `pva_bopet_vocabulary.json` 为目标领域的材料/工艺/性能词库
- 替换 `unit_conversions.json` 为目标领域常用单位
- 替换 `synonym_map.json` 为目标领域的同义词映射
- 在 SKILL.md 中更新 Domain Configuration 部分
- 调整 search query 模板为对应领域

### Q: 运行结果能增量更新吗？

能。Pipeline 设计支持增量模式：
1. 新论文只运行 Module 1-4
2. 合并到已有数据集（基于 `experiment_id` 去重）
3. 重新运行 Module 5-7 更新分析结果
4. `_merge_metadata` 记录合并来源和时间戳

### Q: 如何确保推荐课题的可靠性？

三层保障：
1. **Quality Gate**：数据量不足时拒绝执行，报告用户
2. **Phase 1.5 Gap Verification**：排除"假空白"（已有文献覆盖但搜索遗漏的）
3. **Phase 3 Evidence Grading**：每个 claim 按 A/B/C/D/F 分级，F 级排除

---

## 许可

内部工具，仅供实验室研究使用。
