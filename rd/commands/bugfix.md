---
description: Bug 修复上下文卡片，设定项目规范和修复流程
---

# Bug 修复

**Bug 描述：** $ARGUMENTS

## 项目上下文

- 技术栈：读取 `.claude/docs/tech-stack.md` 获取
- 代码规范：读取 `.claude/rules/` 下对应语言的规则文件
- 异常处理：统一交由 GlobalExceptionHandler 处理

## 修复流程

1. **定位根因** — 阅读调用链，找到真正出问题的代码，不打表面补丁
2. **最小化修复** — 只改必要的代码，不顺手重构不相关内容
3. **code-reviewer 介入** — 修复后调用 `code-reviewer` agent 审查变更
4. **回归测试** — 补充覆盖 bug 场景的测试用例
5. **Sonar 扫描** — 如修改较多，运行 `/sonar` 验证 Quality Gate
6. **Git 提交** — 运行 `/git-commit` 生成 fix message

> 每步完成后停止，输出摘要，等待 **[继续]**。
