# 设计：`start-distill-domain-knowledge` — per-domain 开张向导

- **日期**：2026-06-30
- **状态**：设计已获批（经 grill-me 压测），待转 writing-plans
- **对应 ROADMAP**：#1（原 `init-knowledge-base`；本 spec 将其更名收敛为 `start-distill-domain-knowledge`）
- **关联 skill**：`distill-domain-knowledge`（引擎，本 skill 是其前门）、`find-domain-knowledge`（消费侧 L2 发现）、`grill-me`（访谈子工具，外部依赖）

> **公开仓 hygiene 声明（不变量 I8）**：distill-me 是 public 仓，本 spec 一切示例一律用**抽象占位符 `domainA` / `domainB` / `<core-service>` / `<key>`**，零雇主内部内容。skill 跑起来生成到用户**私有** KB 仓时才填真值。

---

## 1. 问题与目的

`distill-domain-knowledge` 假定一套**相对固定的目录/文件结构已存在**（其输出形态直接引用 `knowledge/`、`glossary.md`、`pending/gaps.md`），并以 per-slice（单子域切片）粒度反复运行。但"一个新领域从零开张"——立目录、定边界、锁核心服务（代码锚点）、产一张"该蒸哪些子域"的菜单——目前只散落在 `AGENTS.md §B`（B3 搭骨架）的**手工 prose SOP** 里，靠人记得去读、手动照搬。

`start-distill-domain-knowledge` 把这段"建库前置"固化成一个 **grill 驱动的 per-domain 开张向导 + 分发器**：

```
/start-distill-domain-knowledge "我要蒸馏 domainB 领域"
```

四件事：① 搭/扩固定结构 → ② grill 领域身份（边界 + 核心服务）→ ③ 可选方向性侦察 → ④ 产**初始 backlog** → **交棒** `distill-domain-knowledge` 做 per-slice。

## 2. 角色、边界与 altitude 模型

三级粒度，本 skill 工作在**领域级**：

```
仓库 / KB
  ├─ 领域 domainA   ← 之前 start- 过一次（已有 knowledge/domainA/ + recall-domainA-knowledge + glossary 的 domainA 词条）
  └─ 领域 domainB   ← 现在 start-distill-domain-knowledge 要新增的      ← 本 skill 粒度（per-domain，跑一次 genesis）
        slice s1 / s2 / s3 …                                          ← distill-domain-knowledge 粒度（per-slice，反复跑）
```

- **per-domain，跑一次 genesis**：一个领域开张一次；一个仓**累积多个领域**。
- **交棒，不越界**：产出初始 backlog 后交给 `distill-domain-knowledge`；**不**做 per-slice 蒸馏，**不**装协议 skill。

### 2.1 "Scaffold" 术语消歧（glossary-first）

本 skill 的"scaffold"与 `distill-domain-knowledge` 的 **①Scaffold 步不是一回事**：

| | distill ①Scaffold（`distill-domain-knowledge/SKILL.md:57`） | 本 skill 的 scaffold |
|---|---|---|
| 层级 | **内容**骨架：某切片的 typed entity catalog | **容器** + 领域身份 + 粗粒度域级侦察 |
| 粒度 | per-slice | per-domain |
| 前提 | **假定**目录结构已存在 | **创建/扩展**目录结构 |

→ 本 skill 内部步骤一律用 **STEP0–4**，避开裸 "Scaffold"。

### 2.2 与 `AGENTS.md §B` 的关系

`§B` 是跨宿主手工 bootstrap prose，保留为可移植降级路径。本 skill 是 **Claude-Code-native 自动化路径**，覆盖**加强版 B3**（搭骨架 + 领域 grill + 侦察）。**不**接管 B1（装协议 skill）、B2（装外部 plugin）；B0 能力自检 / B4 就绪自检以**报告**呈现，不强制。

## 3. 造法（已定）

**独立 skill，复用 distill 的 references（DRY，不拷贝）。**

