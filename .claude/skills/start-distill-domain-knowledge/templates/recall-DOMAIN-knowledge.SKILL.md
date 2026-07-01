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
