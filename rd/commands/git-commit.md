---
name: git-commit
description: 展示变更摘要和 commit message 供 review，确认后执行 git 提交
---

## 执行步骤

1. 运行 git diff --stat 和 git status，展示：
   - 变更文件列表
   - 新增/删除行数统计
   - 未追踪文件列表

2. 获取本次提交的任务号
   - 从上一个归属于当前用户的commit记录中提取#任务号，如果没有找到，提示用户输入任务号。
   - 任务号格式：#数字，例如 #123456
3. 根据变更内容生成 Conventional Commits 格式的 commit message：
   格式：#任务号 {type}({scope}): {subject}
   {body}（列出主要变更点）

   可用 type：feat / fix / docs / refactor / test / chore

3. 展示完整 commit message 后停止。
   提示：请确认以上 commit message，可直接回复修改意见，或输入 [继续] 执行提交。

4. 收到确认后执行：
   git add .
   git commit -m "{确认的 commit message}"

   输出 commit hash，然后停止。
   提示：✅ 阶段 6 完成。确认后输入 [继续] 进入 Docker 构建阶段。