- 新建 `.claude/skills/start-distill-domain-knowledge/SKILL.md`（spine）。
- grill 纪律、glossary-first、Memory loop 一律**引用** `distill-domain-knowledge/references/{grilling,glossary-first,memory-loop}.md`，不复制（references 是单一真相源，避免漂移）。
- 自带 `templates/`：`kb-CLAUDE.md`（§6.6）、glossary 表头、gaps/backlog 表头、L1 stub 模板——**全用占位符**（I8）。

**否决的替代**：折进 distill 当 mode-0（污染 per-slice altitude）；极薄 delegator（独立可发现性弱）。

## 4. 运行时决策树（grill 驱动，每问带推荐答案）

```
/start-distill-domain-knowledge "我要蒸馏 domainB 领域"
│
├ STEP0 检测环境
│   有 knowledge/ + glossary.md + recall-*-knowledge?
│   ├ 有既有 KB 结构 → ADDITIVE
│   └ 无 → GREENFIELD，但**先 confirm 落点**（"这里没有既有 KB，在 <cwd> 新建 domainB 知识库吗?"）  ← I4
│   ※ "别在 distill-me 本仓建" 的守卫**不进本 skill**（保持通用），由 distill-me 自己的 CLAUDE.md 一句话挡（见 §10 action item）
│
├ STEP1 [仅 ADDITIVE] recall@start 反哺 + 关系 grill                                                 ← I6
│   先 recall 记忆库（domainB 是否已被提及挂在某域下?）→ 带提示问归属：
│   ┌ 同 KB 并列 domain → knowledge/domainB/ 与 domainA/ 并列；新 recall-domainB-knowledge；共享命名空间
│   ├ 不同 KB           → 用户指定**另一路径** greenfield 起新结构（§6 末）；独立路由；独立命名空间
│   └ 大 domain 的 sub-domain → 嵌套 knowledge/<大域>/domainB/；**扩展**既有路由；沿用大域命名空间
│
├ STEP2 grill 领域身份（both）
│   边界（domainB 含/不含什么）? 核心服务 = 哪些代码仓/路径?
│   → **让用户指定每个核心服务的本地路径** → 存在性核验：                                            ← I3 输入
│       可读 → 开 STEP3；不可读 → warn + 记 gaps + 对该服务**跳过** STEP3
│   → glossary 收录 umbrella 词；additive 时**先**跨域碰撞检查                                       ← I2
│
├ STEP3 [可选，opt-in 后问深度] 方向性探索
│   扫核心服务 → 初始 backlog；默认**提议跑**、深度**必问**（轻/中，默认中=排序菜单）、可 skip
│
└ STEP4 落产物（非破坏式）                                                                          ← I1
    greenfield：建容器 + 第一个域 + L1 stub + 命名空间 + 薄指针 CLAUDE.md
    additive：  只新增/扩展，绝不覆盖既有域**知识文件**
    → index@end（记忆库落 namespace + umbrella 词 + backlog）                                       ← I6
    → 交棒："挑 backlog 一个子域，对我说 /distill…"
```

> **所有分支值（greenfield/additive、归属关系、侦察深度）均运行时由 grill 确定，绝不在设计/代码硬编码。** 设计交付的是这棵树的形状 + 不变量 + 产物。

## 5. 设计时不变量（从 RED 场景反推）

