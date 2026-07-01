# start-distill-domain-knowledge Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 造一个 grill 驱动的 per-domain 开张向导 skill，把"建库前置"从 `AGENTS.md §B` 的手工 prose 固化成可触发 skill + 模板 + RED→GREEN fixture。

**Architecture:** 独立 skill（spine `SKILL.md` + `templates/`），复用 `distill-domain-knowledge` 的 references（DRY，不拷贝）。产物是 **prose + 模板**、非传统代码——故"测试"是 **RED→GREEN**：让 agent 在 fixture KB 上跑，判它产的产物是否守 I1/I2/I3/I5，无 skill=RED 应违规、带 skill=GREEN 应干净。

**Tech Stack:** Markdown `SKILL.md`（Claude Code skill 约定：YAML frontmatter `name`+`description`）；无运行时代码/依赖；验证靠 Claude Code agent（**Opus 档**）跑 fixture。

**内容真相源**：已批 spec `docs/superpowers/specs/2026-06-30-start-distill-domain-knowledge-design.md`（提交 `baaa366`）。本计划为其**执行序列**；大段 prose 以 spec §N 为准，短产物在此内联。

## Global Constraints
> 每个任务的要求都隐含包含本节。逐条 verbatim 自 spec。

- **I8 public-repo hygiene**：所有产物（SKILL.md/templates/fixture/本计划）**只许抽象占位符**（`domainA` / `domainB` / `{{...}}` / `<placeholder>`），**零**雇主内部内容。grep 到 `moego` / 真实 svc 名 = 失败。
- **复用不复制**：grill / glossary-first / Memory loop 一律**引用** `distill-domain-knowledge/references/{grilling,glossary-first,memory-loop}.md`，不拷贝其内容。
- **名字锁定** `start-distill-domain-knowledge`（glossary 已准入，spec §8）。内部步骤命名用 **STEP0–4**，避开裸 "Scaffold"（与 distill ①Scaffold 消歧）。
- **RED→GREEN**：Opus 档跑；预置 grill 答案含"**探索 skip**"（绕 STEP3 fan-out 嵌套测试上限）。
- **skill 提交在本 agent 环境不会 SSH 签名**（key 不可达）；如需 Verified，会话后在本地终端 `git rebase --gpg-sign` 补签。

## File Structure（decomposition 锁定于此）

| 路径 | 责任 |
|---|---|
| `.claude/skills/start-distill-domain-knowledge/SKILL.md` | spine：role/boundary/altitude + STEP0–4 决策树 + I1–I8 + 引用 distill references + 交棒 |
| `.claude/skills/start-distill-domain-knowledge/templates/kb-CLAUDE.md` | greenfield KB 的薄指针 CLAUDE.md（占位符） |
| `.claude/skills/start-distill-domain-knowledge/templates/glossary.md` | 空 glossary 表头 |
| `.claude/skills/start-distill-domain-knowledge/templates/gaps.md` | 待验证 punch-list 表头 |
| `.claude/skills/start-distill-domain-knowledge/templates/backlog.md` | 候选子域活账本表头（带 status） |
| `.claude/skills/start-distill-domain-knowledge/templates/recall-DOMAIN-knowledge.SKILL.md` | L1 路由 stub 模板（可移植，占位符） |
| `CLAUDE.md`（distill-me 根） | 加"本仓不在此建库"守卫行（STEP0 distill-me guard，I4） |
| `test/fixtures/kb-domainA/` | RED→GREEN fixture KB（既有 domainA + overloaded 词 T + recall-domainA-knowledge + 假 svc 目录） |
| `ROADMAP.md` | #1 更新为"已出 spec+plan、更名 start-distill-domain-knowledge、in progress" |

> 单子系统、单 plan，scope 自洽（无需拆分）。

---

### Task 1: RED→GREEN fixture（测试先行）

