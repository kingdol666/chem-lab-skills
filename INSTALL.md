# Chem-Skill 安装指南

> **仓库地址**: https://github.com/kingdol666/chem-lab-skills.git
> **适用版本**: v1.0.0
> **支持框架**: Claude Code, oh-my-claudecode, OpenClaw, Hermes

---

## 目录

- [前置要求](#前置要求)
- [方式一：Claude Code 原生安装（最简单）](#方式一claude-code-原生安装最简单)
- [方式二：oh-my-claudecode 安装（推荐）](#方式二oh-my-claudecode-安装推荐)
- [方式三：OpenClaw 安装](#方式三openclaw-安装)
- [方式四：Hermes 安装](#方式四hermes-安装)
- [安装验证](#安装验证)
- [常见问题](#常见问题)

---

## 前置要求

### 基本要求

| 项目 | 要求 | 说明 |
|------|------|------|
| Claude Code | ≥ 最新稳定版 | AI 编程助手，[安装指南](https://docs.anthropic.com/en/docs/claude-code) |
| GitHub 访问 | 有 | 用于 clone 仓库 |
| 网络 | 有 | 文献搜索需要 WebSearch 功能 |

### 可选依赖

以下工具不是必须的（主流程由 LLM-native 执行），但运行辅助脚本时会用到：

```bash
# 数据处理的 Python 脚本（辅助功能）
pip install pandas numpy matplotlib scipy openpyxl seaborn jsonschema
```

---

## 方式一：Claude Code 原生安装（最简单）

直接将 Skill 文件放到 Claude Code 的 skill 目录下，Claude Code 会自动识别。

### 步骤

```bash
# 1. 进入你的项目目录
cd your-project/

# 2. 创建 skills 目录（如果不存在）
mkdir -p .claude/skills/

# 3. 下载 skill 文件
# 方式 A：直接从仓库下载具体 skill
curl -L https://github.com/kingdol666/chem-lab-skills/archive/main.tar.gz | \
  tar xz --strip-components=1 -C .claude/skills/ \
  */chem-auto-lab-skill */domain-literature-experiment-extraction-ontology-skill */literature-to-lab-bridge

# 方式 B：完整 clone 仓库后复制
git clone https://github.com/kingdol666/chem-lab-skills.git /tmp/chem-skills
cp -r /tmp/chem-skills/.claude/skills/* .claude/skills/
rm -rf /tmp/chem-skills
```

### 文件结构

安装后，你的项目目录结构如下：

```
your-project/
├── .claude/
│   └── skills/
│       ├── chem-auto-lab-skill/                          # 实验数据自动处理
│       │   ├── SKILL.md
│       │   ├── pipeline-execution.md
│       │   ├── references/module-1-data-cleaning.md
│       │   ├── references/module-2-spectroscopy.md
│       │   ├── references/module-3-log-structuring.md
│       │   ├── references/module-4-report-generation.md
│       │   ├── references/module-5-experiment-recommend.md
│       │   ├── schemas/experiment_record.schema.json
│       │   ├── schemas/spectroscopy_output.schema.json
│       │   ├── schemas/recommendation.schema.json
│       │   └── schemas/report.schema.json
│       │
│       ├── domain-literature-experiment-extraction-ontology-skill/  # 文献数据挖掘
│       │   ├── SKILL.md
│       │   ├── pipeline-execution.md
│       │   ├── references/module-1-literature-acquisition.md
│       │   ├── references/module-2-experiment-extraction.md
│       │   ├── references/module-3-data-normalization.md
│       │   ├── references/module-4-evidence-traceability.md
│       │   ├── references/module-5-explanation-generation.md
│       │   ├── references/module-6-ontology-modeling.md
│       │   ├── references/module-7-literature-summary.md
│       │   ├── schemas/*.json
│       │   ├── assets/pva_bopet_vocabulary.json
│       │   ├── assets/unit_conversions.json
│       │   ├── assets/synonym_map.json
│       │   └── templates/extraction_config_template.json
│       │
│       └── literature-to-lab-bridge/                      # 文献→实验室桥梁
│           ├── SKILL.md
│           ├── pipeline-execution.md
│           ├── schemas/bridge_manifest.schema.json
│           └── scripts/*.py
│
└── .claude/CLAUDE.md                      # （可选）项目配置
```

### 验证

安装完成后，在 Claude Code 中输入以下任意关键词即可触发：

```
帮我分析这个文件夹里的化学实验数据    → chem-auto-lab-skill
帮我解析这张 FTIR 谱图               → chem-auto-lab-skill (Module 2)
从这些论文里提取实验数据             → domain-literature-experiment-extraction-ontology-skill
搜索PVA光学膜文献，分析趋势，推荐实验 → literature-to-lab-bridge
```

---

## 方式二：oh-my-claudecode 安装（推荐）

[oh-my-claudecode](https://github.com/superanos/oh-my-claudecode)（OMC）是 Claude Code 的多 Agent 编排框架，Skill 可以自动被发现和注册。

### 步骤

```bash
# 1. 安装 oh-my-claudecode（如未安装）
# 在 Claude Code 中运行：
/oh-my-claudecode:omc-setup

# 2. 将 skill 复制到 OMC 的 skill 目录
# 先确认 OMC 的路径（通常为 ~/.claude/plugins/oh-my-claudecode/）
# 或直接放入项目 .claude/skills/（OMC 会自动扫描）
cd your-project/
mkdir -p .claude/skills/

# 3. Clone chem-skill 仓库
git clone https://github.com/kingdol666/chem-lab-skills.git /tmp/chem-skills
cp -r /tmp/chem-skills/.claude/skills/* .claude/skills/
rm -rf /tmp/chem-skills

# 4. 纳入版本控制（可选）
echo ".claude/skills/" >> .gitignore   # 如果不想跟踪 skill 文件
```

### 通过 OMC 手动调用

安装后，可以通过 OMC 的 slash command 直接调用：

```
/oh-my-claudecode:chem-auto-lab-skill 帮我分析实验数据
/oh-my-claudecode:literature-to-lab-bridge 搜索PVA光学膜文献
```

### OMC 钩子触发

OMC 的钩子系统会自动检测你的输入关键词并触发对应的 Skill：

```
"清洗数据"         → 自动触发 chem-auto-lab-skill Module 1
"提取实验数据"     → 自动触发 domain-literature skill Module 2
"搜索文献+分析"    → 自动触发 literature-to-lab-bridge
```

---

## 方式三：OpenClaw 安装

[OpenClaw](https://github.com/TheAppleTucker/open-claw) 是一个本地运行的 AI 助手，通过 **Agent Skill (SKILL.md)** 格式的技能文件扩展能力。OpenClaw 兼容所有遵循 Agent Skill 规范的技能，无需修改即可直接使用。

### 方式 A：OpenClaw 原生 CLI 安装

前提：确保已安装 Node.js ≥ 22.x，并且已安装 OpenClaw。

```bash
# 安装单个 skill（需要先将此仓库发布到 ClawHub 注册表）
openclaw skills install chem-auto-lab-skill
openclaw skills install domain-literature-experiment-extraction-ontology-skill
openclaw skills install literature-to-lab-bridge

# 查看已安装的 skill
openclaw skills list --eligible

# 查看 skill 详情
openclaw skills info chem-auto-lab-skill

# 启用/禁用
openclaw skills enable chem-auto-lab-skill
openclaw skills disable chem-auto-lab-skill
```

### 方式 B：在 OpenClaw 聊天中粘贴 GitHub 链接（最简单）

无需安装任何 CLI 工具。直接在 OpenClaw 的聊天框中 **粘贴本仓库的 GitHub 链接**，助手会自动完成安装和配置：

```
用户: https://github.com/kingdol666/chem-lab-skills.git
→ OpenClaw 自动识别并安装其中的 3 个 Skill
```

### 方式 C：ClawHub CLI 安装

使用 `npx` 直接运行，无需全局安装：

```bash
# 安装单个 skill（需要先将此仓库发布到 ClawHub 注册表）
npx clawhub@latest install chem-auto-lab-skill
npx clawhub@latest install domain-literature-experiment-extraction-ontology-skill
npx clawhub@latest install literature-to-lab-bridge

# 搜索可用的 skill
npx clawhub search chem

# 批量更新
npx clawhub update --all
```

### 方式 D：手动复制到 Skills 目录

OpenClaw 支持 **三个优先级的目录**，按查找顺序排列：

| 优先级 | 路径 | 说明 |
|--------|------|------|
| 🥇 最高 | `<project>/skills/` | 项目级 skill（推荐，随项目共享） |
| 🥈 中等 | `~/.openclaw/workspace/skills/` | 用户全局 skill |
| 🥉 最低 | 内置 skill | OpenClaw 自带的 skill |

```bash
# 1. Clone 仓库
git clone https://github.com/kingdol666/chem-lab-skills.git /tmp/chem-skills

# 2. 选择安装位置
# 选项 A：项目级（推荐，skill 随项目共享）
mkdir -p your-project/skills/
cp -r /tmp/chem-skills/.claude/skills/* your-project/skills/

# 选项 B：全局
mkdir -p ~/.openclaw/workspace/skills/
cp -r /tmp/chem-skills/.claude/skills/* ~/.openclaw/workspace/skills/

# 3. 清理
rm -rf /tmp/chem-skills
```

**注意**：新版本 OpenClaw 可能需要配置目录白名单：

```bash
openclaw config set fs.allow-path "/root/.openclaw/workspace"
```

### 方式 E：多 Agent 安装器

如果同时使用 OpenClaw、Cursor、Codex 等多个 AI 助手，可用统一安装器：

```bash
# 使用 @goodpostidea-tech/skills 交互式安装
npx @goodpostidea-tech/skills add https://github.com/kingdol666/chem-lab-skills.git

# 或使用 skills.sh 指定目标
npx skills add kingdol666/chem-lab-skills -a openclaw
```

### 验证

在 OpenClaw 中尝试以下对话：

```
用户: 帮我分析化学实验数据
→ 自动匹配 chem-auto-lab-skill

用户: 搜索PVA光学膜的文献，提取数据，推荐课题
→ 自动匹配 literature-to-lab-bridge
```

---

## 方式四：Hermes 安装

[Hermes](https://github.com/angrysky56/hermes-cli) 是一个轻量级的 AI 编程助手框架，支持加载 Claude Code 兼容的 Skill 文件。

### 前提

```bash
# 安装 Hermes CLI
pip install hermes-cli
# 或
npm install -g @hermes/cli
```

### 步骤

```bash
# 1. 先 clone chem-skill 仓库
git clone https://github.com/kingdol666/chem-lab-skills.git /tmp/chem-skills

# 2. 复制到 Hermes 的 skill 目录
#    （具体路径取决于你的 Hermes 配置，通常为 ~/.hermes/skills/ 或项目根目录的 .hermes/skills/）
mkdir -p ~/.hermes/skills/
cp -r /tmp/chem-skills/.claude/skills/* ~/.hermes/skills/

# 3. 让 Hermes 重新加载技能
hermes reload

# 4. 清理
rm -rf /tmp/chem-skills
```

### 验证

```bash
hermes skills list
# 输出中应包含:
# - chem-auto-lab-skill
# - domain-literature-experiment-extraction-ontology-skill
# - literature-to-lab-bridge

# 或直接测试触发
hermes run "帮我解析FTIR谱图"
# → 自动匹配 chem-auto-lab-skill Module 2
```

### Hermes 配置示例

如果 Hermes 使用 YAML/JSON 配置文件来注册技能，在配置文件中添加：

```yaml
# ~/.hermes/config.yaml 或项目 .hermes/config.yaml
skills:
  - path: ~/.hermes/skills/chem-auto-lab-skill
    enabled: true
  - path: ~/.hermes/skills/domain-literature-experiment-extraction-ontology-skill
    enabled: true
  - path: ~/.hermes/skills/literature-to-lab-bridge
    enabled: true
```

---

## 安装验证

安装完成后，运行以下验证脚本（任一框架均可）：

```bash
# 在项目根目录执行
python -c "
import os, json

skills_dir = '.claude/skills'
required_skills = [
    'chem-auto-lab-skill',
    'domain-literature-experiment-extraction-ontology-skill',
    'literature-to-lab-bridge'
]

print('=' * 50)
print('Chem-Skill 安装验证')
print('=' * 50)

for skill in required_skills:
    skill_path = os.path.join(skills_dir, skill)
    skill_file = os.path.join(skill_path, 'SKILL.md')
    if os.path.exists(skill_file):
        # 验证 SKILL.md 包含必要的 frontmatter
        with open(skill_file) as f:
            content = f.read()
            has_name = 'name: ' in content
            has_desc = 'description: ' in content
            has_version = 'version: ' in content
        print(f'  ✅ {skill} — SKILL.md 完整 ({has_name}/{has_desc}/{has_version})')
    else:
        print(f'  ❌ {skill} — SKILL.md 缺失')

# 检查 bridge-runs 示例（如果 clone 了完整仓库）
runs_dir = 'bridge-runs'
if os.path.exists(runs_dir):
    runs = [d for d in os.listdir(runs_dir) if d.startswith('bridge_')]
    print(f'  📁 示例运行记录: {len(runs)} 个')
    for run in runs:
        plan = os.path.join(runs_dir, run, 'phase3_output', 'TOP_PLAN.md')
        manifest = os.path.join(runs_dir, run, 'bridge_manifest.json')
        if os.path.exists(plan) and os.path.exists(manifest):
            print(f'     ✅ {run} — TOP_PLAN.md + bridge_manifest.json 完整')

print('=' * 50)
print('验证完成！在 Claude Code 中输入以下内容测试触发：')
print('  \"帮我分析化学实验数据\" → chem-auto-lab-skill')
print('  \"搜索PVA光学膜文献\"     → domain-literature skill')
print('  \"搜索文献+分析课题\"     → literature-to-lab-bridge')
print('=' * 50)
"
```

预期输出：

```
==================================================
Chem-Skill 安装验证
==================================================
  ✅ chem-auto-lab-skill — SKILL.md 完整 (True/True/True)
  ✅ domain-literature-experiment-extraction-ontology-skill — SKILL.md 完整 (True/True/True)
  ✅ literature-to-lab-bridge — SKILL.md 完整 (True/True/True)
  📁 示例运行记录: 1 个
     ✅ bridge_20260602_pva_optical_film — TOP_PLAN.md + bridge_manifest.json 完整
==================================================
验证完成！在 Claude Code 中输入以下内容测试触发：
  "帮我分析化学实验数据" → chem-auto-lab-skill
  "搜索PVA光学膜文献"     → domain-literature skill
  "搜索文献+分析课题"     → literature-to-lab-bridge
==================================================
```

---

## 常见问题

### Q1：Skill 不自动触发怎么办？

```markdown
确保：
1. SKILL.md 文件位于正确的目录（.claude/skills/<skill-name>/SKILL.md）
2. SKILL.md 的 frontmatter 中 name 和 description 字段完整
3. 使用正确的触发关键词（中英文皆可）

手动调试：
- Claude Code: 直接在对话中输入关键词
- oh-my-claudecode: /oh-my-claudecode:<skill-name>
- OpenClaw: 在聊天中粘贴 skill 的 GitHub 链接，或用 `openclaw skills install <name>`
- Hermes: hermes run <skill-name>
```

### Q2：只想安装某一个 Skill 而不是全部？

```
三种方式：
1. 只复制需要的 skill 目录
   cp -r source/.claude/skills/chem-auto-lab-skill target/.claude/skills/

2. 用 sparse checkout 部分 clone
   git clone --filter=blob:none --sparse https://github.com/kingdol666/chem-lab-skills.git
   cd chem-lab-skills
   git sparse-checkout set .claude/skills/chem-auto-lab-skill

3. 从 GitHub 直接下载单个目录
   # 使用 GitHub 的 subdirectory 下载（需要第三方工具如 svn）
   svn export https://github.com/kingdol666/chem-lab-skills/trunk/.claude/skills/chem-auto-lab-skill
```

### Q3：如何测试 Skill 是否正常工作？

```
在框架中输入测试语句观察触发情况：

1. "帮我分析化学实验数据"
   → 应触发 chem-auto-lab-skill
   → 进一步问 "数据在哪里？"

2. "提取PVA光学膜文献的实验数据"
   → 应触发 domain-literature skill
   → 进一步问 "搜索关键词是什么？"

3. "搜索PVA光学膜文献，分析趋势，推荐课题"
   → 应触发 literature-to-lab-bridge
   → 启动完整的 6-Phase 流水线
```

### Q4：Skill 触发后说"找不到这个技能"？

```
可能是框架的 Skill 发现路径问题：
- Claude Code: 检查 `.claude/skills/` 目录结构是否正确
- oh-my-claudecode: 运行 `/omc-setup` 重新初始化
- OpenClaw: 检查 skill 是否在 `<project>/skills/` 或 `~/.openclaw/workspace/skills/` 下，或运行 `openclaw skills list --eligible`
- Hermes: 运行 `hermes reload` 重新加载
```

### Q5：如何更新 Skill 到最新版本？

根据你的框架选择对应路径：

**Claude Code / oh-my-claudecode:**
```bash
rm -rf .claude/skills/chem-auto-lab-skill .claude/skills/domain-literature-experiment-extraction-ontology-skill .claude/skills/literature-to-lab-bridge
git clone https://github.com/kingdol666/chem-lab-skills.git /tmp/chem-skills
cp -r /tmp/chem-skills/.claude/skills/* .claude/skills/
rm -rf /tmp/chem-skills
```

**OpenClaw:**
```bash
rm -rf ~/.openclaw/workspace/skills/chem-auto-lab-skill ~/.openclaw/workspace/skills/domain-literature-experiment-extraction-ontology-skill ~/.openclaw/workspace/skills/literature-to-lab-bridge
git clone https://github.com/kingdol666/chem-lab-skills.git /tmp/chem-skills
cp -r /tmp/chem-skills/.claude/skills/* ~/.openclaw/workspace/skills/
rm -rf /tmp/chem-skills
```

**用 git submodule（如果你已将此仓库作为依赖加入项目）：**
```bash
git submodule update --remote .claude/skills
```

### Q6：Python 脚本需要运行吗？

```
不需要。主流程由 LLM 通过 Claude Code 内置工具全自动执行。
Python 脚本仅用于辅助场景（如批量文件转换），不是核心执行路径。
```

---

## 参考

| 资源 | 链接 |
|------|------|
| Chem-Skill 仓库 | https://github.com/kingdol666/chem-lab-skills |
| Claude Code 文档 | https://docs.anthropic.com/en/docs/claude-code |
| oh-my-claudecode | https://github.com/superanos/oh-my-claudecode |
| OpenClaw 官网 | https://openclaw.ai / https://github.com/TheAppleTucker/open-claw |
| OpenClaw Skills 生态 (1715+ skills) | https://github.com/sjkncs/awesome-openclaw-skills |
| ClawHub 注册中心 + CLI 工作流 | https://www.w3cschool.cn/openclawdocs/openclaw-tools-clawhub.html |
| Hermes CLI | https://github.com/angrysky56/hermes-cli |
