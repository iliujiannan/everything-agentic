---
name: sonar
description: 运行 SonarQube Cloud 扫描，自动修复问题，验证 Quality Gate，最多 3 轮
origin: ear
---

调用 `/ear:sonar` 执行以下流程：

1. 运行 `mvn clean verify sonar:sonar -Dsonar.token=$SONAR_TOKEN`
2. 读取 Quality Gate 状态和 Issues 列表
3. 整体修复 Bugs / Security Hotspots，不逐条孤立处理
4. 最多 3 轮修复，超出后停止并列出未解决问题
5. Quality Gate PASS 后输出扫描报告（总轮次、修复清单、覆盖率、Quality Gate 状态）

**Quality Gate 未通过时**：禁止进入 Git 提交阶段。

> 规则参考：`rd/rules/java/` 编码规范、`rd/rules/common/security.md` 安全审查清单
