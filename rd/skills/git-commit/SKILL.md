---
name: git-commit
description: 展示变更摘要和 commit message 供 review，确认后执行 git 提交
origin: ear
---

调用 `/ear:git-commit` 执行以下流程：

1. 运行 `git diff --stat` 和 `git status`，展示变更文件列表和行数统计
2. 从上一个归属于当前用户的 commit 记录中提取任务号（格式：`#数字`）
3. 生成 Conventional Commits 格式 commit message：格式 `#{任务号} {type}({scope}): {subject}`
4. 展示完整 commit message 后停止，等待确认
5. 确认后执行 `git add .` + `git commit`

可用 type：`feat` / `fix` / `docs` / `refactor` / `test` / `chore`

> 规则参考：`rd/rules/common/git-workflow.md`、`rd/rules/common/code-review.md`