**Files:**
- Create: `test/fixtures/kb-domainA/knowledge/domainA/rule-x.md`
- Create: `test/fixtures/kb-domainA/knowledge/glossary.md`
- Create: `test/fixtures/kb-domainA/pending/gaps.md`
- Create: `test/fixtures/kb-domainA/.claude/skills/recall-domainA-knowledge/SKILL.md`
- Create: `test/fixtures/kb-domainA/CLAUDE.md`
- Create: `test/fixtures/svc-fake/domainb_handler.txt`（假核心服务代码靶，给 STEP2 路径核验用）
- Create: `test/fixtures/README.md`（RED→GREEN procedure + 4 项 invariant 判据）

**Interfaces:**
- Produces: 一个"已含 domainA 的 KB"现场，glossary 里**故意埋一个 overloaded 词 `T`**（后续 domainB 会想复用 `T` 表另一义 → 触发 I2 碰撞）。Task 7 的 RED/GREEN 都跑在此 fixture 上。

- [ ] **Step 1: 建 fixture 目录与既有 domainA 内容**

`test/fixtures/kb-domainA/knowledge/domainA/rule-x.md`：
```markdown
# domainA · rule-x
✅ verified @ svc-fake/domaina_core.txt:1 — domainA 的 `T` 指「状态机的终态标记」。
```

`test/fixtures/kb-domainA/knowledge/glossary.md`（含 overloaded 词 `T`）：
```markdown
# domainA Glossary
| 术语 | 规范定义 | scope-type | 锚点 |
|---|---|---|---|
| T | domainA：状态机终态标记 | domainA-term | svc-fake/domaina_core.txt:1 |
```

`test/fixtures/kb-domainA/pending/gaps.md`：
```markdown
# pending gaps
| # | claim/疑点 | why unresolved | what's needed | owner |
|---|---|---|---|---|
```

- [ ] **Step 2: 建既有 L1 stub 与 KB CLAUDE.md**

`.../recall-domainA-knowledge/SKILL.md`：
```markdown
---
name: recall-domainA-knowledge
description: Load distilled domainA knowledge before reasoning about domainA. Invoke BEFORE concluding.
---
# Recall domainA Knowledge
本 KB 覆盖 domainA。记忆命名空间：`knowledge/kb-domainA`。
```

`test/fixtures/kb-domainA/CLAUDE.md`：
```markdown
# domainA 知识库（fixture）
扩展本 KB 用 distill-me workflow。记忆命名空间 `knowledge/kb-domainA`。
```

- [ ] **Step 3: 建假核心服务靶 + procedure 文档**

`test/fixtures/svc-fake/domainb_handler.txt`（几行伪代码，让 STEP2 路径核验有真实可读靶 + STEP3 有东西可扫）：
```text
// fake domainB service — 用于 fixture，非真实代码
func HandleT(x) { /* domainB 想用 T 表「一次交易凭据」——与 domainA 的 T 冲突 */ }
```

`test/fixtures/README.md`：写明 **RED→GREEN procedure**——预置 grill 答案="加 domainB、并列 domain、核心服务=`test/fixtures/svc-fake`、探索 skip"；4 项判据：**I1** 不改 `knowledge/domainA/*`、**I2** 检出 `T` 碰撞并 grill/消歧、**I3** 新 `recall-domainB-knowledge` 无 repo-relative 路径/host:port、**I5** additive 不当 greenfield（问了关系）。

- [ ] **Step 4: 验证 fixture 结构良好**

Run:
```bash
find test/fixtures/kb-domainA -type f | sort
grep -n 'T ' test/fixtures/kb-domainA/knowledge/glossary.md
grep -rn 'moego' test/fixtures/ || echo "I8 OK: no internal tokens"
```
Expected: 5 个 fixture 文件在列；glossary 有 `T` 词条；grep `moego` 无命中（I8）。

- [ ] **Step 5: Commit**

```bash
git add test/fixtures/
git commit -m "test(start-distill): RED→GREEN fixture KB（含 domainA 与 overloaded 词 T）"
```

