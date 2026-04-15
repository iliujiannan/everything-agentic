---
name: subagent-design-docs
description: 工作流阶段2：设计文档产出。基于技术方案生成 API 文档、系统设计文档等交付物。由 workflow-orchestrator 派发。
---

# 阶段 2：设计文档产出

## 前置条件

- 已接收：阶段 1 产出的技术方案文档路径
- 技术方案已通过用户确认

## 执行步骤

1. **读取方案** — 读取阶段 1 的 `plan-{feature}.md`
2. **生成设计文档** — 调用 `/design-docs` 命令，产出：
   - API 接口文档（参考 `rd/templates/api-doc-template.md`）
   - 系统设计文档（数据模型、接口定义、交互流程）
   - 数据库 DDL 变更说明（如有）
3. **写入文档** — 写入 `.claude/skills/{项目目录}/design-{feature}-{date}.md`

## 产出物

- `.claude/skills/{项目}/design-{feature}-{date}.md` — 设计文档
- （可选）`.claude/skills/{项目}/api-{feature}-{date}.md` — API 文档

## 阶段摘要

完成后输出：

```
## 阶段摘要：设计文档产出
- 状态：PASS
- 产出物：.claude/skills/{项目}/design-{feature}-{date}.md
- Gate 条件：设计文档已生成，等待用户确认
- 待确认事项：请审阅设计文档，输入 [继续] 进入编码阶段
```

**输出摘要后停止，等待用户 [继续]。**