| # | 不变量 | 来源 |
|---|---|---|
| I1 | additive 非破坏式：严格保护既有域**知识文件** `knowledge/<域>/*.md` + 其 recall stub，**永不覆盖/删改** | coding-style immutability |
| I2 | additive 跨域 glossary 碰撞检查：碰撞 → **grill 专家** → 消歧式 **additive** 编辑（加 scope 限定/拆 overloaded headword，**绝不删除或反转**既有含义）；当场定不了 → 入 `pending/gaps.md` | 铁律#2 / glossary-first |
| I3 | 生成的 L1 stub **禁** repo-relative `knowledge/*.md` 路径、**禁** host:port；只依赖全局 skill + 可移植 memory MCP | `find-domain-knowledge/SKILL.md:49` |
| I4 | greenfield 须 **confirm 落点**；"distill-me 本仓守卫"归本仓 CLAUDE.md（非本 skill，保持通用） | grill Q1 |
| I5 | greenfield **极简**；关系-grill 分支**只 additive 触发** | KISS |
| I6 | Memory loop：recall@start **反哺**检测/关系判断，index@end 落 namespace + umbrella + backlog | distill `references/memory-loop.md` |
| I7 | **不**装协议 skill（B1 出界）；产 initial backlog 后**交棒** distill，不做 per-slice 蒸馏 | 用户界定 |
| I8 | **public-repo hygiene**：本 skill 自身一切产物（SKILL.md / template / spec / fixture）只许抽象占位符 / 中性示例，**零**雇主内部内容；生成的**输出**落用户私有 KB 才填真值；提交公开仓用 **clean git 身份** | distill-me public |

> **I1↔I2 不矛盾**：I1 锁死"既有域**知识文件**不可破坏"；glossary 是 cross-domain 工件，I2 授权对它做经专家裁决的**消歧式 additive 编辑**。

## 6. 产物

### 6.1 Greenfield 骨架
```
<kb-repo>/
├── CLAUDE.md                  # 薄指针（§6.6）+ 三铁律一行各一句安全网
├── AGENTS.md                  # 双入口（symlink/copy）
├── knowledge/
│   ├── glossary.md            # 空表 + 表头
│   └── <domain>/              # 如 domainB/（空，distill 填）
├── pending/
│   ├── gaps.md                # 待验证 punch-list： # | claim/疑点 | why unresolved | what's needed | owner
│   └── backlog.md             # 候选子域活账本（§6.4）
└── .claude/skills/recall-<domain>-knowledge/SKILL.md   # L1 路由 stub（可移植，§6.3）
```

### 6.2 Additive：只增不改
按 STEP1 归属关系，**仅**新增/扩展（并列 → `knowledge/<新域>/` + 新 `recall-<新域>-knowledge`；sub-domain → 嵌套 + **扩展**既有路由）。**绝不触碰**既有域的知识文件（I1）；glossary 仅允许 I2 的消歧式 additive 编辑。

### 6.3 L1 stub 内容（可移植，I3）
init 时尚无已蒸馏 domain skill，故 stub 是**前向声明**薄路由：写明本 KB 覆盖 `<domain>` 及入口语义；硬编码**自己** KB 坐标（记忆命名空间键、随 distill 产出补全的 domain-skill 名列表）；**禁** repo-relative `knowledge/*.md` 路径、**禁** host:port；依赖项只有全局 skill（Skill 工具）+ 可移植 memory MCP。

### 6.4 `backlog.md` = 带状态活账本（option b，前向兼容 #2）
schema：`# | 候选子域 | why worth（最常问/最易错）| 核心服务（代码锚点）| status（pending/in-progress/done）| 优先级`
- start- 只产**初始 backlog（全 pending）**；**下游** distill/#2 在蒸馏过程中**更新状态、追加新候选**——**出 start- scope**，这里只保证 schema 支持更新、**留缝**。
- **机器可读**（为"继续蒸馏"/#2 checkpoint 前向兼容），但 distill **不为控制流读它**（人挑 slice，保持 distill 与 init 产物格式解耦）。

### 6.5 gaps vs backlog 分立（两条消费回路）
`backlog.md`="待蒸的子域"（人挑选消费）；`gaps.md`="待验证的断言"（distill ③Verify 产、代码核验消费）。分立让各自"完成 = 清空"语义干净。