---

### Task 2: SKILL.md 骨架 — frontmatter + 角色/边界/altitude/交棒

**Files:**
- Create: `.claude/skills/start-distill-domain-knowledge/SKILL.md`

**Interfaces:**
- Produces: 合法可加载的 skill（`name` + 触发性 `description`）；§角色/§altitude/§交棒 三节（内容依 spec §1/§2）。后续 Task 3/4 往同一文件追加决策树与不变量。

- [ ] **Step 1: 写 frontmatter + 标题 + 角色边界**

```markdown
---
name: start-distill-domain-knowledge
description: Use to START distilling a NEW domain into a knowledge base — the per-domain 开张 wizard that scaffolds/extends the fixed KB structure, grills domain identity (boundary + core services), optionally scouts an initial backlog, then hands off to distill-domain-knowledge for per-slice work. Handles both greenfield (fresh KB) and additive (add a domain to an existing KB) via runtime grill.
---

# start-distill-domain-knowledge — per-domain 开张向导

（角色/边界：见 spec §1/§2——per-domain 跑一次 genesis；仓累积多域；交棒 distill、不做 per-slice、不装协议 skill。）
```

- [ ] **Step 2: 写 altitude 模型 + "Scaffold" 消歧 + 交棒**

内联 spec §2 的三级粒度图（仓/KB ⊇ domains ⊇ slices）、§2.1 "Scaffold" 消歧表、STEP4 末的交棒话术（"挑 backlog 一个子域，对我说 /distill…"）。

- [ ] **Step 3: 验证 skill 可加载 + description 触发词**

Run:
```bash
head -5 .claude/skills/start-distill-domain-knowledge/SKILL.md
grep -c 'per-slice\|per-domain\|交棒' .claude/skills/start-distill-domain-knowledge/SKILL.md
```
Expected: frontmatter 合法（`name:` 正确）；关键定位词命中 ≥2。

- [ ] **Step 4: Commit**

```bash
git add .claude/skills/start-distill-domain-knowledge/SKILL.md
git commit -m "feat(start-distill): SKILL.md 骨架（frontmatter+角色/altitude/交棒）"
```

---

### Task 3: SKILL.md — STEP0–4 运行时决策树

**Files:**
- Modify: `.claude/skills/start-distill-domain-knowledge/SKILL.md`（追加决策树节）

**Interfaces:**
- Consumes: Task 2 的角色框架。
- Produces: STEP0–4 完整分支（含每步引用的 distill reference 与不变量 tag）。Task 7 的 RED/GREEN 判据对应这些分支。

- [ ] **Step 1: 内联决策树（verbatim 自 spec §4）**

把 spec §4 的 ASCII 决策树全文写入 SKILL.md，逐步补充执行指令：
- STEP0：检测 `knowledge/ + glossary.md + recall-*-knowledge` → additive；否则 greenfield 且**先 confirm 落点**；distill-me 守卫靠本仓 CLAUDE.md（Task 6）。
- STEP1（仅 additive）：**recall@start 反哺** → 关系 grill（并列/独立 KB/sub-domain，带推荐答案）——引用 `references/grilling.md`（一次一分支、每问带荐答）。
- STEP2：grill 边界+核心服务 → **让用户指定每个仓本地路径** → 存在性核验（可读开 STEP3；不可读 warn+记 gaps+跳过）；glossary 收录 umbrella 词、additive 先跨域碰撞（引用 `references/glossary-first.md`）。
- STEP3：可选，opt-in 后问深度（默认中=排序菜单），可 skip。
- STEP4：非破坏式落产物；`index@end`（引用 `references/memory-loop.md`）；交棒。

- [ ] **Step 2: 加"分支值运行时决定、绝不硬编码"声明**

一句强调：greenfield/additive、归属关系、侦察深度均运行时 grill 定。

