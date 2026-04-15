---
name: subagent-code
description: 工作流阶段3：编码实现。依据设计文档完成功能开发，code-reviewer agent 自动介入审查。由 workflow-orchestrator 派发。
---

# 阶段 3：编码实现

## 前置条件

- 已接收：阶段 2 产出的设计文档路径
- 设计文档已通过用户确认

## 执行步骤

1. **读取设计文档** — 读取 `design-{feature}.md` 和 `api-{feature}.md`
2. **实现功能** — 按设计文档逐步实现，遵循项目编码规范（`.claude/rules/`）：
   - 优先实现核心逻辑
   - 遵循分层架构（Controller → Service → Repository）
   - 使用构造器注入，禁止 Field 注入
3. **代码审查** — 每完成一个模块，使用 `Agent(subagent_type: "code-reviewer")` 审查：
   - 修复 CRITICAL 和 HIGH 级别问题后再继续
4. **安全审查** — 涉及认证、用户输入、数据库操作时，使用 `Agent(subagent_type: "security-reviewer")` 审查

## 产出物

- 功能代码文件（按项目目录结构）
- 代码审查报告（如有 CRITICAL/HIGH 问题需记录修复情况）

## 阶段摘要

完成后输出：

```
## 阶段摘要：编码实现
- 状态：PASS / FAIL
- 产出物：{修改/新增的文件列表}
- Gate 条件：代码完成，无编译错误，CRITICAL/HIGH 问题已修复
- 待确认事项：输入 [继续] 进入单元测试阶段
```

**输出摘要后停止，等待用户 [继续]。**
