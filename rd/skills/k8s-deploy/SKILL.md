---
name: k8s-deploy
description: 展示 K8s manifest 供 review，确认后执行部署并监控健康状态
origin: ear
---

调用 `/ear:k8s-deploy` 执行以下流程：

1. 读取 `k8s/` 目录下所有 manifest 文件
2. 输出完整 manifest 供 review，重点检查：
   - 镜像 tag 与刚构建的一致
   - `resources.limits` 已配置
   - `readinessProbe` 和 `livenessProbe` 已配置
   - 敏感配置通过 Secret 注入（不写在 ConfigMap 里）
3. 缺失项先补充完整再展示
4. 停止等待确认
5. 确认后执行 `kubectl apply -f k8s/` + `kubectl rollout status`
6. 全部 Pod Running 后输出状态，停止

> 规则参考：`rd/rules/common/patterns.md` K8s 部署规范
