---
name: k8s-deploy
description: 展示 K8s manifest 供 review，确认后执行部署并监控健康状态
---

## 执行步骤

1. 读取 k8s/ 目录下所有 manifest 文件
2. 将完整 manifest 内容输出供 review，重点检查：
   - 镜像 tag 是否与刚构建的一致
   - resources.limits 是否已配置
   - readinessProbe 和 livenessProbe 是否已配置
   - 敏感配置是否通过 Secret 注入
   - 如果发现以上任一项缺失，先补充完整再展示

3. 停止等待确认。
   提示：请 review 以上 K8s manifest，确认后输入 [继续] 执行部署。

4. 收到确认后执行：
   kubectl apply -f k8s/
   kubectl rollout status deployment/{app-name} --timeout=120s

5. 等待 rollout 完成后执行：
   kubectl get pods -l app={app-name}

   输出每个 Pod 的状态

6. 全部 Pod Running 后停止。
   提示：✅ 阶段 8 完成。部署成功，所有 Pod 健康 🎉
   如果有 Pod 未就绪，输出 kubectl logs 和 kubectl describe pod 的关键信息。