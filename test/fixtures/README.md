# RED→GREEN Fixture — start-distill-domain-knowledge

## 用途

本 fixture 是 `start-distill-domain-knowledge` skill 的 RED→GREEN 验证现场。

## Fixture 结构

- `kb-domainA/` — 既有 domainA 知识库（含 overloaded 词 `T`）
- `svc-fake/` — 假核心服务代码靶（STEP2 路径核验 + STEP3 侦察用）

## RED→GREEN Procedure

**预置 grill 答案**（两次运行均用此输入，确保可复现）：
- 目标：加 domainB 到此 KB
- 归属关系：并列 domain（domainB 与 domainA 并列）
- 核心服务路径：`test/fixtures/svc-fake`
- 侦察深度：**探索 skip**（绕开 STEP3 fan-out 嵌套测试上限）

**RED 基线**（无 skill）：`start-distill-domain-knowledge` 不装进 `kb-domainA/.claude/skills/`，在此 fixture 上直接跑上述任务。

**GREEN**（带 skill）：同一 fixture，加载 `start-distill-domain-knowledge` 后复跑。

## 4 项判据

| 项 | 不变量 | RED 期望（无 skill）| GREEN 期望（带 skill）|
|---|---|---|---|
| **I1** | 非破坏式：不改 `knowledge/domainA/*` | 可能违反（覆盖/删改 domainA 文件）| 严格不触碰既有域知识文件 |
| **I2** | glossary `T` 碰撞检出并消歧 | 可能漏检（直接用 T 或静默覆盖）| 检出碰撞、grill 专家、消歧式 additive 编辑 |
| **I3** | 新 `recall-domainB-knowledge` stub 无 repo-relative 路径/host:port | 可能硬编码本机路径 | 只依赖全局 skill + 可移植 memory MCP |
| **I5** | additive 时问归属关系（不当 greenfield 跳过）| 可能跳过关系-grill | 问归属（并列/独立/sub-domain）后再落产物 |

判定：GREEN 四项全清 **且** RED 至少违 1 项 → skill 证明有增益。
