# agentmemory — setup & gotchas（可选基建）

> distill-me 的 Memory loop 可**可选**接入跨会话语义记忆。本仓示例用的是开源 [agentmemory](https://github.com/rohitg00/agentmemory)（经 Claude Code plugin 接入：MCP server + hooks + skills；REST 锚点默认 `127.0.0.1:3111`）。
> 以下不是 agentmemory 官方文档，而是**用它跑蒸馏 Memory loop 实战踩出的配置要点与坑**——换别的记忆库时，"为什么这么做"多半同样适用。

## 配置要点

- `EMBEDDING_PROVIDER=local`（向量本机跑，不外发）。
- LLM 端点用你自己的——consolidation / summarize / reflect 都需要 LLM；**纯 noop（无 LLM）下 Observation 无转化路径**（semantic 层吃 session summaries、reflect 无 fallback），只开 `CONSOLIDATION_ENABLED` 无效。
- consolidation / graph-extraction / auto-compress / inject-context 视需要开；**auto-compress** 在捕获即 LLM 消化，治 hook 工具流水噪音于源头。

## 启动 / 重启坑

- **重启前必先清残留**：`agentmemory stop` 只停底层引擎、不杀 node 包装进程——先 `pkill -f <agentmemory 可执行路径>`，再 `agentmemory stop --force`，然后启动。否则新旧 worker 错配、REST 半挂（症状：status 显示 `Memories: 0`）。
- status CLI 的通道计数偶发 `unknown`/`0`（cosmetic）——**以 REST search 实测为准**。

## data 目录钉死（升级会覆盖）

- 底层引擎默认把数据写到 CWD 相对的 `./data`——改配置（如 `dist/iii-config.yaml`，注意是构建产物那份、不是包根那份）把 `file_path` 钉成**绝对路径**。
- ⚠️ `upgrade` / npm 升级**会覆盖该配置**——升级后若 `Memories: 0` 且 CWD 冒出 `data/`，按上法重新钉死。
- `.gitignore` 掉 `data/`：它是运行时状态，可由知识文件 + 快照重建（记忆库是**可换索引**，不是真相源）。
- **重建不变量**：curated 知识命名空间是知识仓的**纯派生缓存**——任何条目都应能从仓库完整重建。一条不变量同时解两件事：**污染** → 重建清零重来；**多人本地 store 漂移** → `git pull + reindex` 收敛到仓库状态（store 单机、git 仓才是共享介质）。"重建丢未提交的本地写入"是**特性不是缺陷**——它逼每条结论走知识仓 PR 正路。

## 命名空间（防召回被工具噪音霸榜）

- 默认 `search` 多是**纯 BM25**（向量/图谱混合检索也常不区分"策划知识 vs 自动捕获 obs"）。关键词查询会被 hook 自动捕获的 `obs_` 工具流水**霸榜**（实测：窄查询 top-N 普遍全是 obs）。
- 治理：给 curated 知识固定一个**含 `/` 的规范 project 键**（如 `knowledge/<你的域>`）。hook 的 obs 其 project 常取 git 目录 **basename**，含斜杠的键**结构性免撞**。知识召回**必带该键过滤**、写入也**必带该键**。
- **哨兵**：知识召回若命中 `obs_` 前缀条目 = 命名空间泄漏，立查。

## 已知 bug / 反直觉点（写入前必读）

- 某些实现里 **MCP 的 `memory_save` 不把 `project` 落库**——知识写入一律走 **REST `/remember`**（必带 `concepts` + `files` + project 键）。
- **null-project 条目可能漏过所有 project 过滤**（过滤仅在条目 project 非 null 且不等时才排除）——定期 export 查 null。
- 这类问题适合反馈上游（如按 `git remote get-url origin` 自动派生 `owner/repo` 作 project；修 `memory_save` 的 project 落库）。

## 索引纪律（蒸馏 Memory loop）

- 索引时机由 Memory loop 接管：召回@① · 复验所触条目@③ · 索引[**先删后写**]@④ · 收尾对账@⑥。蒸馏切片的 memory 增量是验收产物之一。
- **修订先删后写**：同文件不留多版本（旧版会竞争召回、压过新知识）；删除走治理删除 API。
- 历史 session 摘要可回填（`summarize`）；流程教训存 `lessons`。
