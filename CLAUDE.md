# distill-me — 领域知识蒸馏 workflow（可迁移 · 可复用）

**首要目的**：构建可迁移、可复用的领域知识蒸馏 workflow / skills——六步协议 + Glossary-first + Memory loop，让任何领域专家把脑中隐性知识外化成**被代码验证的**知识库 + 可触发的 skills。本仓是 **workflow 本体**；要在你自己的领域建知识库，见 [README.md](README.md) / [AGENTS.md](AGENTS.md) 的复用指南。

## 仓库结构

- `.claude/skills/distill-domain-knowledge/` — 协议本体（spine `SKILL.md` + `references/` 单步 depth，按作用域拆：跨阶段纪律常驻骨架、单阶段细节下沉 reference）。
- `.claude/skills/find-domain-knowledge/` — 双层召回的 **L2 通用发现 skill**（零领域细节，按命名约定 `recall-<域>-knowledge` 路由到各知识库的 L1 入口）。
- `.claude/skills/grill-me/` — Grill 步的访谈子工具（**vendor 自 [`mattpocock/skills`](https://github.com/mattpocock/skills)**，MIT；归属见其 `NOTICE`）。
- `README.md` / `AGENTS.md` — 复用指南（人读 / agent 读）。
- 你 fork 后建的知识库会另加 `knowledge/`、`knowledge/glossary.md`、`pending/gaps.md`——它们属**你的知识库**、不属本 workflow 仓。

## 核心 workflow

`Scaffold → Grill → Verify(代码证实/证伪) → Capture[只落已验证] → Glossary → Distill[RED→GREEN]`，**Memory loop 贯穿**（有记忆库则必走，是协议步骤而非旁挂约定）：召回@① · 复验所触条目@③ · 落库即索引[修订先删后写]@④ · 收尾对账@⑥——切片留下过期条目不算完成。协议细节见 `distill-domain-knowledge` skill。

## 核心规则

1. **落库前先验证**：专家口述的机制类断言默认未验证，必须代码证实/证伪后才进 knowledge / skill；有偏差**回炉 grill**，绝不静默取舍（偏差的成因往往本身就是知识）。"为什么 / 业务意图"类专家权威。
2. **术语先行**：新名词（含 AI 自创的文件名 / skill 名）出现的瞬间过术语表准入——查重、查组织内既有含义、定义 + scope-type——之后才允许扩散。二义会静默反转业务含义（"用户 = 商家还是顾客？"答反代码就反）；AI 自创名撞既有产品名 → 一连串改名，准入时一问可免。
3. **知识分层存放**：①领域语义（机制 / 不变量 / 术语 / 意图）→ distill 管线进 `knowledge/` + skill——叙述性最强、无验证时腐烂最快；②架构断言 / 拓扑 / 方向 → 记忆库**时点验证条目 + 召回即复验**——天然随重构漂移，验证成本按使用摊销；③代码坐标（类 / 表 / 位置）最稳定，作前两层的锚点。

## 通用编码 / 协作原则

- 中文回复，技术术语 / 代码标识符 / 代码注释保持英文；解释"为什么"而非只说"怎么做"。
- TDD / KISS / YAGNI / DRY；可读性优先；注释解释"为什么"。
- 架构决策、重大变更、多方案时先询问；简单任务直接执行。不确定时明确说明。
