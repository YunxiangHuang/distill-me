# AGENTS.md

本文件有两个用途：**A. 你正在本仓库内工作** → 读第一节后转 CLAUDE.md；**B. 你被要求把"知识蒸馏环境"搭到一个新环境/新仓库** → 执行第二节的 bootstrap 流程。

---

## A. 在本仓库内工作

工作规则与 [CLAUDE.md](CLAUDE.md) 完全一致（三条铁律：落库前先验证 / 术语先行 / 知识分层存放）。如果你的宿主不自动加载 CLAUDE.md，现在去读它。

---

## B. Bootstrap：为用户搭建蒸馏环境（面向 agent 的执行指南）

目标：让用户在自己的领域跑通 `Scaffold → Grill → Verify → Capture → Glossary → Distill` 六步协议（外加两条贯穿全程的横切纪律：Glossary-first 准入；Memory loop——仅当环境配有持久记忆库时必走）。适用于 Claude Code、Codex 及任何具备**文件读写 + 子任务**能力的 coding agent。

### B0. 能力自检（先做，决定后续降级路径）

逐项检查你的宿主能力并记录结果：

| 能力 | 具备 → 用法 | 缺失 → 降级路径 |
|---|---|---|
| Skill / 指令文件加载机制 | 直接加载 SKILL.md | 把两个 SKILL.md **全文读入上下文**当作行为指令，效果等同 |
| 并行子 agent / workflow 编排 | Verify 阶段 fan-out（按代码区域分组并行核验） | 顺序逐条验证——慢但结论等效，不许跳过 |
| 结构化提问 UI | 单选题 + 推荐答案置顶 | 纯文本提问：**一次只问一个分支，每问必带你的推荐答案**供专家确认/纠正 |
| 持久记忆 / MCP | 执行 Memory loop：召回@①、复验所触条目@③、落库即索引（修订先删后写）@④、收尾对账@⑥ | 跳过 Memory loop，知识只存 markdown 文件（协议核心不受影响） |
| APM / 可观测 MCP | 缺陷定级前查生产流量与失败位置 | 跳过动态验证，在 gaps 中记"生产可达性未验证" |

### B1. 安装协议 skills（2 个，均在本仓库 `.claude/skills/` 内）

> ⓘ 归属：`distill-domain-knowledge` 为本仓自有；`grill-me` **vendor 自上游 [`github.com/mattpocock/skills`](https://github.com/mattpocock/skills)**（忠实拷贝，见 `.claude/skills/grill-me/NOTICE`），随仓携带仅为 clone 即用。也可改装上游版后指向它。

- **Claude Code**：`ln -s <本仓库>/.claude/skills/distill-domain-knowledge ~/.claude/skills/` 同理 `grill-me`（或 cp -r）。
- **Codex**：复制到 `~/.agents/skills/`。
- **其他 agent**：读取两个 SKILL.md 全文，作为本次及后续蒸馏会话的系统级指令。

### B2.（仅 Claude Code，推荐非必需）安装外部 plugin

```
/plugin marketplace add obra/superpowers-marketplace   # → install superpowers（RED→GREEN skill 测试方法论）
/plugin marketplace add rohitg00/agentmemory           # → install（跨会话记忆；配置要点与坑见 docs/agentmemory-setup.md，装好后按 B5「记忆库命名空间」定 project 键）
```

装不了不阻塞：`distill-domain-knowledge` 的 ⑥ Distill 步已内嵌 RED→GREEN 最小要求（无 skill 的 agent 在典型任务上犯错 → 带 skill 复跑不犯，方可上线）。

### B3. 搭目标仓库骨架

在用户指定的位置创建：

```
<their-distill>/
├── CLAUDE.md            # 从本仓库 CLAUDE.md 抄"核心 workflow + 三条铁律"，删去示例域专有的术语/进度/基建节
├── AGENTS.md            # 若宿主不读 CLAUDE.md：symlink 或复制，保证规则双入口可达
├── knowledge/
│   └── glossary.md      # 初始化为空表 + 表头说明（一概念一定义；一词多义强制消歧）
├── pending/
│   └── gaps.md          # 表头：| # | claim/疑点 | why unresolved | what's needed | owner |
└── .claude/skills/      # B1 的两个 skill（如果用户希望仓库自包含）
```

### B4. 自检（向用户报告后才算就绪）

1. 确认两个 skill 可加载/已读入；
2. 确认目标代码仓库路径可读（验证环节要扫真实代码——**这是硬前提**）；
3. 对用户说："环境就绪。对我说『开始蒸馏我自己』即可开跑，第一题建议选你最常被问、或新人最常做错的子域。"

### B5. 运行期纪律（执行蒸馏时你必须遵守的最小集）

- **Grill**：一次一个分支；每个问题带你的推荐答案；专家说"不确定"→ 记 gaps，不许编造。
- **Verify**：机制类断言逐条拿真实代码证实/证伪；`✅` 必须带 `路径:行号` 锚点，无锚点不得标 ✅；偏差**带证据回炉再问**，绝不静默取舍。
- **Answer-mismatch**：问 A 答 B（兄弟方法/路径）时，错位本身即断言——当场验证 A↔B 关系并同轮反馈。
- **Glossary-first**：新名词（含你自创的文件名/skill 名）出现的瞬间过术语表准入（查重→查组织既有含义→定义+scope-type），之后才允许扩散。
- **Memory loop**（仅当配有记忆库）：①召回起步、③召回条目即复验、④落库即索引（修订**先删旧条目再写**——旧快照会在召回时压过新知识）、⑥收尾对账所触条目。已验证的架构副产品入记忆库（是已解决事实，**不进** gaps punch-list）；切片留下过期条目不算完成。
- **记忆库命名空间**（配 agentmemory 时必做，防工具流水噪音霸榜召回）：为 curated 知识固定一个**含 `/` 的规范 project 键**（如 `knowledge/<你的域>`）——hook 自动捕获的 obs 其 project 取 git 目录 basename，含斜杠的键结构性免撞（勿用真实 owner/repo 字符串）。知识召回必带该键过滤、写入走 REST `remember` 必带该键（MCP `memory_save` 的 project 参数不落库，v0.9.27 实测）；哨兵：知识召回命中 `obs_` 前缀条目 = 命名空间泄漏。细节见 docs/agentmemory-setup.md。
- **产物验收**：`knowledge/<域>/*.md`（带锚点）+ glossary 增量 + gaps punch-list + 过了 RED→GREEN 的 skill。四样缺一不算完成；配有记忆库时外加第五样：**memory 增量**（新/修订知识已重索引、所触条目已对账）。

### 预期管理（提前告知用户）

预期基线：单子题一个会话；专家口述机制断言会有相当比例被代码验证标记偏差（分层/版本视角差，属正常）；专家的投入只是回答问题与裁决偏差，不写文档。
