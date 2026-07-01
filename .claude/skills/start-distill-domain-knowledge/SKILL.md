---
name: start-distill-domain-knowledge
description: Use to START distilling a NEW domain into a knowledge base — the per-domain 开张 wizard that scaffolds/extends the fixed KB structure, grills domain identity (boundary + core services), optionally scouts an initial backlog, then hands off to distill-domain-knowledge for per-slice work. Handles both greenfield (fresh KB) and additive (add a domain to an existing KB) via runtime grill.
---

# start-distill-domain-knowledge — per-domain 开张向导

## 角色与边界

本 skill 是 **per-domain 开张向导 + 分发器**：一个领域开张一次，产出初始 backlog 后**交棒** `distill-domain-knowledge` 做 per-slice 蒸馏。

四件事：
1. 搭/扩固定 KB 结构
2. Grill 领域身份（边界 + 核心服务）
3. 可选方向性侦察（opt-in）
4. 产初始 backlog → 交棒

**不**做的事（出界）：
- 不做 per-slice 蒸馏（`distill-domain-knowledge` 职责）
- 不装协议 skill（B1，宿主手动）
- 不接管 B1/B2 bootstrap

## Altitude 模型

三级粒度，本 skill 工作在**领域级**：

```
仓库 / KB
  ├─ 领域 domainA   ← 之前 start- 过一次（已有 knowledge/domainA/ + recall-domainA-knowledge + glossary 的 domainA 词条）
  └─ 领域 domainB   ← 现在 start-distill-domain-knowledge 要新增的      ← 本 skill 粒度（per-domain，跑一次 genesis）
        slice s1 / s2 / s3 …                                          ← distill-domain-knowledge 粒度（per-slice，反复跑）
```

- **per-domain，跑一次 genesis**：一个领域开张一次；一个仓**累积多个领域**。
- **交棒，不越界**：产出初始 backlog 后交给 `distill-domain-knowledge`；**不**做 per-slice 蒸馏，**不**装协议 skill。

### "Scaffold" 术语消歧（glossary-first）

本 skill 的"scaffold"与 `distill-domain-knowledge` 的 **①Scaffold 步不是一回事**：

| | distill ①Scaffold（`distill-domain-knowledge/SKILL.md:57`） | 本 skill 的 scaffold |
|---|---|---|
| 层级 | **内容**骨架：某切片的 typed entity catalog | **容器** + 领域身份 + 粗粒度域级侦察 |
| 粒度 | per-slice | per-domain |
| 前提 | **假定**目录结构已存在 | **创建/扩展**目录结构 |

→ 本 skill 内部步骤一律用 **STEP0–4**，避开裸 "Scaffold"。

### 与 `AGENTS.md §B` 的关系

`§B` 是跨宿主手工 bootstrap prose，保留为可移植降级路径。本 skill 是 **Claude-Code-native 自动化路径**，覆盖**加强版 B3**（搭骨架 + 领域 grill + 侦察）。**不**接管 B1（装协议 skill）、B2（装外部 plugin）；B0 能力自检 / B4 就绪自检以**报告**呈现，不强制。

## 运行时决策树（STEP0–4）

> **所有分支值（greenfield/additive、归属关系、侦察深度）均运行时由 grill 确定，绝不在设计/代码硬编码。** 设计交付的是这棵树的形状 + 不变量 + 产物。

```
/start-distill-domain-knowledge "我要蒸馏 domainB 领域"
│
├ STEP0 检测环境
│   有 knowledge/ + glossary.md + recall-*-knowledge?
│   ├ 有既有 KB 结构 → ADDITIVE
│   └ 无 → GREENFIELD，但**先 confirm 落点**（"这里没有既有 KB，在 <cwd> 新建 domainB 知识库吗?"）  ← I4
│   ※ "别在 distill-me 本仓建" 的守卫**不进本 skill**（保持通用），由 distill-me 自己的 CLAUDE.md 一句话挡
│
├ STEP1 [仅 ADDITIVE] recall@start 反哺 + 关系 grill                                                 ← I6
│   先 recall 记忆库（domainB 是否已被提及挂在某域下?）→ 带提示问归属：
│   ┌ 同 KB 并列 domain → knowledge/domainB/ 与 domainA/ 并列；新 recall-domainB-knowledge；共享命名空间
│   ├ 不同 KB           → 用户指定**另一路径** greenfield 起新结构；独立路由；独立命名空间
│   └ 大 domain 的 sub-domain → 嵌套 knowledge/<大域>/domainB/；**扩展**既有路由；沿用大域命名空间
│   （引用 distill-domain-knowledge/references/grilling.md：一次一分支、每问带荐答）
│
├ STEP2 grill 领域身份（both）
│   边界（domainB 含/不含什么）? 核心服务 = 哪些代码仓/路径?
│   → **让用户指定每个核心服务的本地路径** → 存在性核验：                                            ← I3 输入
│       可读 → 开 STEP3；不可读 → warn + 记 gaps + 对该服务**跳过** STEP3
│   → glossary 收录 umbrella 词；additive 时**先**跨域碰撞检查                                       ← I2
│   （引用 distill-domain-knowledge/references/glossary-first.md：碰撞检查纪律）
│
├ STEP3 [可选，opt-in 后问深度] 方向性探索
│   扫核心服务 → 初始 backlog；默认**提议跑**、深度**必问**（轻/中，默认中=排序菜单）、可 skip
│
└ STEP4 落产物（非破坏式）                                                                          ← I1
    greenfield：建容器 + 第一个域 + L1 stub + 命名空间 + 薄指针 CLAUDE.md
    additive：  只新增/扩展，绝不覆盖既有域**知识文件**
    → index@end（记忆库落 namespace + umbrella 词 + backlog）                                       ← I6
    （引用 distill-domain-knowledge/references/memory-loop.md：index@end 纪律）
    → 交棒："挑 backlog 一个子域，对我说 /distill…"
```

### 执行纪律

- **STEP2 grill**：每问一次只问一个分支，带推荐答案（参见 `distill-domain-knowledge/references/grilling.md`）。
- **STEP2 glossary 碰撞**：additive 时必须先检查所有已有词条再落新词（参见 `distill-domain-knowledge/references/glossary-first.md`）。
- **STEP4 index@end**：Memory loop 收尾必走（参见 `distill-domain-knowledge/references/memory-loop.md`）。
- **所有产物模板**：见 `templates/` 目录（占位符 `{{DOMAIN}}` 等在此 KB 的产出时填真值）。

## 交棒

STEP4 完成后，明确告知用户：

> "domainB 已开张。挑 backlog 一个子域，对我说 `/distill-domain-knowledge <子域>` 开始 per-slice 蒸馏。"