- [ ] **Step 3: 走查覆盖验证**

Run:
```bash
grep -n 'STEP0\|STEP1\|STEP2\|STEP3\|STEP4' .claude/skills/start-distill-domain-knowledge/SKILL.md
grep -c 'references/grilling\|references/glossary-first\|references/memory-loop' .claude/skills/start-distill-domain-knowledge/SKILL.md
```
Expected: STEP0–4 五步齐；三个 reference 引用各 ≥1（复用不复制）。

- [ ] **Step 4: Commit**

```bash
git add .claude/skills/start-distill-domain-knowledge/SKILL.md
git commit -m "feat(start-distill): STEP0–4 运行时决策树"
```

---

### Task 4: SKILL.md — 不变量 I1–I8

**Files:**
- Modify: `.claude/skills/start-distill-domain-knowledge/SKILL.md`（追加不变量节）

**Interfaces:**
- Produces: I1–I8 表（含来源）+ "I1↔I2 不矛盾"注。每条 = Task 7 的一个 RED 判据。

- [ ] **Step 1: 内联 I1–I8 表（verbatim 自 spec §5）**

把 spec §5 不变量表 + "I1↔I2 不矛盾"注全文写入。关键：I1 只锁**知识文件**、I2 授权 glossary 消歧式 additive 编辑、I3 stub 可移植、I4 greenfield confirm、I5 关系-grill 仅 additive、I6 Memory loop、I7 不装 skill/交棒、I8 public-repo hygiene。

- [ ] **Step 2: 验证不变量齐全 + 有来源**

Run:
```bash
grep -c 'I[1-8]' .claude/skills/start-distill-domain-knowledge/SKILL.md
grep -n 'find-domain-knowledge/SKILL.md:49\|immutability\|铁律#2' .claude/skills/start-distill-domain-knowledge/SKILL.md
```
Expected: I1–I8 八条在列；关键来源锚点命中。

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/start-distill-domain-knowledge/SKILL.md
git commit -m "feat(start-distill): 不变量 I1–I8"
```

---

### Task 5: templates/（5 个模板，全占位符）

**Files:**
- Create: `.claude/skills/start-distill-domain-knowledge/templates/kb-CLAUDE.md`
- Create: `.../templates/glossary.md`
- Create: `.../templates/gaps.md`
- Create: `.../templates/backlog.md`
- Create: `.../templates/recall-DOMAIN-knowledge.SKILL.md`

**Interfaces:**
- Consumes: SKILL.md STEP4 引用这些模板名。
- Produces: STEP4 落产物时填充的占位符模板（`{{DOMAIN}}` / `{{MEMORY_NAMESPACE}}` 等）。

- [ ] **Step 1: 写 `kb-CLAUDE.md`（薄指针 + 三铁律一行安全网）**

```markdown
# {{DOMAIN}} 知识库

本仓是 `{{DOMAIN}}` 的蒸馏知识库（distill-me workflow 产物）。

**要扩展/更新它**：用 distill-me workflow——加载 `distill-domain-knowledge` skill，遵其纪律。
**记忆命名空间**：`{{MEMORY_NAMESPACE}}`（知识召回/写入必带此键）。

## 三铁律（安全网；详见 distill-domain-knowledge）
1. 落库前先验证：机制类断言默认未验证，代码证实/证伪后才入库。
2. 术语先行：新名词先过 glossary 准入（查重+消歧）再扩散。
3. 知识分层：领域语义→knowledge/；架构断言→记忆库时点条目；代码坐标作锚点。
```

- [ ] **Step 2: 写 `glossary.md` / `gaps.md` / `backlog.md` 表头**

`glossary.md`：
```markdown
# {{DOMAIN}} Glossary — ubiquitous language
> 一概念一定义；一词多义强制消歧（scope 限定/拆条）。headword 用术语原文。

