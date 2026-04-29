---
description: Sonar 问题修复上下文卡片，直接调用 sonar agent 扫描修复，然后 git-commit
---

# Sonar 问题修复

**范围：** $ARGUMENTS（留空 = 当前分支全量扫描）

## 执行步骤

直接调用 `sonar` agent 执行扫描和修复流程：

1. 运行扫描，读取 Quality Gate 状态和问题列表
2. 整体修复（Bugs 优先 → Security Hotspots → Major Code Smells），最多 **3 轮**
3. 输出报告（轮次、修复清单、覆盖率、Quality Gate 状态）
4. Quality Gate PASS 后，运行 `/git-commit` 提交

> 超出 3 轮停止汇报，不再循环。Quality Gate 未通过**禁止**提交。
