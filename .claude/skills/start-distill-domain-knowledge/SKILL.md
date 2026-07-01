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

## 交棒

STEP4 完成后，明确告知用户：

> "domainB 已开张。挑 backlog 一个子域，对我说 `/distill-domain-knowledge <子域>` 开始 per-slice 蒸馏。"
