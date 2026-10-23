# Trellis

科研全流程 Skill。**四张逻辑图 + 五道闸门 + 三段回流 + 一个证据库。**

面向需要把研究做扎实、而不是只把论文写出来的人。适配 WorkBuddy / Claude Code / Codex / DeepSeek Harness / Trae 及任何符合 [Agent Skills 规范](https://agentskills.io/specification) 的宿主。

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

核心只有一个 `name` + `description` + Markdown 正文，无脚本、无外部依赖。任何遵循 Agent Skills 规范的宿主都能直接读。

目录结构统一为 `<skills目录>/trellis/SKILL.md`，**目录名必须是 `trellis`**。

### 各宿主放哪里

| 宿主 | 项目级 | 用户级 |
|---|---|---|
| **WorkBuddy** | `<项目>/.workbuddy-ai/skills/` | `~/.workbuddy-ai/skills/` |
| **Claude Code** | `<项目>/.claude/skills/` | `~/.claude/skills/` |
| **Codex** | `<仓库>/.agents/skills/`（从当前目录向上扫到仓库根） | `~/.agents/skills/` |
| **DeepSeek Harness** | `.dsh/skills/`、`.agents/skills/`（前者优先级更高） | `~/.dsh/skills/`、`~/.agents/skills/` |
| **Trae** | `<项目>/.trae/skills/` | 设置里的「全局技能」目录 |
| 其他（符合 Agent Skills 规范） | 查该宿主文档的 skills 目录 | 同上 |

### 方式一：一份源码，多宿主链接（推荐）

Codex 和 DeepSeek Harness 原生读 `.agents/skills`。把它当源目录，其他宿主用符号链接接进去——改一处，全部生效。

```bash
git clone https://github.com/Meruem27/trellis-skill.git /tmp/trellis-skill
mkdir -p .agents/skills
cp -r /tmp/trellis-skill/trellis .agents/skills/

# Claude Code
mkdir -p .claude/skills       && ln -s ../../.agents/skills/trellis .claude/skills/trellis
# WorkBuddy
mkdir -p .workbuddy-ai/skills && ln -s ../../.agents/skills/trellis .workbuddy-ai/skills/trellis
# Trae
mkdir -p .trae/skills         && ln -s ../../.agents/skills/trellis .trae/skills/trellis
```

Windows 用 `mklink /D`，或直接复制目录。

两个坑：

- Git 提交时要保留符号链接。`core.symlinks=false` 会把链接写成普通文本，宿主扫不到。
- 用复制方案的话，在 CI 里校验两份文件的哈希，避免同一个 Skill 被改出两个版本。

### 方式二：装到用户级（所有项目可用）

```bash
git clone https://github.com/Meruem27/trellis-skill.git /tmp/trellis-skill
mkdir -p ~/.workbuddy-ai/skills ~/.claude/skills ~/.agents/skills
cp -r /tmp/trellis-skill/trellis ~/.workbuddy-ai/skills/
cp -r /tmp/trellis-skill/trellis ~/.claude/skills/
cp -r /tmp/trellis-skill/trellis ~/.agents/skills/
```

Codex 与 DeepSeek Harness 读 `~/.agents/skills`，一次覆盖三个宿主。Trae 在设置里用「全局技能」导入。

### 方式三：只装一个宿主

```bash
# WorkBuddy
mkdir -p ~/.workbuddy-ai/skills && cp -r trellis-skill/trellis ~/.workbuddy-ai/skills/

# Claude Code
mkdir -p ~/.claude/skills && cp -r trellis-skill/trellis ~/.claude/skills/

# Codex / DeepSeek Harness
mkdir -p ~/.agents/skills && cp -r trellis-skill/trellis ~/.agents/skills/

# Trae（项目内）
mkdir -p .trae/skills && cp -r trellis-skill/trellis .trae/skills/
```

装完在宿主里发一句「帮我看看这个研究方向值不值得做」，能触发就说明加载成功。

### 方式四：不支持 SKILL.md 的宿主

Cursor / Windsurf / Cline / Kiro 这类用规则文件的宿主，把 `SKILL.md` 正文整段复制成规则文件（frontmatter 转成对应格式）：

| 宿主 | 常见规则路径 |
|---|---|
| Cursor | `.cursor/rules/trellis.mdc` |
| Windsurf | `.windsurf/rules/` |
| Cline | `.clinerules/` |
| Kiro | `.kiro/steering/` |

路径以各宿主当前文档为准。注意规则文件是**常驻上下文**，会一直占 token；`trellis` 正文 800 行，全量塞进规则里代价不小。这类宿主建议只用 `quick` 模式（单点问答），完整的三阶段流程还是放在支持按需加载的宿主里跑。

### 兼容性说明

- frontmatter 里的 `agent_created: true` 是 WorkBuddy 专用字段，其他宿主会忽略，不影响加载。
- 所有宿主都是**渐进式披露**：先读 `description` 判断要不要调用，命中后才读正文。所以 description 写得准比正文写得长更重要。
- Skill 只是知识与流程，不附带任何执行权限。脚本、沙箱、审批仍由各宿主自己管。

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

触发方式随宿主而变：Claude Code 用 `/trellis`，Codex 用 `$trellis` 或选择器，WorkBuddy / Trae 直接说人话即可，不需要记命令。

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