### 6.6 KB `CLAUDE.md` = 薄指针（单一职责）
唯一职责：告诉 agent "**扩展本 KB 要用 distill-me 纪律**"。内容形如：
> "本仓是 `<domain>` 的蒸馏知识库。要扩展/更新它，用 distill-me workflow（加载 `distill-domain-knowledge`，遵其纪律）。记忆命名空间：`<key>`。"
+ 三铁律**一行各一句**安全网（防 skill 未加载时裸写）。**不**整段嵌入规则（指针化 → 消除与本仓 CLAUDE.md 的漂移）。来自 `templates/kb-CLAUDE.md` 占位填充。

### 6.7 记忆命名空间
含 `/` 的规范 project 键（如 `knowledge/<域>`，随 STEP1 关系定）；写入走 `remember` 必带该键。

> **"不同 KB" 机制（实现细节）**：= 在用户指定的**另一路径** greenfield 起一套新结构，**不**嵌套进当前仓。

## 7. RED→GREEN 验证（本 skill 自己也得过）

可测失败模式即 §5 不变量：

- **在独立 fixture KB 仓跑**：start- **不装进** fixture 的 `.claude/skills/` → 普通跑一遍**天然干净 RED 基线**（隔离即净，**不需** `claude -p` ceremony）。GREEN 时把 skill 指过去。
- **fixture**：小 KB 仓 = 既有 `knowledge/domainA/` + glossary（含**故意 overloaded 词 `T`**）+ `recall-domainA-knowledge` + **假核心服务目录**。预置 grill 答案 = "加 domainB、并列 domain、核心服务在 `<fixture 路径>`、**探索 skip**"。
- **判产物/纪律，不判活对话**：RED（无 skill）看是否 clobber domainA（I1）/漏 `T` 碰撞（I2）/ stub 硬编码路径（I3）/当 greenfield 跳关系（I5）；GREEN（带 skill）四项全清。**Opus 档跑**。
- **探索 skip 一举两得**：让 grill 可预置 + **绕开 STEP3 fan-out 的嵌套测试上限**。
- **诚实声明**：grill 的**对话流本身**（顺序/分支）**不被 headless RED→GREEN 覆盖**——靠 §4 显式决策树 + 人工走查保证，不假装被自动测了。

## 8. 命名 / glossary 准入记录

- **headword**：`start-distill-domain-knowledge`（scope-type：user-invocable skill / 蒸馏 workflow 的 per-domain 入口与分发器）
- **碰撞检查**：vs `distill-domain-knowledge`（引擎，本 skill 是其前门，刻意配对）· vs `find-domain-knowledge`（消费侧姊妹）· vs 仓名 `distill-me`——**已避开** `start-distill-me` 会撞的两处（仓名 + `AGENTS.md:64`『开始蒸馏我自己』的 §B 环境-bootstrap 语义）。
- **代价**：家族里最长、唯一双动词；但该家族不靠命名约定路由（只 `recall-*-knowledge` 靠），一致性仅可读性 nicety。

## 9. 非目标（Out of scope）

- 装协议 skill（B1）/ 装外部 plugin（B2）—— 宿主一次性手动。
- per-slice 蒸馏 —— `distill-domain-knowledge` 职责。
- "继续蒸馏"/checkpoint resume + backlog 状态推进 —— ROADMAP **#2** 职责；start- 只产 initial backlog + 留前向兼容缝。
- plugin 形态（ROADMAP M1/M2）—— 独立演进；与 M2「bootstrap command」同诉求，定形态时再合并。

## 10. 实现期 action items / 开放

- **给 distill-me 本仓 `CLAUDE.md` 加一句**"本仓是 workflow 本体，不在此 distill / 建库"（STEP0 的 distill-me 守卫，I4）。
- backlog 排序启发式（"最常问/最易错"如何量化）—— 实现期细化。
- 与 #2 checkpoint 的"backlog 状态推进"seam 对接时机。
- 三铁律"规范源"是否抽公共：模板薄指针化后漂移已消，**暂不需**。
