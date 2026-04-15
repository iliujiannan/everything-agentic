---
name: subagent-sonar
description: 工作流阶段5：Sonar 代码扫描。执行 /sonar，自动修复问题，最多循环3轮，Quality Gate 通过后才允许进入 Git 提交。由 workflow-orchestrator 派发。
---

# 阶段 5：Sonar 扫描

## 前置条件

- 已接收：阶段 4 的覆盖率结果（≥ 80%）
- 单元测试全部通过

## 执行步骤

内部 loop，最多 3 轮：

**每轮流程：**
1. 调用 `/sonar` 命令执行扫描
2. 解析扫描结果：Quality Gate 状态、Issues 列表（Blocker/Critical/Major）
3. 若 Quality Gate 通过 → 退出 loop，标记 PASS
4. 若未通过 → 按优先级修复 Blocker → Critical → Major
5. 修复后进入下一轮

**3 轮后仍未通过：**
- 停止修复，不再循环
- 输出未解决问题清单，标记 BLOCKED，等待用户决策

## 产出物

- Sonar 扫描报告摘要（每轮）
- 修复的 Issues 列表

## Gate 条件

- Quality Gate PASS（不满足时禁止进入 Git 提交阶段）

## 阶段摘要

完成后输出：

```
## 阶段摘要：Sonar 扫描
- 状态：PASS / BLOCKED
- 执行轮次：{N}/3
- Quality Gate：PASS / FAIL
- 未解决问题：{若有，列出 Blocker/Critical 问题}
- Gate 条件：{满足 / 未满足}
- 待确认事项：[PASS] 输入 [继续] 进入 Git 提交 / [BLOCKED] 请手动处理上述问题后告知
```

**输出摘要后停止，等待用户 [继续]。**