| 术语 (headword) | 规范定义 | scope-type | 锚点/来源 |
|---|---|---|---|
```

`gaps.md`：
```markdown
# pending gaps — 待验证 punch-list
> distill ③Verify 产出；代码核验后清空。

| # | claim/疑点 | why unresolved | what's needed | owner |
|---|---|---|---|---|
```

`backlog.md`：
```markdown
# backlog — 候选子域账本
> start-distill 产初始（全 pending）；下游 distill/#2 更新 status、追加候选。

| # | 候选子域 | why worth（最常问/最易错）| 核心服务（代码锚点）| status（pending/in-progress/done）| 优先级 |
|---|---|---|---|---|---|
```

- [ ] **Step 3: 写 `recall-DOMAIN-knowledge.SKILL.md`（L1 stub，可移植）**

```markdown
---
name: recall-{{DOMAIN}}-knowledge
description: Load distilled {{DOMAIN}} knowledge before reasoning about {{DOMAIN_ENTITIES}}. Routes to the {{DOMAIN}} KB's domain skills + memory. Invoke BEFORE concluding about {{DOMAIN}}.
---

# Recall {{DOMAIN}} Knowledge

本 KB 覆盖 **{{DOMAIN}}**（{{DOMAIN_ENTITIES}}）。这是 {{DOMAIN}} KB 的 L1 recall 入口。

## 加载什么
- 领域 skills（随 distill 产出补全）：{{DOMAIN_SKILLS_LIST}}
- 记忆命名空间：`{{MEMORY_NAMESPACE}}`（用可移植 memory MCP 工具按此键召回）

## 可移植性约束（勿改成本机相关）
- 只依赖**全局 skill**（Skill 工具）+ **可移植 memory MCP 工具**。
- 禁 repo-relative `knowledge/*.md` 路径、禁 host:port REST 端点。
```

- [ ] **Step 4: 验证占位符 + I8 洁净**

Run:
```bash
grep -rl '{{DOMAIN}}' .claude/skills/start-distill-domain-knowledge/templates/ | wc -l
grep -rn 'moego\|knowledge/[a-z]*\.md\|127.0.0.1\|:3111' .claude/skills/start-distill-domain-knowledge/templates/ || echo "I8+I3 OK: 无内部内容/repo 路径/host:port"
```
Expected: 多个模板含 `{{DOMAIN}}`；无 `moego`/repo-relative 路径/host:port 命中。

- [ ] **Step 5: Commit**

```bash
git add .claude/skills/start-distill-domain-knowledge/templates/
git commit -m "feat(start-distill): 5 个占位符模板（kb-CLAUDE/glossary/gaps/backlog/L1 stub）"
```

---

### Task 6: distill-me 本仓 CLAUDE.md 守卫行（STEP0 guard, I4）

**Files:**
- Modify: `CLAUDE.md`（distill-me 根，"仓库结构"节后加一条）

**Interfaces:**
- Consumes: SKILL.md STEP0 依赖"本仓 CLAUDE.md 有守卫"这一前提。
- Produces: 在 distill-me 仓内跑 start- 时，agent 读到此行 → refuse 建库。

- [ ] **Step 1: 加守卫行**

在 `CLAUDE.md` "## 仓库结构" 节末追加：
```markdown
- **本仓是 workflow 本体，不在此 distill / 建知识库**：`start-distill-domain-knowledge` 检测到本仓（有 `.claude/skills/` 却无 `knowledge/`）时应 refuse——建库请到你自己的领域仓。
```

- [ ] **Step 2: 验证**

Run: `grep -n '不在此 distill' CLAUDE.md`
Expected: 命中该守卫行。

- [ ] **Step 3: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: 加'本仓不在此建库'守卫（start-distill STEP0 guard）"
```

---

### Task 7: RED→GREEN 验证（整体测试门）

**Files:**
- Create: `docs/superpowers/specs/2026-06-30-start-distill-domain-knowledge-redgreen-log.md`（记录 RED/GREEN 结果）

