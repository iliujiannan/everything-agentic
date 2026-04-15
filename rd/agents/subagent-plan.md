---
name: subagent-plan
description: 工作流阶段1：技术方案设计。分析需求，调用 planner/architect agent，产出技术方案文档。由 workflow-orchestrator 派发，不直接由用户调用。
---

# 阶段 1：技术方案设计

## 前置条件

- 已接收：功能描述、技术栈信息
- 已读取：`.claude/docs/tech-stack.md`

## 执行步骤

1. **需求拆解** — 将功能描述拆解为具体技术子任务，识别关键风险点
2. **方案设计** — 使用 `Agent(subagent_type: "planner")` 制定实现计划：
   - 产出：PRD、架构设计、任务拆解、依赖项、风险评估
3. **架构评审**（复杂变更时）— 使用 `Agent(subagent_type: "architect")` 评审方案
4. **写入文档** — 将方案写入 `.claude/skills/{项目目录}/plan-{feature}-{date}.md`

## 产出物

- `.claude/skills/{项目}/plan-{feature}-{date}.md` — 技术方案文档

## 阶段摘要

完成后输出：

```
## 阶段摘要：技术方案设计
- 状态：PASS
- 产出物：.claude/skills/{项目}/plan-{feature}-{date}.md
- Gate 条件：方案文档已生成，等待用户确认
- 待确认事项：请审阅技术方案，输入 [继续] 进入设计文档阶段
```

**输出摘要后停止，等待用户 [继续]。**
