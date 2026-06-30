# distill-me — Roadmap

> 方向性待办（raw 想法，多数尚未设计）。启动任一条前走完整 brainstorming → spec → plan。
> 这是 distill-me **workflow 本身**的演进，与"用它蒸馏出的领域知识"无关。

## 形态：往 Claude Code plugin 演进

当前分发 = 手动 bootstrap（clone + symlink/cp，见 [AGENTS.md](AGENTS.md)）。摩擦点：安装步骤多、无版本化升级、skill 更新靠各自重新 symlink、约定（如记忆库命名空间）靠人记。目标是 **plugin 形态一条命令安装、升级随 marketplace 走**。

- **M1 — plugin clamshell**：仓库根加 `.claude-plugin/`（plugin.json + marketplace.json），skills 暴露为 plugin skills；安装从手动 bootstrap 降级为可选（非 Claude Code 宿主仍需）。
- **M2 — bootstrap command**：一个 `bootstrap` command 替代手工搭骨架（含宿主能力自检与降级矩阵的交互式落地）。与下方 #1 同诉求，定形态时合并。
- **M3 — 守护 hooks（可选）**：记忆库命名空间哨兵、Memory loop 收尾检查等。

> ⚠️ 约束：plugin cache 会被升级覆盖——任何需要用户本地修改的内容不得进 plugin。对 SKILL.md 的任何修改仍走 RED→GREEN。plugin/command/marketplace 名都是自创名，扩散前过 glossary-first 准入。

## 工作流能力待办

1. **`init-knowledge-base` skill（建库脚手架）**：从"领域名 + 关键代码仓"一键 scaffold 标准骨架（目录约定 + 空 glossary + gaps + 派生 CLAUDE.md + 记忆库命名空间约定 + **L1 `recall-<域>-knowledge` 召回路由 stub**），替代手动 bootstrap。（与 M2 同诉求。）L2 通用发现层（`find-domain-knowledge`）已在仓内常驻、零领域细节、靠命名约定路由；L1 则是**每域一个的薄路由**（不抄 domain facts 防腐烂、跨 repo 可达，构造约束见 `find-domain-knowledge` SKILL「Building a recall-*-knowledge (L1) entry」），属 scaffold 产物而非通用层——现状要手写,应由 scaffold 自动产出 stub 以免漏建（编排型 workflow 里领域知识不自燃，缺 L1 入口召回链就断）。
2. **checkpoint 机制**：一轮蒸馏（Scaffold→Grill→Verify→Capture→Glossary→Distill）链长、跨多步，中断即丢进度。定义 checkpoint 捕获什么——已 grill 的问答与裁决、Verify 已证实/证伪条目、未决 gaps、当前 slice 状态——使新会话能 resume 而非重来。
3. **大领域的分步骤蒸馏指引**：大领域每次开工都要重跑枚举/扫描去发现"还有哪些子片没蒸"，重复且贵。沉淀一份**持久的领域分解图**（子片清单 + 各自状态 done/pending/blocked + 推荐顺序）作活文档随蒸馏更新，新会话读它直接挑下一片，不重做 discovery。
4. **领域 scope 锚点**：没有显式声明"本域活在哪些服务/仓库/目录"，探索就**过度发散**——agent 漂进邻域、反复要人工校准焦点。每个域一个 **scope 锚点**产物（key services / repos / dirs + 明确的 out-of-scope 列表），喂给验证/探索 agent 约束搜索面；也是 #1 的输入。与 #3 互补：#4 划边界（哪里找）、#3 排顺序（先蒸哪片）。
5. **定制 Agent（减少内置 agent 摩擦）**：内置 agent 常带默认行为坑——例如只读探索 agent 默认跑在轻量模型上，判别性任务（零遗漏审计/证伪/completeness）不可信，每次都要手动覆盖模型。项目自带 agent 类型（只读 + 强模型的 explorer、审计用 critic），把模型/工具/effort 预设对，免去逐调用覆盖与复发踩坑。（实证：workflow `agentType:'Explore'` 继承 agent 定义里的 Haiku frontmatter、**不**继承主循环强模型，判别性阶段须显式传 `model`+`effort`，否则静默降级——已两次踩坑。）

> 路线在使用中反复打磨，随实践更新。