**Interfaces:**
- Consumes: Task 1 fixture + Task 2–6 的 skill。
- Produces: RED→GREEN 判定证据（GREEN 四项全清、RED 至少违 1 项）。

- [ ] **Step 1: RED 基线（无 skill）**

在 fixture **副本**里（`start-distill-domain-knowledge` **不装**进 fixture 的 `.claude/skills/`，隔离即净基线），用 **Opus 档** agent 跑预置任务："在此 KB 加 domainB、并列 domain、核心服务=`test/fixtures/svc-fake`、探索 skip"。记录它是否 ① 改了 `knowledge/domainA/*`（违 I1）② 漏 `T` 碰撞（违 I2）③ 新 stub 硬编码路径（违 I3）④ 当 greenfield 跳关系（违 I5）。

- [ ] **Step 2: GREEN（带 skill）**

同一 fixture 副本，这次让 agent 加载 `start-distill-domain-knowledge`，同样预置输入复跑。

- [ ] **Step 3: 判定 + 记录**

判据：**GREEN 四项全清**（domainA 未动、`T` 碰撞被 grill/消歧、新 stub 可移植、问了关系）**且 RED 至少违 1 项**。写入 red-green-log.md（含两次运行的产物 diff 摘要）。
Expected: GREEN pass、RED fail —— skill 证明有增益。若 GREEN 未全清 → 回 Task 3/4 补 SKILL.md 对应分支/不变量，重跑。

- [ ] **Step 4: Commit**

```bash
git add docs/superpowers/specs/2026-06-30-start-distill-domain-knowledge-redgreen-log.md
git commit -m "test(start-distill): RED→GREEN 验证记录（GREEN 四项清）"
```

---

### Task 8: ROADMAP #1 更新

**Files:**
- Modify: `ROADMAP.md`（#1 条目）

**Interfaces:**
- Produces: #1 反映已更名 `start-distill-domain-knowledge`、已出 spec+plan、状态 in-progress/done。

- [ ] **Step 1: 更新 #1 文字**

把 `ROADMAP.md` #1 的 `init-knowledge-base` 更名为 `start-distill-domain-knowledge`，加一行指向 spec/plan 路径，标注"设计已定、实现中"。

- [ ] **Step 2: 验证**

Run: `grep -n 'start-distill-domain-knowledge' ROADMAP.md`
Expected: #1 已更名并有指向。

- [ ] **Step 3: Commit**

```bash
git add ROADMAP.md
git commit -m "docs(roadmap): #1 更名 start-distill-domain-knowledge，链 spec+plan"
```

---

## Self-Review

**1. Spec coverage**（逐节对账）：
- §1 目的 → Task 2 角色。§2 altitude/Scaffold 消歧 → Task 2。§3 造法（独立 skill+复用 refs）→ Task 2/3。§4 决策树 → Task 3。§5 I1–I8 → Task 4。§6 产物（模板/stub/backlog/KB CLAUDE.md）→ Task 5。§7 RED→GREEN → Task 1（fixture）+ Task 7（跑）。§8 命名 → Task 2 frontmatter。§10 action item（distill-me 守卫）→ Task 6；ROADMAP → Task 8。**无遗漏**。
- §6.7"不同 KB 机制"（另一路径 greenfield）→ 已入 Task 3 STEP1 决策树文本，无独立任务（属决策树分支，不新增文件）。

**2. Placeholder scan**：模板里的 `{{DOMAIN}}` 等是**产物设计的占位符**（I8 要求），非计划占位；每个 code step 均给实际内容；RED→GREEN 步给了预置输入与判据，非"写测试略"。✓

**3. Type/naming consistency**：全程 `start-distill-domain-knowledge`（无简称漂移）；STEP0–4、I1–I8、模板文件名在 Task 2–7 间一致引用；fixture 域名 `domainA`(既有)/`domainB`(新增)、overloaded 词 `T` 三处一致。✓
