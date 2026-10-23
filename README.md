# Trellis

科研全流程 Skill。**四张逻辑图 + 五道闸门 + 三段回流 + 一个证据库。**

面向需要把研究做扎实、而不是只把论文写出来的人。适用于 WorkBuddy / Claude Code / Codex 等支持 `SKILL.md` 的 agent 环境。

---

## 它解决什么问题

科研最常出错的地方不在写，而在**逻辑链断了自己看不见**：

- 读了 60 篇文献，说不清 gap 到底在哪
- 选题看着不错，动手才发现数据拿不到
- 实验做完了，结论比数据支持的强了一档
- 引用里有一条是编的，投出去才被审稿人抓住

Trellis 的应对是把整条流程做成**可回退的状态机**：每一步有明确的判据和产物，每个闸门有四个出口（通过 / 小修 / 回退 / 终止），走错了能精确退回最小可修复点，而不是推倒重来。

---

## 核心设计

### 四张图

同一套逻辑，四种粒度，逐层同构：

| 图 | 何时建 | 内容 |
|---|---|---|
| **L-Graph** 文献图 | 读完 8–10 篇 | 问题、方法、数据、结论、局限、指标 + 7 类关系边 |
| **I-Graph** 选题图 | 定题时 | 变量、机制、假设 + 因果方向边 |
| **D-Graph** 设计图 | 动手前 | 变量、对照、分析步骤 + 识别策略边 |
| **A-Graph** 论证图 | 写作前 | 图表、E-ID、主张 + 支持/限定/反驳边 |

**同构铁律**：L 的 gap 边 → I 的假设边 → D 的识别边 → A 的论证边，逐层对应。断在哪层，就是哪层的漏洞。

四种 gap 不靠"我觉得没人做过"，靠图上的结构特征：矛盾边、孤岛、断链、边界断点。

### 五道闸门

| 闸门 | 时机 | 核心问题 |
|---|---|---|
| G1 | 选题 → 设计 | 值得做吗？做得动吗？ |
| G2 | 动手前 | 设计能回答这个问题吗？ |
| G3 | 下结论前 | 证据支持这个说法吗？ |
| G4 | 投出前 | 每个数字每条引用都站得住吗？ |
| G5 | 返修提交前 | 改动破坏原有逻辑了吗？ |

每个闸门只能给四个结论之一：`PASS` / `REVISE` / `ROLLBACK` / `KILL`。

### 三段回流

回流目标是**最小可修复点**，不是起点。附 12 行回流矩阵、每闸门回流预算（防死循环）、以及最小重算规则（改上游下游全废，改下游上游不动）。

### 证据溯源

阶段二的每条结论生成 `E-001` 式条目，阶段三每句事实陈述必须指回某个 E-ID。指不回去的标 `[UNVERIFIED]`。

---

## 安装

Skill 目录名必须为 `trellis`。

### 方式一：克隆后复制（推荐）

```bash
git clone https://github.com/Meruem27/trellis-skill.git
mkdir -p ~/.workbuddy-ai/skills
cp -r trellis-skill/trellis ~/.workbuddy-ai/skills/
```

Claude Code / Codex 用户把目标目录换成 `~/.claude/skills/` 或 `~/.codex/skills/` 即可。

### 方式二：只装到单个项目

```bash
git clone https://github.com/Meruem27/trellis-skill.git
mkdir -p <你的项目>/.workbuddy-ai/skills
cp -r trellis-skill/trellis <你的项目>/.workbuddy-ai/skills/
```

项目级安装只在当前工作区生效，适合团队共享同一套规范。

### 方式三：手动

下载 `trellis/SKILL.md`，放进 `<skills目录>/trellis/SKILL.md`。单文件，无依赖。

---

## 怎么用

装好后直接用自然语言触发，不需要记命令：

| 你说 | 它做什么 |
|---|---|
| "我想研究 XX，但不知道做什么" | 从检索策略开始，建文献图，找 gap |
| "我有方向了，帮我看看值不值得做" | 六维打分 + 强制否决项 + 三个候选 |
| "我有 idea，怎么设计实验" | PICO 拆解、样本量、预注册、设计图 |
| "数据跑完了，帮我分析" | 数据体检 → 统计决策树 → 图表规范 |
| "帮我写论文" | 先索要证据库，建论证图，再分节写 |
| "审稿意见回来了" | 意见分类 + 动作映射码 + 逐条回复 |

六种工作模式：`quick`（单点问答）、`standard`（默认，单阶段）、`full`（全流程）、`resume`（断点续跑）、`reflow`（按回流矩阵定位）、`audit`（只审不写）。

---

## 它会产出什么

跑起来之后，你的项目里会生成这套结构：

```
<项目>/
├ 00-admin/      project.yaml · run-ledger.md · reflow-log.md · topic-brief.md
├ 10-literature/ search-log.md · cards/ · matrix.md
├ 15-graph/      L-graph.md · I-graph.md · D-graph.md · A-graph.md
├ 20-data/       raw/ · processed/ · data-dictionary.md · cleaning-log.md
├ 30-code/       src/ · experiments/ · run-ledger.csv
├ 40-results/    figures/ · figure-plan.md · stats-report.md · evidence-pack.md
├ 50-writing/    manuscript.md · citation-audit.md
└ 60-submission/ cover-letter.md · submission-kit.md · responses/
```

`project.yaml` 记录当前阶段、各闸门已用轮数、累计回退次数——中断了能接着跑。

---

## 包含哪些具体判据

不是空泛的方法论，是可执行的清单：

- **文献**：数据库路由表、三层检索（核心/邻域/跨界）、饱和度判据、A/B/C 质量分级、安全论文七问
- **会议期刊**：16 个计算机方向的 CCF-A 会议与顶刊，安全四大（S&P / USENIX Security / CCS / NDSS）单列
- **设计**：设计类型-因果强度对照、混杂控制优先级、样本量与 MDE、预注册清单
- **统计**：检验选择决策树、前提检查、必报三件套、效应量对照表、七类谬误自检
- **图表**：Nature 系图规范（rcParams、尺寸、字号）、一图一问、图表清单
- **写作**：Abstract 六段弧、Introduction 五步漏斗、Results 证据阶梯、润色硬规则、中式英语病灶表
- **引用**：五类阻断错误、支撑强度分级
- **投稿**：投稿包清单、Cover Letter 四段、Data Availability 三种写法、审稿回复四段结构
- **反谄媚**：选题必须给否决理由，审稿模拟必须给至少三条实质批评

---

## 设计来源

架构借鉴了几个公开项目的工程经验，内容为重写：

- [academic-research-skills](https://github.com/Imbad0202/academic-research-skills) — 阶段闸门、引用锚点、反谄媚协议
- [SciAgents](https://github.com/lamm-mit/SciAgentsDiscovery) — 概念-关系本体图、路径采样即假设
- [ScholarGraph](https://github.com/Josephyb97/ScholarGraph) — 概念分类与加权边、gap 检测
- [nature-skills](https://github.com/snipp-zha/nature-skills) — 路由式 SKILL.md、按需加载、Nature 图规范

---

## 边界

- 不生成实验数据、不编造引用、不虚构统计结果。
- 不替你做"这个发现很重要"的判断，只给证据强度和替代解释。
- 领域默认值（安全 / AI / 系统 / 生物医学 / 社科 / 人文）是通行共识，投前请核对目标期刊当年要求，以及 CCF 目录最新版。

---

## License

MIT
