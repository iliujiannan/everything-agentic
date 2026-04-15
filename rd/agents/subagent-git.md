---
name: subagent-git
description: 工作流阶段6：Git 提交。在 Sonar Quality Gate 通过后，生成规范的提交信息并执行提交。由 workflow-orchestrator 派发。
---

# 阶段 6：Git 提交

## 前置条件

- 已接收：阶段 5 的 Quality Gate 状态（必须为 PASS）
- **Quality Gate 未通过时，本阶段不得执行**

## 执行步骤

1. **核查前置条件** — 确认 Quality Gate PASS，否则拒绝执行并提示
2. **生成提交信息** — 调用 `/git-commit` 命令：
   - 遵循 Conventional Commits 格式：`<type>: <description>`
   - type：feat / fix / refactor / docs / test / chore / perf / ci
   - 描述简洁准确，说明"为什么"而非"做了什么"
3. **展示提交信息** — 在执行前展示给用户确认
4. **执行提交** — 用户确认后执行，不使用 `--no-verify`
5. **安全检查** — 提交前确认无硬编码密钥、密码、Token

## 产出物

- Git commit（含 commit hash）

## 阶段摘要

完成后输出：

```
## 阶段摘要：Git 提交
- 状态：PASS
- Commit：{hash} {commit message}
- Gate 条件：提交成功
- 待确认事项：输入 [继续] 进入构建 & 部署阶段
```

**输出摘要后停止，等待用户 [继续]。**
