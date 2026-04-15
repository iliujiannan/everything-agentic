---
name: sonar
description: 运行 SonarQube Cloud 扫描，自动修复问题，验证 Quality Gate，最多 3 轮
---

## 执行步骤

1. 运行扫描命令：
   mvn clean verify sonar:sonar -Dsonar.token=$SONAR_TOKEN

2. 通过 MCP 工具读取 Quality Gate 状态和 Issues 列表

3. 如果有 Bugs 或 Security Hotspots：
   - 按文件和行号定位问题
   - 整体修复，不逐条孤立处理
   - 重新运行扫描
   - 最多执行 3 轮，超出后停止并列出未解决问题，不再继续

4. Quality Gate PASS 后，输出扫描报告：
   - 总轮次
   - 修复问题清单
   - 最终覆盖率
   - Quality Gate 状态

5. 输出报告后停止。
   提示：✅ 阶段 5 完成。Quality Gate 已通过，确认后输入 [继续] 进入 Git 提交阶段。
   如果 Quality Gate 未通过，提示：❌ 阶段 5 失败。已达 3 轮修复上限，以下问题需人工处理：[列表]